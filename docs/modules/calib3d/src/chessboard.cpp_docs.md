# Documentation for `modules/calib3d/src/chessboard.cpp`

## File Metadata

- **Full Path**: `modules/calib3d/src/chessboard.cpp`
- **File Name**: `chessboard.cpp`
- **File Size**: 127,555 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/calib3d/src/chessboard.cpp](../../../modules/calib3d/src/chessboard.cpp)

## Purpose and Role

This file is located in the `modules/calib3d/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"
#include "opencv2/flann.hpp"
#include "chessboard.hpp"
#include "math.h"

//#define CV_DETECTORS_CHESSBOARD_DEBUG
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
#include <opencv2/highgui.hpp>
static cv::Mat debug_image;
#endif

using namespace std;
namespace cv {
namespace details {

/////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////
// magic numbers used for chessboard corner detection
/////////////////////////////////////////////////////////////////////////////
static const float CORNERS_SEARCH = 0.5F;                       // percentage of the edge length to the next corner used to find new corners
static const float MAX_ANGLE = float(48.0/180.0*CV_PI);          // max angle between line segments supposed to be straight
static const float MIN_COS_ANGLE = float(cos(35.0/180*CV_PI));   // min cos angle between board edges
static const float MIN_RESPONSE_RATIO = 0.1F;
static const float ELLIPSE_WIDTH = 0.35F;                       // width of the search ellipse in percentage of its length
static const float RAD2DEG = float(180.0/CV_PI);
static const int MAX_SYMMETRY_ERRORS = 5;                       // maximal number of failures during point symmetry test (filtering out lines)
/////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////////////

// some helper methods
static float calcSharpness(cv::InputArray _values,float rise_distance);
static bool isPointOnLine(cv::Point2f l1,cv::Point2f l2,cv::Point2f pt,float min_angle);
static int testPointSymmetry(const cv::Mat& mat,cv::Point2f pt,float dist,float max_error);
static float calcSubpixel(const float &x_l,const float &x,const float &x_r);
static float calcSubPos(const float &x_l,const float &x,const float &x_r);
static void polyfit(const Mat& src_x, const Mat& src_y, Mat& dst, int order);
static float calcSignedDistance(const cv::Vec2f &n,const cv::Point2f &a,const cv::Point2f &pt);
static void normalizePoints1D(cv::InputArray _points,cv::OutputArray _T,cv::OutputArray _new_points);
static cv::Mat findHomography1D(cv::InputArray _src,cv::InputArray _dst);
static cv::Mat normalizeVector(cv::InputArray _points);

cv::Mat normalizeVector(cv::InputArray _points)
{
    cv::Mat points = _points.getMat();
    if(points.cols > 1)
    {
        if(points.rows == 1)
            points = points.reshape(points.channels(),points.cols);
        else if(points.channels() == 1)
            points = points.reshape(points.cols,points.rows);
        else
            CV_Error(Error::StsBadArg, "unsupported format");
    }
    return points;
}

float calcSharpness(cv::InputArray _values,float rise_distance)
{
    CV_CheckTypeEQ(_values.type(),CV_8UC1, "values must be of the type CV_8UC1");
    cv::Mat values = normalizeVector(_values);
    if(values.empty())
        return 0;
    if(values.rows != 1 && values.cols != 1)
        CV_Error(Error::StsBadArg, "values must be 1xn or nx1");
    if(rise_distance <= 0.0 || rise_distance > 1.0)
        CV_Error(Error::StsBadArg, "rise_distance must lie in th interval ]0..1]");

    // find global min max
    cv::Point min_loc,max_loc;
    double min_val,max_val;
    cv::minMaxLoc(values,&min_val,&max_val,&min_loc,&max_loc);
    int max_pos = std::max(max_loc.x,max_loc.y);
    int min_pos = std::max(min_loc.x,min_loc.y);
    if(max_pos == min_pos)
        return 0;

    // calc new interval according to the rise distance
    double delta = max_val-min_val;
    double min_val2 = min_val+delta*0.5*(1.0-rise_distance);
    double max_val2 = max_val-delta*0.5*(1.0-rise_distance);

    // find new max starting at min pos
    int dt = 1;
    if(max_pos < min_pos)
        dt= -1;
    int max_pos2 = max_pos;
    for(int i=min_pos+dt;i != max_pos;i+=dt)
    {
        uint8_t val = values.at<uint8_t>(i);
        if(val >= max_val2)
        {
            max_pos2 = i;
            break;
        }
    }

    // find new min starting at max pos
    int min_pos2 = min_pos;
    for(int i=max_pos-dt;i != min_pos;i-=dt)
    {
        uint8_t val = values.at<uint8_t>(i);
        if(val <= min_val2)
        {
            min_pos2 = i;
            break;
        }
    }

    // calc sub pixel max pos
    double max_pos3 = max_pos2;
    uint8_t val1 = values.at<uint8_t>(max_pos2-dt); // <= val2
    uint8_t val2 = values.at<uint8_t>(max_pos2);
    double m = (val2-val1)/dt;
    if(m != 0)
        max_pos3 = max_pos2+(max_val2-val2)/m;

    // calc sub pixel min pos
    double min_pos3 = min_pos2;
    val1 = values.at<uint8_t>(min_pos2); // <= val2
    val2 = values.at<uint8_t>(min_pos2+dt);
    m = (val2-val1)/dt;
    if(m != 0)
        min_pos3 = min_pos2+(min_val2-val1)/m;

    return float(fabs(max_pos3-min_pos3));
}


void normalizePoints1D(cv::InputArray _points,cv::OutputArray _T,cv::OutputArray _new_points)
{
    cv::Mat points = _points.getMat();
    if(points.cols > 1 && points.rows == 1)
        points = points.reshape(1,points.cols);
    CV_CheckChannelsEQ(points.channels(), 1, "points must have only one channel");

    // calc centroid
    double centroid= cv::mean(points)[0];

    // shift origin to centroid
    cv::Mat new_points = points-centroid;

    // calc mean distance
    double mean_dist = cv::mean(cv::abs(new_points))[0];
    if(mean_dist<= DBL_EPSILON)
        CV_Error(Error::StsBadArg, "all given points are identical");
    double scale = 1.0/mean_dist;


    // generate transformation
    cv::Matx22d Tx(
        scale, -scale*centroid,
        0,     1
    );
    Mat(Tx, false).copyTo(_T);

    // calc normalized points;
    _new_points.create(points.rows,1,points.type());
    new_points = _new_points.getMat();
    switch(points.type())
    {
    case CV_32FC1:
        for(int i=0;i < points.rows;++i)
        {
            cv::Vec2d p(points.at<float>(i), 1.0);
            p = Tx*p;
            new_points.at<float>(i) = float(p(0)/p(1));
        }
        break;
    case CV_64FC1:
        for(int i=0;i < points.rows;++i)
        {
            cv::Vec2d p(points.at<double>(i), 1.0);
            p = Tx*p;
            new_points.at<double>(i) = p(0)/p(1);
        }
        break;
    default:
        CV_Error(Error::StsUnsupportedFormat, "unsupported point type");
    }
}

cv::Mat findHomography1D(cv::InputArray _src,cv::InputArray _dst)
{
    // check inputs
    cv::Mat src = _src.getMat();
    cv::Mat dst = _dst.getMat();
    if(src.cols > 1 && src.rows == 1)
        src = src.reshape(1,src.cols);
    if(dst.cols > 1 && dst.rows == 1)
        dst = dst.reshape(1,dst.cols);
    CV_CheckEQ(src.rows, dst.rows, "size mismatch");
    CV_CheckChannelsEQ(src.channels(), 1, "data with only one channel are supported");
    CV_CheckChannelsEQ(dst.channels(), 1, "data with only one channel are supported");
    CV_CheckTypeEQ(src.type(), dst.type(), "src and dst must have the same type");
    CV_Check(src.rows, src.rows >= 3,"at least three point pairs are needed");

    // normalize points
    cv::Mat src_T,dst_T, src_n,dst_n;
    normalizePoints1D(src,src_T,src_n);
    normalizePoints1D(dst,dst_T,dst_n);

    int count = src_n.rows;
    cv::Mat A = cv::Mat::zeros(count,3,CV_64FC1);
    cv::Mat b = cv::Mat::zeros(count,1,CV_64FC1);

    // fill A;b and perform singular value decomposition
    // it is assumed that w is one for both coordinates
    // h22 is kept to 1
    switch(src_n.type())
    {
    case CV_32FC1:
        for(int i=0;i<count;++i)
        {
            double s = src_n.at<float>(i);
            double d = dst_n.at<float>(i);
            A.at<double>(i,0) = s;
            A.at<double>(i,1) = 1.0;
            A.at<double>(i,2) = -s*d;
            b.at<double>(i) = d;
        }
        break;
    case CV_64FC1:
        for(int i=0;i<count;++i)
        {
            double s = src_n.at<double>(i);
            double d = dst_n.at<double>(i);
            A.at<double>(i,0) = s;
            A.at<double>(i,1) = 1.0;
            A.at<double>(i,2) = -s*d;
            b.at<double>(i) = d;
        }
        break;
    default:
        CV_Error(Error::StsUnsupportedFormat,"unsupported type");
    }

    cv::Mat u,d,vt;
    cv::SVD::compute(A,d,u,vt);
    cv::Mat b_ = u.t()*b;

    cv::Mat y(b_.rows,1,CV_64FC1);
    for(int i=0;i<b_.rows;++i)
        y.at<double>(i) = b_.at<double>(i)/d.at<double>(i);

    cv::Mat x = vt.t()*y;
    cv::Matx22d H_(x.at<double>(0), x.at<double>(1), x.at<double>(2), 1.0);

    // denormalize
    Mat H = dst_T.inv()*Mat(H_, false)*src_T;

    // enforce frobeniusnorm of one
    double scale = cv::norm(H);
    CV_Assert(fabs(scale) > DBL_EPSILON);
    scale = 1.0 / scale;
    return H*scale;
}
void polyfit(const Mat& src_x, const Mat& src_y, Mat& dst, int order)
{
    int npoints = src_x.checkVector(1);
    int nypoints = src_y.checkVector(1);
    CV_Assert(npoints == nypoints && npoints >= order+1);
    Mat_<double> srcX(src_x), srcY(src_y);
    Mat_<double> A = Mat_<double>::ones(npoints,order + 1);
    // build A matrix
    for (int y = 0; y < npoints; ++y)
    {
        for (int x = 1; x < A.cols; ++x)
            A.at<double>(y,x) = srcX.at<double>(y)*A.at<double>(y,x-1);
    }
    cv::Mat w;
    solve(A,srcY,w,DECOMP_SVD);
    w.convertTo(dst, ((src_x.depth() == CV_64F || src_y.depth() == CV_64F) ? CV_64F : CV_32F));
}

float calcSignedDistance(const cv::Vec2f &n,const cv::Point2f &a,const cv::Point2f &pt)
{
    cv::Vec3f v1(n[0],n[1],0);
    cv::Vec3f v2(pt.x-a.x,pt.y-a.y,0);
    return v1.cross(v2)[2];
}

bool isPointOnLine(cv::Point2f l1,cv::Point2f l2,cv::Point2f pt,float min_angle)
{
    cv::Vec2f vec1(l1-pt);
    cv::Vec2f vec2(pt-l2);
    if(vec1.dot(vec2) < min_angle*cv::norm(vec1)*cv::norm(vec2))
        return false;
    return true;
}

// returns how many tests fails out of 10
int testPointSymmetry(const cv::Mat &mat,cv::Point2f pt,float dist,float max_error)
{
    cv::Rect image_rect(int(0.5*dist),int(0.5*dist),int(mat.cols-0.5*dist),int(mat.rows-0.5*dist));
    cv::Size size(int(0.5*dist),int(0.5*dist));
    int count = 0;
    cv::Mat patch1,patch2;
    cv::Point2f center1,center2;
    for (int angle_i = 0; angle_i < 10; angle_i++)
    {
        double angle = angle_i * (CV_PI * 0.1);
        cv::Point2f n(float(cos(angle)),float(-sin(angle)));
        center1 = pt+dist*n;
        if(!image_rect.contains(center1))
            return false;
        center2 = pt-dist*n;
        if(!image_rect.contains(center2))
            return false;
        cv::getRectSubPix(mat,size,center1,patch1);
        cv::getRectSubPix(mat,size,center2,patch2);
        if(fabs(cv::mean(patch1)[0]-cv::mean(patch2)[0]) > max_error)
            ++count;
    }
    return count;
}

inline float calcSubpixel(const float &x_l,const float &x,const float &x_r)
{
    // prevent zero values
    if(x_l <= 0)
        return 0;
    if(x <= 0)
        return 0;
    if(x_r <= 0)
        return 0;
    const float l0 = float(std::log(x_l+1e-6));
    const float l1 = float(std::log(x+1e-6));
    const float l2 = float(std::log(x_r+1e-6));
    float delta = l2-l1-l1+l0;
    if(!delta) // this happens if all values are identical
        return 0;
    delta = (l0-l2)/(delta+delta);
    return delta;
}

inline float calcSubPos(const float &x_l,const float &x,const float &x_r)
{
    float val = 2.0F *(x_l-2.0F*x+x_r);
    if(val == 0.0F)
        return 0.0F;
    val = (x_l-x_r)/val;
    if(val > 1.0F)
        return 1.0F;
    if(val < -1.0F)
        return -1.0F;
    return val;
}

FastX::FastX(const Parameters &para)
{
    reconfigure(para);
}

void FastX::reconfigure(const Parameters &para)
{
    CV_Check(para.min_scale, para.min_scale >= 0 && para.min_scale <= para.max_scale, "invalid scale");
    parameters = para;
}

// rotates the image around its center
void FastX::rotate(float angle,cv::InputArray img,cv::Size size,cv::OutputArray out)const
{
    if(angle == 0)
    {
        img.copyTo(out);
        return;
    }
    else
    {
        cv::Matx23d m = cv::getRotationMatrix2D(cv::Point2f(float(img.cols()*0.5),float(img.rows()*0.5)),float(angle/CV_PI*180),1);
        m(0,2) += 0.5*(size.width-img.cols());
        m(1,2) += 0.5*(size.height-img.rows());
        cv::warpAffine(img,out,m,size);
    }
}

void FastX::calcFeatureMap(const Mat &images,Mat& out)const
{
    if(images.empty())
        CV_Error(Error::StsBadArg,"no rotation images");
    int type = images.type(), depth = CV_MAT_DEPTH(type);
    CV_CheckType(type,depth == CV_8U,
            "Only 8-bit grayscale or color images are supported");
    if(!images.isContinuous())
        CV_Error(Error::StsBadArg,"image must be continuous");

    float signal,noise,rating;
    int count1;
    unsigned char val1,val2,val3;
    const unsigned char* wrap_around;
    const unsigned char* pend;
    const unsigned char* pimages = images.data;
    const int channels = images.channels();
    if(channels < 4)
        CV_Error(Error::StsBadArg,"images must have at least four channels");

    // for each pixel
    out = cv::Mat::zeros(images.rows,images.cols,CV_32FC1);
    const float *pout_end = reinterpret_cast<const float*>(out.dataend);
    for(float *pout=out.ptr<float>(0,0);pout != pout_end;++pout)
    {
        //reset values
        rating = 0.0; count1 = 0;
        noise = 255; signal = 0;

        //calc rating
        pend = pimages+channels;
        val1 = *(pend-1);                       // wrap around (last value)
        wrap_around = pimages++;                // store for wrap around (first value)
        val2 = *wrap_around;                    // first value
        for(;pimages != pend;++pimages)
        {
            val3 = *pimages;
            if(val1 <= val2)
            {
                if(val3 < val2) // maxima
                {
                    if(signal < val2)
                        signal = val2;
                    ++count1;
                }
            }
            else if(val1 > val2 && val3 >= val2) // minima
            {
                if(noise > val2)
                    noise = val2;
                ++count1;
            }
            val1 = val2;
            val2 = val3;
        }
        // wrap around
        if(val1 <= val2) // maxima
        {
            if(*wrap_around < val2)
            {
                if(signal < val2)
                    signal = val2;
                ++count1;
            }
        }
        else if(val1 > val2 && *wrap_around >= val2) // minima
        {
            if(noise > val2)
                noise = val2;
            ++count1;
        }

        // store rating
        if(count1 == parameters.branches)
        {
            rating = signal-noise;
            *pout = rating*rating; //store rating in the feature map
        }
    }
}

std::vector<std::vector<float> > FastX::calcAngles(const std::vector<cv::Mat> &rotated_images,std::vector<cv::KeyPoint> &keypoints)const
{
    // validate rotated_images
    if(rotated_images.empty())
        CV_Error(Error::StsBadArg,"no rotated images");
    std::vector<cv::Mat>::const_iterator iter = rotated_images.begin();
    for(;iter != rotated_images.end();++iter)
    {
        if(iter->empty())
            CV_Error(Error::StsBadArg,"empty rotated images");
        if(iter->channels() < 4)
            CV_Error(Error::StsBadArg,"rotated images must have at least four channels");
    }

    // assuming all elements of the same channel
    const int channels = rotated_images.front().channels();
    const int channels_1 = channels-1;
    const float resolution = float(CV_PI/channels);
    const float scale = float(parameters.super_resolution)+1.0F;

    // for each keypoint
    std::vector<std::vector<float> > angles;
    angles.resize(keypoints.size());
    parallel_for_(Range(0,(int)keypoints.size()),[&](const Range& range)
    {
        float angle;
        float val1,val2,val3,wrap_around;
        const unsigned char *pimages1,*pimages2,*pimages3,*pimages4;
        std::vector<cv::KeyPoint>::iterator pt_iter = keypoints.begin()+range.start;
        std::vector<cv::KeyPoint>::iterator pt_end = keypoints.begin()+range.end;
        for(int id=range.start ;pt_iter != pt_end;++pt_iter,++id)
        {
            int scale_id = pt_iter->octave - parameters.min_scale;
            if(scale_id>= int(rotated_images.size()) ||scale_id < 0)
                CV_Error(Error::StsBadArg,"no rotated image for requested keypoint octave");
            const cv::Mat &s_rotated_images = rotated_images[scale_id];

            float x2 = pt_iter->pt.x*scale;
            float y2 = pt_iter->pt.y*scale;
            int row = int(y2);
            int col = int(x2);
            x2 -= col;
            y2 -= row;
            float x1 = 1.0F-x2; float y1 = 1.0F-y2;
            float a = x1*y1; float b = x2*y1; float c = x1*y2; float d = x2*y2;
            pimages1 = s_rotated_images.ptr<unsigned char>(row,col);
            pimages2 = s_rotated_images.ptr<unsigned char>(row,col+1);
            pimages3 = s_rotated_images.ptr<unsigned char>(row+1,col);
            pimages4 = s_rotated_images.ptr<unsigned char>(row+1,col+1);
            std::vector<float> &angles_i = angles[id];

            //calc rating
            val1 = a**(pimages1+channels_1)+b**(pimages2+channels_1)+
                c**(pimages3+channels_1)+d**(pimages4+channels_1);        // wrap around (last value)
            wrap_around = a**(pimages1++)+b**(pimages2++)+c**(pimages3++)+d**(pimages4++); // first value
            val2 = wrap_around;                                                             // first value
            for(int i=0;i<channels-1;++pimages1,++pimages2,++pimages3,++pimages4,++i)
            {
                val3 = a**(pimages1)+b**(pimages2)+c**(pimages3)+d**(pimages4);
                if(val1 <= val2)
                {
                    if(val3 < val2)
                    {
                        angle = float((calcSubPos(val1,val2,val3)+i)*resolution);
                        if(angle < 0)
                            angle += float(CV_PI);
                        else if(angle > CV_PI)
                            angle -= float(CV_PI);
                        angles_i.push_back(angle);
                        pt_iter->angle = 360.0F-angle*RAD2DEG;
                    }
                }
                else if(val1 > val2 && val3 >= val2)
                {
                    angle = float((calcSubPos(val1,val2,val3)+i)*resolution);
                    if(angle < 0)
                        angle += float(CV_PI);
                    else if(angle > CV_PI)
                        angle -= float(CV_PI);
                    angles_i.push_back(-angle);
                    pt_iter->angle = 360.0F-angle*RAD2DEG;
                }
                val1 = val2;
                val2 = val3;
            }
            // wrap around
            if(val1 <= val2)
            {
                if(wrap_around< val2)
                {
                    angle = float((calcSubPos(val1,val2,wrap_around)+channels-1)*resolution);
                    if(angle < 0)
                        angle += float(CV_PI);
                    else if(angle > CV_PI)
                        angle -= float(CV_PI);
                    angles_i.push_back(angle);
                    pt_iter->angle = 360.0F-angle*RAD2DEG;
                }
            }
            else if(val1 > val2 && wrap_around >= val2)
            {
                angle = float((calcSubPos(val1,val2,wrap_around)+channels-1)*resolution);
                if(angle < 0)
                    angle += float(CV_PI);
                else if(angle > CV_PI)
                    angle -= float(CV_PI);
                angles_i.push_back(-angle);
                pt_iter->angle = 360.0F-angle*RAD2DEG;
            }
        }
    });
    return angles;
}

void FastX::findKeyPoints(const std::vector<cv::Mat> &feature_maps, std::vector<KeyPoint>& keypoints,const Mat& _mask) const
{
    //TODO check that all feature_maps have the same size
    int num_scales = parameters.max_scale-parameters.min_scale;
    CV_CheckGE(int(feature_maps.size()), num_scales, "missing feature maps");
    if (!_mask.empty())
    {
        CV_CheckTypeEQ(_mask.type(), CV_8UC1, "wrong mask type");
        CV_CheckEQ(_mask.size(), feature_maps.front().size(),"wrong mask type or size");
    }
    keypoints.clear();

    cv::Mat mask;
    if(!_mask.empty())
        mask = _mask;
    else
        mask = cv::Mat::ones(feature_maps.front().size(),CV_8UC1);

    int super_res = int(parameters.super_resolution);
    int super_scale = super_res+1;
    float super_comp = 0.25F*super_res;

    // for each scale
    float strength = parameters.strength;
    std::vector<int> windows;
    cv::Point pt,pt2;
    double min,max;
    cv::Mat src;
    for(int scale=parameters.max_scale;scale>=parameters.min_scale;--scale)
    {
        int window_size = (1 << (scale + super_res)) + 1;
        float window_size2 = 0.5F*window_size;
        float window_size4 = 0.25F*window_size;
        int window_size2i = cvRound(window_size2);

        const cv::Mat &feature_map = feature_maps[scale-parameters.min_scale];
        int y = ((feature_map.rows)/window_size)-2;
        int x = ((feature_map.cols)/window_size)-2;
        for(int row=1;row<y;++row)
        {
            for(int col=1;col<x;++col)
            {
                Rect rect(col*window_size,row*window_size,window_size,window_size);
                src = feature_map(rect);
                cv::minMaxLoc(src,&min,&max,NULL,&pt);
                if(min == max || max < strength)
                    continue;

                cv::Point pos(pt.x+rect.x,pt.y+rect.y);
                if(mask.at<unsigned char>(pos.y,pos.x) == 0)
                    continue;

                Rect rect2(int(pos.x-window_size2),int(pos.y-window_size2),window_size,window_size);
                src = feature_map(rect2);
                cv::minMaxLoc(src,NULL,NULL,NULL,&pt2);
                if(pos.x == pt2.x+rect2.x && pos.y == pt2.y+rect2.y)
                {
                    // the point is the best one on the current scale
                    // check all larger scales if there is a stronger one
                    double max2;
                    int scale2= scale-1;
                    //parameters.min_scale;
                    for(;scale2>=parameters.min_scale;--scale2)
                    {
                        cv::minMaxLoc(feature_maps[scale2-parameters.min_scale](rect),NULL,&max2,NULL,NULL);
                        if(max2 > max)
                            break;
                    }
                    if(scale2<parameters.min_scale && pos.x+1 < feature_map.cols && pos.y+1 < feature_map.rows)
                    {
                        float sub_x = float(calcSubpixel(feature_map.at<float>(pos.y,pos.x-1),
                                feature_map.at<float>(pos.y,pos.x),
                                feature_map.at<float>(pos.y,pos.x+1)));
                        float sub_y = float(calcSubpixel(feature_map.at<float>(pos.y-1,pos.x),
                                feature_map.at<float>(pos.y,pos.x),
                                feature_map.at<float>(pos.y+1,pos.x)));
                        cv::KeyPoint kpt(sub_x+pos.x,sub_y+pos.y,float(window_size),0.F,float(max),scale);
                        int x2 = std::max(0,int(kpt.pt.x-window_size4));
                        int y2 = std::max(0,int(kpt.pt.y-window_size4));
                        int w = std::min(int(mask.cols-x2),window_size2i);
                        int h = std::min(int(mask.rows-y2),window_size2i);
                        mask(cv::Rect(x2,y2,w,h)) = 0.0;
                        if(super_scale != 1)
                        {
                            kpt.pt.x /= super_scale;
                            kpt.pt.y /= super_scale;
                            kpt.pt.x -= super_comp;
                            kpt.pt.y -= super_comp;
                            kpt.size /= super_scale;
                        }
                        keypoints.push_back(kpt);
                    }
                }
            }
        }
    }
}

void FastX::detectAndCompute(cv::InputArray image,cv::InputArray mask,std::vector<cv::KeyPoint>& keypoints,
        cv::OutputArray _descriptors,bool useProvidedKeyPoints)
{
    useProvidedKeyPoints = false;
    detectImpl(image.getMat(),keypoints,mask.getMat());
    if(!_descriptors.needed())
        return;

    // generate descriptors based on their position
    _descriptors.create(int(keypoints.size()),2,CV_32FC1);
    cv::Mat descriptors = _descriptors.getMat();
    std::vector<cv::KeyPoint>::const_iterator iter = keypoints.begin();
    for(int row=0;iter != keypoints.end();++iter,++row)
    {
        descriptors.at<float>(row,0) = iter->pt.x;
        descriptors.at<float>(row,1) = iter->pt.y;
    }
    if(!useProvidedKeyPoints)        // suppress compiler warning
        return;
    return;
}

void FastX::detectImpl(const cv::Mat& _gray_image,
        std::vector<cv::Mat> &rotated_images,
        std::vector<cv::Mat> &feature_maps,
        const cv::Mat &_mask)const
{
    if(!_mask.empty())
        CV_Error(Error::StsBadSize, "Mask is not supported");
    CV_CheckTypeEQ(_gray_image.type(), CV_8UC1, "Unsupported image type");

    // up-sample if needed
    cv::UMat gray_image;
    const int super_res = int(parameters.super_resolution);
    if(super_res)
        cv::resize(_gray_image,gray_image,cv::Size(),2,2);
    else
        _gray_image.copyTo(gray_image);

    //for each scale
    const int num_scales = parameters.max_scale-parameters.min_scale+1;
    const int diag = int(sqrt(gray_image.rows*gray_image.rows+gray_image.cols*gray_image.cols));
    const cv::Size size(diag,diag);
    const int num = int(0.5001*CV_PI/parameters.resolution);

    rotated_images.resize(num_scales);
    feature_maps.resize(num_scales);

    parallel_for_(Range(parameters.min_scale,parameters.max_scale+1),[&](const Range& range){
        for(int scale=range.start;scale < range.end;++scale)
        {
            // calc images
            // for each angle step
            int scale_id = scale-parameters.min_scale;
            int scale_size = int(pow(2.0,scale+1+super_res));
            int scale_size2 = int((scale_size/7)*2+1);
            std::vector<cv::UMat> images;
            images.resize(2*num);
            cv::UMat rotated,filtered_h,filtered_v;
            cv::boxFilter(gray_image,images[0],-1,cv::Size(scale_size,scale_size2));
            cv::boxFilter(gray_image,images[num],-1,cv::Size(scale_size2,scale_size));
            for(int i=1;i<num;++i)
            {
                float angle = parameters.resolution*i;
                rotate(-angle,gray_image,size,rotated);
                cv::boxFilter(rotated,filtered_h,-1,cv::Size(scale_size,scale_size2));
                cv::boxFilter(rotated,filtered_v,-1,cv::Size(scale_size2,scale_size));

                // rotate filtered images back
                rotate(angle,filtered_h,gray_image.size(),images[i]);
                rotate(angle,filtered_v,gray_image.size(),images[i+num]);
            }
            cv::merge(images,rotated_images[scale_id]);

            // calc feature map
            calcFeatureMap(rotated_images[scale_id],feature_maps[scale_id]);

            // filter feature map to improve impulse responses
            if(parameters.filter)
            {
                cv::Mat high,low;
                cv::boxFilter(feature_maps[scale_id],low,-1,cv::Size(scale_size,scale_size));
                int scale2 = int((scale_size/6))*2+1;
                cv::boxFilter(feature_maps[scale_id],high,-1,cv::Size(scale2,scale2));
                feature_maps[scale_id] = high-0.8*low;
            }
        }
    });
}

void FastX::detectImpl(const cv::Mat& image,std::vector<cv::KeyPoint>& keypoints,std::vector<cv::Mat> &feature_maps,const cv::Mat &mask)const
{
    std::vector<cv::Mat> rotated_images;
    detectImpl(image,rotated_images,feature_maps,mask);
    findKeyPoints(feature_maps,keypoints,mask);
}

void FastX::detectImpl(InputArray image, std::vector<KeyPoint>& keypoints, InputArray mask)const
{
    std::vector<cv::Mat> feature_maps;
    detectImpl(image.getMat(),keypoints,feature_maps,mask.getMat());
}

void FastX::detectImpl(const Mat& src, std::vector<KeyPoint>& keypoints, const Mat& mask)const
{
    std::vector<cv::Mat> feature_maps;
    detectImpl(src,keypoints,feature_maps,mask);
}


Ellipse::Ellipse():
    angle(0),
    cosf(0),
    sinf(0)
{
}

Ellipse::Ellipse(const cv::Point2f &_center, const cv::Size2f &_axes, float _angle):
    center(_center),
    axes(_axes),
    angle(_angle),
    cosf(cos(-_angle)),
    sinf(sin(-_angle))
{
}

const cv::Size2f &Ellipse::getAxes()const
{
    return axes;
}

cv::Point2f Ellipse::getCenter()const
{
    return center;
}

void Ellipse::draw(cv::InputOutputArray img,const cv::Scalar &color)const
{
    cv::ellipse(img,center,axes,360-angle/CV_PI*180,0,360,color);
}

bool Ellipse::contains(const cv::Point2f &pt)const
{
    cv::Point2f ptc = pt-center;
    float x = cosf*ptc.x+sinf*ptc.y;
    float y = -sinf*ptc.x+cosf*ptc.y;
    if(x*x/(axes.width*axes.width)+y*y/(axes.height*axes.height) <= 1.0)
        return true;
    return false;
}


// returns false if the angle from the line pt1-pt2 to the line pt3-pt4 is negative
static bool checkOrientation(const cv::Point2f &pt1,const cv::Point2f &pt2,
        const cv::Point2f &pt3,const cv::Point2f &pt4)
{
    cv::Point3f p1(pt2.x-pt1.x,pt2.y-pt1.y,0);
    cv::Point3f p2(pt4.x-pt3.x,pt4.y-pt3.y,0);
    return p1.cross(p2).z > 0;
}

static bool sortKeyPoint(const cv::KeyPoint &pt1,const cv::KeyPoint &pt2)
{
    // used as comparison function for partial sort
    // the keypoints with the best score should be first
    return pt1.response > pt2.response;
}

cv::Mat Chessboard::getObjectPoints(const cv::Size &pattern_size,float cell_size)
{
    cv::Mat result(pattern_size.width*pattern_size.height,1,CV_32FC3);
    for(int row=0;row < pattern_size.height;++row)
    {
        for(int col=0;col< pattern_size.width;++col)
        {
            cv::Point3f &pt = *result.ptr<cv::Point3f>(row*pattern_size.width+col);
            pt.x = cell_size*col;
            pt.y = cell_size*row;
            pt.z = 0;
        }
    }
    return result;
}

bool Chessboard::Board::Cell::isInside(const cv::Point2f &pt)const
{
    if(empty())
        return false;
    float l1 = (pt.x-top_left->x)*(bottom_left->y-top_left->y) - (bottom_left->x-top_left->x)*(pt.y-top_left->y);
    float  l2 = (pt.x-top_right->x)*(top_left->y-top_right->y) - (top_left->x-top_right->x)*(pt.y-top_right->y);
    float  l3 = (pt.x-bottom_left->x)*(top_right->y-bottom_left->y) - (top_right->x-bottom_left->x)*(pt.y-bottom_left->y);
    if((l1>0 && l2>0  && l3>0) || (l1<0 && l2<0 && l3<0))
        return true;
    l1 = (pt.x-top_left->x)*(bottom_right->y-top_left->y) - (bottom_right->x-top_left->x)*(pt.y-top_left->y);
    l2 = (pt.x-top_right->x)*(top_left->y-top_right->y) - (top_left->x-top_right->x)*(pt.y-top_right->y);
    l3 = (pt.x-bottom_right->x)*(top_right->y-bottom_right->y) - (top_right->x-bottom_right->x)*(pt.y-bottom_right->y);
    return (l1>0 && l2>0  && l3>0) || (l1<0 && l2<0 && l3<0);
}

bool Chessboard::Board::Cell::empty()const
{
    // check if one of its corners has NaN
    if(top_left->x != top_left->x || top_left->y != top_left->y)
        return true;
    if(top_right->x != top_right->x || top_right->y != top_right->y)
        return true;
    if(bottom_right->x != bottom_right->x || bottom_right->y != bottom_right->y)
        return true;
    if(bottom_left->x != bottom_left->x || bottom_left->y != bottom_left->y)
        return true;
    return false;
}

int Chessboard::Board::Cell::getRow()const
{
    int row = 0;
    Cell const* temp = this;
    for(;temp->top;temp=temp->top,++row);
    return row;
}

cv::Point2f Chessboard::Board::Cell::getCenter()const
{
    if(empty())
        CV_Error(Error::StsBadArg,"Cell is empty");
    cv::Point2f center = *top_left+*top_right+*bottom_left+*bottom_right;
    center.x /=4;
    center.y /=4;
    return center;
}

int Chessboard::Board::Cell::getCol()const
{
    int col = 0;
    Cell const* temp = this;
    for(;temp->left;temp=temp->left,++col);
    return col;
}

Chessboard::Board::Cell::Cell() :
    top_left(NULL), top_right(NULL), bottom_right(NULL), bottom_left(NULL),
    left(NULL), top(NULL), right(NULL), bottom(NULL),black(false),marker(false)
{}

Chessboard::Board::PointIter::PointIter(Cell *_cell,CornerIndex _corner_index):
    corner_index(_corner_index),
    cell(_cell)
{
}

Chessboard::Board::PointIter::PointIter(const PointIter &other)
{
    this->operator=(other);
}

void Chessboard::Board::PointIter::operator=(const PointIter &other)
{
    corner_index = other.corner_index;
    cell = other.cell;
}

Chessboard::Board::Cell* Chessboard::Board::PointIter::getCell()
{
    return cell;
}

bool Chessboard::Board::PointIter::valid()const
{
    return cell != NULL;
}

bool Chessboard::Board::PointIter::isNaN()const
{
    const cv::Point2f *pt = operator*();
    if(pt->x != pt->x || pt->y != pt->y)        // NaN check
        return true;
    return false;
}

bool Chessboard::Board::PointIter::checkCorner()const
{
    if(!cell->empty())
        return true;
    // test all other cells
    switch(corner_index)
    {
    case BOTTOM_LEFT:
        if(cell->left)
        {
            if(!cell->left->empty())
                return true;
            if(cell->left->bottom && !cell->left->bottom->empty())
                return true;
        }
        if(cell->bottom)
        {
            if(!cell->bottom->empty())
                return true;
            if(cell->bottom->left && !cell->bottom->left->empty())
                return true;
        }
        break;
    case TOP_LEFT:
        if(cell->left)
        {
            if(!cell->left->empty())
                return true;
            if(cell->left->top && !cell->left->top->empty())
                return true;
        }
        if(cell->top)
        {
            if(!cell->top->empty())
                return true;
            if(cell->top->left && !cell->top->left->empty())
                return true;
        }
        break;
    case TOP_RIGHT:
        if(cell->right)
        {
            if(!cell->right->empty())
                return true;
            if(cell->right->top && !cell->right->top->empty())
                return true;
        }
        if(cell->top)
        {
            if(!cell->top->empty())
                return true;
            if(cell->top->right && !cell->top->right->empty())
                return true;
        }
        break;
    case BOTTOM_RIGHT:
        if(cell->right)
        {
            if(!cell->right->empty())
                return true;
            if(cell->right->bottom && !cell->right->bottom->empty())
                return true;
        }
        if(cell->bottom)
        {
            if(!cell->bottom->empty())
                return true;
            if(cell->bottom->right && !cell->bottom->right->empty())
                return true;
        }
        break;
    default:
        CV_Assert(false);
    }
    return false;
}


bool Chessboard::Board::PointIter::left(bool check_empty)
{
    switch(corner_index)
    {
    case BOTTOM_LEFT:
        if(cell->left && (!check_empty || !cell->left->empty()))
            cell = cell->left;
        else if(check_empty && cell->bottom && cell->bottom->left && !cell->bottom->left->empty())
        {
            cell = cell->bottom->left;
            corner_index = TOP_LEFT;
        }
        else
            return false;
        break;
    case TOP_LEFT:
        if(cell->left && (!check_empty || !cell->left->empty()))
            cell = cell->left;
        else if(check_empty && cell->top && cell->top->left && !cell->top->left->empty())
        {
            cell = cell->top->left;
            corner_index = BOTTOM_LEFT;
        }
        else
            return false;
        break;
    case TOP_RIGHT:
        corner_index = TOP_LEFT;
        break;
    case BOTTOM_RIGHT:
        corner_index = BOTTOM_LEFT;
        break;
    default:
        CV_Assert(false);
    }
    return true;
}

bool Chessboard::Board::PointIter::top(bool check_empty)

{
    switch(corner_index)
    {
    case TOP_RIGHT:
        if(cell->top && (!check_empty || !cell->top->empty()))
            cell = cell->top;
        else if(check_empty && cell->right && cell->right->top&& !cell->right->top->empty())
        {
            cell = cell->right->top;
            corner_index = TOP_LEFT;
        }
        else
            return false;
        break;
    case TOP_LEFT:
        if(cell->top && (!check_empty || !cell->top->empty()))
            cell = cell->top;
        else if(check_empty && cell->left && cell->left->top&& !cell->left->top->empty())
        {
            cell = cell->left->top;
            corner_index = TOP_RIGHT;
        }
        else
            return false;
        break;
    case BOTTOM_LEFT:
        corner_index = TOP_LEFT;
        break;
    case BOTTOM_RIGHT:
        corner_index = TOP_RIGHT;
        break;
    default:
        CV_Assert(false);
    }
    return true;
}

bool Chessboard::Board::PointIter::right(bool check_empty)
{
    switch(corner_index)
    {
    case TOP_RIGHT:
        if(cell->right && (!check_empty || !cell->right->empty()))
            cell = cell->right;
        else if(check_empty && cell->top && cell->top->right && !cell->top->right->empty())
        {
            cell = cell->top->right;
            corner_index = BOTTOM_RIGHT;
        }
        else
            return false;
        break;
    case BOTTOM_RIGHT:
        if(cell->right && (!check_empty || !cell->right->empty()))
            cell = cell->right;
        else if(check_empty && cell->bottom && cell->bottom->right && !cell->bottom->right->empty())
        {
            cell = cell->bottom->right;
            corner_index = TOP_RIGHT;
        }
        else
            return false;
        break;
    case TOP_LEFT:
        corner_index = TOP_RIGHT;
        break;
    case BOTTOM_LEFT:
        corner_index = BOTTOM_RIGHT;
        break;
    default:
        CV_Assert(false);
    }
    return true;
}

bool Chessboard::Board::PointIter::bottom(bool check_empty)
{
    switch(corner_index)
    {
    case BOTTOM_LEFT:
        if(cell->bottom && (!check_empty || !cell->bottom->empty()))
            cell = cell->bottom;
        else if(check_empty && cell->left && cell->left->bottom && !cell->left->bottom->empty())
        {
            cell = cell->left->bottom;
            corner_index = BOTTOM_RIGHT;
        }
        else
            return false;
        break;
    case BOTTOM_RIGHT:
        if(cell->bottom && (!check_empty || !cell->bottom->empty()))
            cell = cell->bottom;
        else if(check_empty && cell->right && cell->right->bottom && !cell->right->bottom->empty())
        {
            cell = cell->right->bottom;
            corner_index = BOTTOM_LEFT;
        }
        else
            return false;
        break;
    case TOP_LEFT:
        corner_index = BOTTOM_LEFT;
        break;
    case TOP_RIGHT:
        corner_index = BOTTOM_RIGHT;
        break;
    default:
        CV_Assert(false);
    }
    return true;
}


const cv::Point2f* Chessboard::Board::PointIter::operator*()const
{
    switch(corner_index)
    {
    case TOP_LEFT:
        return cell->top_left;
    case TOP_RIGHT:
        return cell->top_right;
    case BOTTOM_RIGHT:
        return cell->bottom_right;
    case BOTTOM_LEFT:
        return cell->bottom_left;
    }
    CV_Assert(false);
}

const cv::Point2f* Chessboard::Board::PointIter::operator->()const
{
    return operator*();
}

cv::Point2f* Chessboard::Board::PointIter::operator*()
{
    const cv::Point2f *pt = const_cast<const PointIter*>(this)->operator*();
    return const_cast<cv::Point2f*>(pt);
}

cv::Point2f* Chessboard::Board::PointIter::operator->()
{
    return operator*();
}

Chessboard::Board::Board(float _white_angle,float _black_angle):
    top_left(NULL),
    rows(0),
    cols(0),
    white_angle(_white_angle),
    black_angle(_black_angle)
{
}


Chessboard::Board::Board(const Chessboard::Board &other):
    top_left(NULL),
    rows(0),
    cols(0)
{
    *this = other;
}

Chessboard::Board::Board(const cv::Size &size, const std::vector<cv::Point2f> &points,float _white_angle,float _black_angle):
    top_left(NULL),
    rows(0),
    cols(0),
    white_angle(_white_angle),
    black_angle(_black_angle)
{
    if(size.width*size.height != int(points.size()))
        CV_Error(Error::StsBadArg,"size mismatch");
    if(size.width < 3 || size.height < 3)
        CV_Error(Error::StsBadArg,"at least 3 rows and cols are needed to initialize the board");

    // init board with 3x3
    // TODO write function speeding up the copying
    cv::Mat data = cv::Mat(points).reshape(2,size.height);
    cv::Mat temp;
    data(cv::Rect(0,0,3,3)).copyTo(temp);
    std::vector<cv::Point2f> ipoints = temp.reshape(2,1);
    if(!init(ipoints))
        return;

    // add all cols if more than 3
    for(int col=3 ; col< data.cols;++col)
    {
        data(cv::Rect(col,0,1,3)).copyTo(temp);
        ipoints = temp.reshape(2,1);
        addColumnRight(ipoints);
    }

    // add all rows if more than 3
    for(int row=3; row < data.rows;++row)
    {
        data(cv::Rect(0,row,cols,1)).copyTo(temp);
        ipoints = temp.reshape(2,1);
        addRowBottom(ipoints);
    }
}

Chessboard::Board::~Board()
{
    clear();
}

void Chessboard::Board::setAngles(float white, float black)
{
    white_angle = white;
    black_angle = black;
}

float Chessboard::Board::getAngle()const
{
    if(isEmpty())
        CV_Error(Error::StsBadArg,"Board is empty");
    if(colCount() < 3)
        CV_Error(Error::StsBadArg,"Board is too small");

    cv::Point2f delta = *(top_left->right->top_right)-*(top_left->top_left);
    cv::Point3f pt(delta.x,delta.y,0);
    float val;
    if(fabs(pt.x) > fabs(pt.y))
    {
        cv::Point3f ptx(1,0,0);
        val = float(ptx.dot(pt)/cv::norm(pt));
        if(val < 0)
            val = -acos(val);
        else
            val = acos(val);
    }
    else
    {
        cv::Point3f ptx(0,1,0);
        val = float(ptx.dot(pt)/cv::norm(pt));
        if(val < 0)
            val = float(-acos(val)+CV_PI/2);
        else
            val = float(acos(val)+CV_PI/2);
    }
    return val;
}

bool Chessboard::Board::isHorizontal()const
{
    double angle = getAngle();
    if((angle < 0.25*CV_PI && angle > -0.25*CV_PI) || angle > 0.75*CV_PI || angle < -0.75*CV_PI)
        return true;
    return false;
}

cv::Mat Chessboard::Board::getObjectPoints(float cell_size)const
{
    cv::Mat points = Chessboard::getObjectPoints(getSize(),cell_size);

    // check for any offset due to a found marker
    for(auto &&cell : cells)
    {
        if(cell->marker && !cell->black)
        {
            // apply offset
            cv::Point3f offset(cell->getCol()*cell_size,cell->getRow()*cell_size,0);
            for(int i =0;i < points.rows;++i)
                points.at<cv::Point3f>(i) -= offset;
            break;
        }
    }
    return points;
}

std::vector<cv::Point2f> Chessboard::Board::getCellCenters()const
{
    int icols = int(colCount());
    int irows = int(rowCount());
    if(icols < 3 || irows < 3)
        CV_Error(Error::StsBadArg,"Chessboard must be at least consist of 3 rows and cols to calculate the cell centers");

    std::vector<cv::Point2f> points;
    cv::Matx33d H(estimateHomography(DUMMY_FIELD_SIZE));
    cv::Vec3d pt1,pt2;
    pt1[2] = 1;
    for(int row = 0;row < irows;++row)
    {
        pt1[1] = (0.5+row)*DUMMY_FIELD_SIZE;
        for(int col= 0;col< icols;++col)
        {
            pt1[0] = (0.5+col)*DUMMY_FIELD_SIZE;
            pt2 = H*pt1;
            points.push_back(cv::Point2f(float(pt2[0]/pt2[2]),float(pt2[1]/pt2[2])));
        }
    }
    return points;
}

std::vector<cv::Mat> Chessboard::Board::getCells(float shrink_factor,bool bwhite,bool bblack) const
{
    std::vector<cv::Mat> result;
    int icols = int(colCount());
    int irows = int(rowCount());
    if(icols < 3 || irows < 3)
        return result;

    for(int row=0;row<irows-1;++row)
    {
        for(int col=0;col<icols-1;++col)
        {
            const Cell *cell = getCell(row,col);
            if(!bwhite && !cell->black)
                continue;
            if(!bblack && cell->black)
                continue;
            cv::Mat points = cv::Mat(4,1,CV_32FC2);
            points.at<cv::Point2f>(0) = *cell->top_left;
            points.at<cv::Point2f>(1) = *cell->top_right;
            points.at<cv::Point2f>(2) = *cell->bottom_right;
            points.at<cv::Point2f>(3) = *cell->bottom_left;
            if(shrink_factor != 1)
            {
                cv::Point2f center = *cell->top_left+*cell->top_right+*cell->bottom_left+*cell->bottom_right;
                center.x /=4;
                center.y /=4;
                for(int i=0;i<4;++i)
                {
                    auto &pt = points.at<cv::Point2f>(i);
                    pt = center+(pt-center)*shrink_factor;
                }
            }
            result.push_back(points);
        }
    }
    return result;
}

cv::Mat Chessboard::Board::warpImage(cv::InputArray image)const
{
    cv::Mat H = estimateHomography();
    cv::Mat mat;
    cv::Size size = getSize();
    size.width = (size.width+1)*DUMMY_FIELD_SIZE;
    size.height= (size.height+1)*DUMMY_FIELD_SIZE;
    cv::warpPerspective(image,mat,H.inv(),size);
    return mat;
}

void Chessboard::Board::draw(cv::InputArray m,cv::OutputArray out,cv::InputArray _H)const
{
    cv::Mat H = _H.getMat();
    if(H.empty())
        H = estimateHomography();
    cv::Mat image = m.getMat().clone();
    if(image.type() == CV_32FC1)
    {
        double maxVal,minVal;
        cv::minMaxLoc(image, &minVal, &maxVal);
        double scale = 255.0/(maxVal-minVal);
        image.convertTo(image,CV_8UC1,scale,-scale*minVal);
        cv::applyColorMap(image,image,cv::COLORMAP_JET);
    }

    // draw all points and search areas
    std::vector<cv::Point2f> points = getCorners();
    std::vector<cv::Point2f>::const_iterator iter1 = points.begin();
    int icols = int(colCount());
    int irows = int(rowCount());
    int count=0;
    for(int row=0;row<irows;++row)
    {
        for(int col=0;col<icols;++col,++iter1)
        {
            if(!H.empty() && iter1->x != iter1->x)    // NaN check
            {
                // draw search ellipse
                Ellipse ellipse = estimateSearchArea(H,row,col,0.4F);
                ellipse.draw(image,cv::Scalar::all(200));
            }
            else
            {
                cv::circle(image,*iter1,4,cv::Scalar(count*20,count*20,count*20,255),-1);
                ++count;
            }
        }
    }

    // draw field colors
    for(int row=0;row<irows-1;++row)
    {
        for(int col=0;col<icols-1;++col)
        {
            const Cell *cell = getCell(row,col);
            cv::Point2f center = cell->getCenter();
            int size = 4;
            if(row==0&&col==0)
                size=8;
            if(row==0&&col==1)
                size=7;

            if(cell->marker)
            {
                if(cell->black)
                    cv::circle(image,center,2,cv::Scalar::all(0),-1);
                else
                {
                    cv::circle(image,center,2,cv::Scalar::all(255),-1);
                    // draw coordinate
                    if(col+1 < icols)
                    {
                        const Cell *cell2 = getCell(row,col+1);
                        cv::Point2f center2 = cell2->getCenter();
                        cv::line(image,center,center2,cv::Scalar::all(127),2);
                    }
                    if(row+1 < irows)
                    {
                        const Cell *cell2 = getCell(row+1,col);
                        cv::Point2f center2 = cell2->getCenter();
                        cv::line(image,center,center2,cv::Scalar::all(127),2);
                    }
                }
            }
            else
            {
                if(cell->black)
                    cv::circle(image,center,size,cv::Scalar::all(255),-1);
                else
                    cv::circle(image,center,size,cv::Scalar(0,0,10,255),-1);
            }
        }
    }

    out.create(image.rows,image.cols,image.type());
    image.copyTo(out.getMat());
}

bool Chessboard::Board::estimatePose(const cv::Size2f &real_size,cv::InputArray _K,cv::OutputArray rvec,cv::OutputArray tvec)const
{
    cv::Mat K = _K.getMat();
    CV_CheckTypeEQ(K.type(), CV_64FC1, "wrong K type");
    CV_CheckEQ(K.size(), Size(3, 3), "wrong K size");
    if(isEmpty())
        return false;

    int icols = int(colCount());
    int irows = int(rowCount());
    float field_width = real_size.width/(icols+1);
    float field_height= real_size.height/(irows+1);
    // the center of the board is placed at (0,0,1)
    int offset_x = int(-(icols-1)*field_width*0.5F);
    int offset_y = int(-(irows-1)*field_width*0.5F);

    std::vector<cv::Point2f> image_points;
    std::vector<cv::Point3f> object_points;
    std::vector<cv::Point2f> corners_temp = getCorners(true);
    std::vector<cv::Point2f>::const_iterator iter = corners_temp.begin();
    for(int row = 0;row < irows;++row)
    {
        for(int col= 0;col<icols;++col,++iter)
        {
            if(iter == corners_temp.end())
                CV_Error(Error::StsInternal,"internal error");
            if(iter->x != iter->x)      // NaN check
                continue;
            image_points.push_back(*iter);
            object_points.push_back(cv::Point3f(field_width*col-offset_x,field_height*row-offset_y,1.0));
        }
    }
    return cv::solvePnP(object_points,image_points,K,cv::Mat(),rvec,tvec);//,cv::SOLVEPNP_P3P);
}

float Chessboard::Board::getBlackAngle()const
{
    return black_angle;
}

float Chessboard::Board::getWhiteAngle()const
{
    return white_angle;
}

void Chessboard::Board::swap(Chessboard::Board &other)
{
    corners.swap(other.corners);
    cells.swap(other.cells);
    std::swap(rows,other.rows);
    std::swap(cols,other.cols);
    std::swap(top_left,other.top_left);
    std::swap(white_angle,other.white_angle);
    std::swap(black_angle,other.black_angle);
}

Chessboard::Board& Chessboard::Board::operator=(const Chessboard::Board &other)
{
    if(this == &other)
        return *this;
    clear();
    rows = other.rows;
    cols = other.cols;
    white_angle = other.white_angle;
    black_angle = other.black_angle;
    cells.reserve(other.cells.size());
    corners.reserve(other.corners.size());

    //copy all points and generate mapping
    std::map<cv::Point2f*,cv::Point2f*> point_point_mapping;
    point_point_mapping[NULL] = NULL;
    std::vector<cv::Point2f*>::const_iterator iter = other.corners.begin();
    for(;iter != other.corners.end();++iter)
    {
        cv::Point2f *pt = new cv::Point2f(**iter);
        point_point_mapping[*iter] = pt;
        corners.push_back(pt);
    }

    //copy all cells using mapping
    std::map<Cell*,Cell*> cell_cell_mapping;
    std::vector<Cell*>::const_iterator iter2 = other.cells.begin();
    for(;iter2 != other.cells.end();++iter2)
    {
        Cell *cell = new Cell;
        cell->top_left = point_point_mapping[(*iter2)->top_left];
        cell->top_right= point_point_mapping[(*iter2)->top_right];
        cell->bottom_right= point_point_mapping[(*iter2)->bottom_right];
        cell->bottom_left = point_point_mapping[(*iter2)->bottom_left];
        cell->black = (*iter2)->black;
        cell->marker = (*iter2)->marker;
        cell_cell_mapping[*iter2] = cell;
        cells.push_back(cell);
    }

    //set cell connections using mapping
    cell_cell_mapping[NULL] = NULL;
    iter2 = other.cells.begin();
    std::vector<Cell*>::iterator iter3 = cells.begin();
    for(;iter2 != other.cells.end();++iter2,++iter3)
    {
        (*iter3)->left = cell_cell_mapping[(*iter2)->left];
        (*iter3)->top = cell_cell_mapping[(*iter2)->top];
        (*iter3)->right = cell_cell_mapping[(*iter2)->right];
        (*iter3)->bottom= cell_cell_mapping[(*iter2)->bottom];
    }
    top_left = cell_cell_mapping[other.top_left];
    return *this;
}

bool Chessboard::Board::normalizeMarkerOrientation()
{
    // use row by row fashion as cells must not be arranged correctly
    Cell *pcell = NULL;
    int trows = int(rowCount());
    int tcols = int(colCount());
    for(int row=0;row != trows && !pcell;++row)
    {
        for(int col=0;col != tcols;++col)
        {
            Cell* current_cell = getCell(row,col);
            if(!current_cell->marker || !current_cell->right || !current_cell->right->marker)
                continue;

            if(current_cell->right->top && current_cell->right->top->marker)
            {
                rotateLeft();
                rotateLeft();
                pcell = current_cell->right;
                break;
            }
            if(current_cell->right->bottom && current_cell->right->bottom->marker)
            {
                rotateLeft();
                pcell = current_cell->right;
                break;
            }
            if(current_cell->top && current_cell->top->marker)
            {
                rotateRight();
                pcell = current_cell;
                break;
            }
            if(current_cell->bottom && current_cell->bottom->marker)
            {
                // correct orientation
                pcell = current_cell;
                break;
            }
        }
    }
    if(pcell)
    {
        //check for ambiguity
        if(rowCount()-pcell->bottom->getRow() > 2)
        {
            CV_LOG_DEBUG(NULL, "FIX board " << pcell->bottom->getRow() << " " << rowCount());
            flipVertical();
            rotateRight();
        }
        return true;
    }
    return false;
}

void Chessboard::Board::normalizeOrientation(bool bblack)
{
    // fix ordering
    cv::Point2f y = getCorner(0,1)-getCorner(2,1);
    cv::Point2f x = getCorner(1,2)-getCorner(1,0);
    cv::Point3f y3d(y.x,y.y,0);
    cv::Point3f x3d(x.x,x.y,0);
    if(x3d.cross(y3d).z > 0)
        flipHorizontal();

    //normalize orientation so that first element is black or white
    const Cell* cell = getCell(0,0);
    if(cell->black != bblack && colCount()%2 != 0)
        rotateLeft();
    else if(cell->black != bblack && rowCount()%2 != 0)
    {
        rotateLeft();
        rotateLeft();
    }

    //find closest point to top left image corner
    //in case of symmetric checkerboard
    if(colCount() == rowCount())
    {
        PointIter iter_top_right(top_left,TOP_RIGHT);
        while(iter_top_right.right());
        PointIter iter_bottom_right(iter_top_right);
        while(iter_bottom_right.bottom());
        PointIter iter_bottom_left(top_left,BOTTOM_LEFT);
        while(iter_bottom_left.bottom());
        // check if one of the cell is empty and do not normalize if so
        if(top_left->empty() || iter_top_right.getCell()->empty() ||
                iter_bottom_left.getCell()->empty() || iter_bottom_right.getCell()->empty())
            return;

        float d1 = top_left->top_left->dot(*top_left->top_left);
        float d2 = (*iter_top_right)->dot(*(*iter_top_right));
        float d3 = (*iter_bottom_left)->dot(*(*iter_bottom_left));
        float d4 = (*iter_bottom_right)->dot(*(*iter_bottom_right));
        if(d2 <= d1 && d2 <= d3 && d2 <= d4) // top left is top right
            rotateLeft();
        else if(d3 <= d1 && d3 <= d2 && d3 <= d4) // top left is bottom left
            rotateRight();
        else if(d4 <= d1 && d4 <= d2 && d4 <= d3)      // top left is bottom right
        {
            rotateLeft();
            rotateLeft();
        }
    }
}

void Chessboard::Board::rotateRight()
{
    PointIter p_iter(top_left,BOTTOM_LEFT);
    while(p_iter.bottom());

    std::vector<Cell*>::iterator iter = cells.begin();
    for(;iter != cells.end();++iter)
    {
        Cell *temp = (*iter)->bottom;
        (*iter)->bottom = (*iter)->right;
        (*iter)->right= (*iter)->top;
        (*iter)->top= (*iter)->left;
        (*iter)->left = temp;

        cv::Point2f *ptemp = (*iter)->bottom_left;
        (*iter)->bottom_left= (*iter)->bottom_right;
        (*iter)->bottom_right= (*iter)->top_right;
        (*iter)->top_right= (*iter)->top_left;
        (*iter)->top_left= ptemp;
    }
    int temp = rows;
    rows = cols;
    cols = temp;
    top_left = p_iter.getCell();
}


void Chessboard::Board::rotateLeft()
{
    PointIter p_iter(top_left,TOP_RIGHT);
    while(p_iter.right());

    std::vector<Cell*>::iterator iter = cells.begin();
    for(;iter != cells.end();++iter)
    {
        Cell *temp = (*iter)->top;
        (*iter)->top = (*iter)->right;
        (*iter)->right= (*iter)->bottom;
        (*iter)->bottom= (*iter)->left;
        (*iter)->left = temp;

        cv::Point2f *ptemp = (*iter)->top_left;
        (*iter)->top_left = (*iter)->top_right;
        (*iter)->top_right= (*iter)->bottom_right;
        (*iter)->bottom_right = (*iter)->bottom_left;
        (*iter)->bottom_left = ptemp;
    }
    int temp = rows;
    rows = cols;
    cols = temp;
    top_left = p_iter.getCell();
}

void Chessboard::Board::flipHorizontal()
{
    PointIter p_iter(top_left,TOP_RIGHT);
    while(p_iter.right());

    std::vector<Cell*>::iterator iter = cells.begin();
    for(;iter != cells.end();++iter)
    {
        Cell *temp = (*iter)->right;
        (*iter)->right= (*iter)->left;
        (*iter)->left = temp;

        cv::Point2f *ptemp = (*iter)->top_left;
        (*iter)->top_left = (*iter)->top_right;
        (*iter)->top_right = ptemp;

        ptemp = (*iter)->bottom_left;
        (*iter)->bottom_left = (*iter)->bottom_right;
        (*iter)->bottom_right = ptemp;
    }
    top_left = p_iter.getCell();
}

void Chessboard::Board::flipVertical()
{
    PointIter p_iter(top_left,BOTTOM_LEFT);
    while(p_iter.bottom());

    std::vector<Cell*>::iterator iter = cells.begin();
    for(;iter != cells.end();++iter)
    {
        Cell *temp = (*iter)->top;
        (*iter)->top= (*iter)->bottom;
        (*iter)->bottom = temp;

        cv::Point2f *ptemp = (*iter)->top_left;
        (*iter)->top_left = (*iter)->bottom_left;
        (*iter)->bottom_left = ptemp;

        ptemp = (*iter)->top_right;
        (*iter)->top_right = (*iter)->bottom_right;
        (*iter)->bottom_right = ptemp;
    }
    top_left = p_iter.getCell();
}

// returns the best found score
// if NaN is returned for a point no point at all was found
// if 0 is returned the point lies outside of the ellipse
float Chessboard::Board::findMaxPoint(cv::flann::Index &index,const cv::Mat &data,const Ellipse &ellipse,float white_angle,float black_angle,cv::Point2f &point)
{
    // flann data type enriched with angles (third column)
    CV_CheckType(data.type(), CV_32FC1, "type of flann data is not supported");
    CV_CheckEQ(data.cols, 4, "4-cols flann data is expected");

    std::vector<float> query,dists;
    std::vector<int> indices;
    query.resize(2);
    point = ellipse.getCenter();
    query[0] = point.x;
    query[1] = point.y;
    index.knnSearch(query,indices,dists,4,cv::flann::SearchParams(64));
    std::vector<int>::const_iterator iter = indices.begin();
    float best_score = -std::numeric_limits<float>::max();
    point.x = std::numeric_limits<float>::quiet_NaN();
    point.y = std::numeric_limits<float>::quiet_NaN();
    for(;iter != indices.end();++iter)
    {
        const float *val = data.ptr<float>(*iter);
        const float &response = *(val+3);
        if(response < best_score)
            continue;
        const float &a0 = *(val+2);
        float a1 = std::fabs(a0-white_angle);
        float a2 = std::fabs(a0-black_angle);
        if(a1 > CV_PI*0.5)
            a1 = std::fabs(float(a1-CV_PI));
        if(a2> CV_PI*0.5)
            a2 = std::fabs(float(a2-CV_PI));
        if(a1  < MAX_ANGLE || a2 < MAX_ANGLE )
        {
            cv::Point2f pt(val[0], val[1]);
            if(point.x != point.x)       // NaN check
                point = pt;
            if(best_score < response && ellipse.contains(pt))
            {
                best_score = response;
                point = pt;
            }
        }
    }
    if(best_score == -std::numeric_limits<float>::max())
        return 0;
    else
        return best_score;
}

void Chessboard::Board::clear()
{
    top_left = NULL; rows = 0; cols = 0;
    std::vector<Cell*>::iterator iter = cells.begin();
    for(;iter != cells.end();++iter)
        delete *iter;
    cells.clear();
    std::vector<cv::Point2f*>::iterator iter2 = corners.begin();
    for(;iter2 != corners.end();++iter2)
        delete *iter2;
    corners.clear();
}

// p0 p1 p2
// p3 p4 p5
// p6 p7 p8
bool Chessboard::Board::init(const std::vector<cv::Point2f> points)
{
    clear();
    if(points.size() != 9)
        CV_Error(Error::StsBadArg,"exact nine points are expected to initialize the board");

    // generate cells
    corners.resize(9);
    for(int i=0;i < 9;++i)
        corners[i] = new cv::Point2f(points[i]);
    cells.resize(4);
    for(int i=0;i<4;++i)
        cells[i] = new Cell();

    //cell 0
    cells[0]->top_left = corners[0];
    cells[0]->top_right = corners[1];
    cells[0]->bottom_right = corners[4];
    cells[0]->bottom_left = corners[3];
    cells[0]->right = cells[1];
    cells[0]->bottom = cells[2];

    //cell 1
    cells[1]->top_left = corners[1];
    cells[1]->top_right = corners[2];
    cells[1]->bottom_right = corners[5];
    cells[1]->bottom_left = corners[4];
    cells[1]->left = cells[0];
    cells[1]->bottom = cells[3];

    //cell 2
    cells[2]->top_left = corners[3];
    cells[2]->top_right = corners[4];
    cells[2]->bottom_right = corners[7];
    cells[2]->bottom_left = corners[6];
    cells[2]->top = cells[0];
    cells[2]->right = cells[3];

    //cell 3
    cells[3]->top_left = corners[4];
    cells[3]->top_right = corners[5];
    cells[3]->bottom_right = corners[8];
    cells[3]->bottom_left = corners[7];
    cells[3]->top = cells[1];
    cells[3]->left= cells[2];

    top_left = cells.front();
    rows = 3;
    cols = 3;

    // set initial cell colors
    Point2f pt1 = *(cells[0]->top_right)-*(cells[0]->bottom_left);
    pt1 /= cv::norm(pt1);
    cv::Point2f pt2(cos(white_angle),-sin(white_angle));
    cv::Point2f pt3(cos(black_angle),-sin(black_angle));
    if(fabs(pt1.dot(pt2)) < fabs(pt1.dot(pt3)))
    {
        cells[0]->black = false;
        cells[1]->black = true;
        cells[2]->black = true;
        cells[3]->black = false;
    }
    else
    {
        cells[0]->black = true;
        cells[1]->black = false;
        cells[2]->black = false;
        cells[3]->black = true;
    }
    return true;
}

//TODO magic number
bool Chessboard::Board::estimatePoint(const cv::Point2f &p0,const cv::Point2f &p1,const cv::Point2f &p2, cv::Point2f &p3)
{
    // use cross ration to find new point
    if(p0 == p1 || p0 == p2 || p1 == p2)
        return false;
    cv::Point2f p01 = p1-p0;
    cv::Point2f p12 = p2-p1;
    float a = float(cv::norm(p01));
    float b = float(cv::norm(p12));
    float t = (0.75F*a-0.25F*b);
    if(t <= 0)
        return false;
    float c = 0.25F*b*(a+b)/t;
    if(c < 0.1F)
        return false;
    p01 = p01/a;
    p12 = p12/b;
    // check angle between p01 and p12 < 25°
    if(p01.dot(p12) < 0.9)
        return false;
    // calc mean
    // p12 = (p01+p12)*0.5;
    // p3 = p2+p12*c;
    p3 = p2+p12*c;

    // compensate radial distortion by fitting polynom
    std::vector<double> x,y;
    x.resize(3,0); y.resize(3,0);
    x[1] = b;
    x[2] = b+a;
    y[2] = calcSignedDistance(-p12,p2,p0);
    cv::Mat dst;
    polyfit(cv::Mat(x),cv::Mat(y),dst,2);
    double d = dst.at<double>(0)-dst.at<double>(1)*c+dst.at<double>(2)*c*c;
    cv::Vec3f v1(p12.x,p12.y,0);
    cv::Vec3f v2(0,0,1);
    cv::Vec3f v3 = v1.cross(v2);
    cv::Point2f n2(v3[0],v3[1]);
    p3 += d*n2;
    return true;
}

bool Chessboard::Board::estimatePoint(const cv::Point2f &p0,const cv::Point2f &p1,const cv::Point2f &p2, const cv::Point2f &p3, cv::Point2f &p4)
{
    // use 1D homography to find fith point minimizing square error
    if(p0 == p1 || p0 == p2 || p0 == p3 || p1 == p2 || p1 == p3 || p2 == p3 )
        return false;
    static const cv::Mat src = (cv::Mat_<double>(1,4) << 0,10,20,30);
    cv::Point2f p01 = p1-p0;
    cv::Point2f p02 = p2-p0;
    cv::Point2f p03 = p3-p0;
    float a = float(cv::norm(p01));
    float b = float(cv::norm(p02));
    float c = float(cv::norm(p03));
    cv::Mat dst = (cv::Mat_<double>(1,4) << 0,a,b,c);
    cv::Mat h = findHomography1D(src,dst);
    float d = float((h.at<double>(0,0)*40+h.at<double>(0,1))/(h.at<double>(1,0)*40+h.at<double>(1,1)));
    cv::Point2f p12 = p2-p1;
    cv::Point2f p23 = p3-p2;
    p01 = p01/a;
    p12 = p12/cv::norm(p12);
    p23 = p23/cv::norm(p23);
    p4 = p3+(d-c)*p23;

    // compensate radial distortion by fitting polynom
    std::vector<double> x,y;
    x.resize(4,0); y.resize(4,0);
    x[1] = c-b;
    x[2] = c-a;
    x[3] = c;
    y[2] = calcSignedDistance(-p23,p3,p1);
    y[3] = calcSignedDistance(-p23,p3,p0);
    polyfit(cv::Mat(x),cv::Mat(y),dst,2);
    d = d-c;
    double e = dst.at<double>(0)-dst.at<double>(1)*fabs(d)+dst.at<double>(2)*d*d;
    cv::Vec3f v1(p23.x,p23.y,0);
    cv::Vec3f v2(0,0,1);
    cv::Vec3f v3 = v1.cross(v2);
    cv::Point2f n2(v3[0],v3[1]);
    p4 += e*n2;
    return true;
}

// H is describing the transformation from dummy to reality
Ellipse Chessboard::Board::estimateSearchArea(cv::Mat _H,int row, int col,float p,int field_size)
{
    cv::Matx31d point1,point2,center;
    center(0) = (1+col)*field_size;
    center(1) = (1+row)*field_size;
    center(2) = 1.0;
    point1(0) = center(0)-p*field_size;
    point1(1) = center(1);
    point1(2) = center(2);
    point2(0) = center(0);
    point2(1) = center(1)-p*field_size;
    point2(2) = center(2);

    cv::Matx33d H(_H);
    point1 = H*point1;
    point2 = H*point2;
    center = H*center;
    cv::Point2f pt(float(center(0)/center(2)),float(center(1)/center(2)));
    cv::Point2f pt1(float(point1(0)/point1(2)),float(point1(1)/point1(2)));
    cv::Point2f pt2(float(point2(0)/point2(2)),float(point2(1)/point2(2)));

    cv::Point2f p01(pt1-pt);
    cv::Point2f p02(pt2-pt);
    float norm1 = float(cv::norm(p01));
    float norm2 = float(cv::norm(p02));
    float angle = float(acos(p01.dot(p02)/norm1/norm2));
    cv::Size2f axes(norm1,norm2);
    return Ellipse(pt,axes,angle);
}

bool Chessboard::Board::estimateSearchArea(const cv::Point2f &p1,const cv::Point2f &p2,const cv::Point2f &p3,float p,Ellipse &ellipse,const cv::Point2f *p0)
{
    cv::Point2f p4,n;
    if(p0)
    {
        // use 1D homography
        if(!estimatePoint(*p0,p1,p2,p3,p4))
            return false;
        n = p4-*p0;
    }
    else
    {
        // use cross ratio
        if(!estimatePoint(p1,p2,p3,p4))
            return false;
        n = p4-p1;
    }
    float norm = float(cv::norm(n));
    n = n/norm;
    float angle = acos(n.x);
    if(n.y > 0)
        angle = float(2.0F*CV_PI-angle);
    n = p4-p3;
    norm = float(cv::norm(n));
    double delta = std::max(3.0F,p*norm);
    ellipse = Ellipse(p4,cv::Size(int(delta),int(std::max(2.0,delta*ELLIPSE_WIDTH))),angle);
    return true;
}

bool Chessboard::Board::checkRowColumn(const std::vector<cv::Point2f> &points)
{
    if(points.size() < 4)
    {
        if(points.size() == 3)
            return true;
        else
            return false;
    }
    std::vector<cv::Point2f>::const_iterator iter = points.begin();
    std::vector<cv::Point2f>::const_iterator iter2 = iter+1;
    std::vector<cv::Point2f>::const_iterator iter3 = iter2+1;
    std::vector<cv::Point2f>::const_iterator iter4 = iter3+1;
    Ellipse ellipse;
    if(!estimateSearchArea(*iter4,*iter3,*iter2,CORNERS_SEARCH*3,ellipse))
        return false;
    if(!ellipse.contains(*iter))
        return false;

    std::vector<cv::Point2f>::const_iterator iter5 = iter4+1;
    for(;iter5 != points.end();++iter5)
    {
        if(!estimateSearchArea(*iter2,*iter3,*iter4,CORNERS_SEARCH,ellipse,&(*iter)))
            return false;
        if(!ellipse.contains(*iter5))
            return false;
        iter = iter2;
        iter2 = iter3;
        iter3 = iter4;
        iter4 = iter5;
    }
    return true;
}

cv::Point2f &Chessboard::Board::getCorner(int _row,int _col)
{
    int _rows = int(rowCount());
    int _cols = int(colCount());
    if(_row >= _rows || _col >= _cols)
        CV_Error(Error::StsBadArg,"out of bound");
    if(_row == 0)
    {
        PointIter iter(top_left,TOP_LEFT);
        int count = 0;
        do
        {
            if(count == _col)
                return *(*iter);
            ++count;
        }while(iter.right());
    }
    else
    {
        Cell *row_start = top_left;
        int count = 1;
        do
        {
            if(count == _row)
            {
                PointIter iter(row_start,BOTTOM_LEFT);
                int count2 = 0;
                do
                {
                    if(count2 == _col)
                        return *(*iter);
                    ++count2;
                }while(iter.right());
            }
            ++count;
            row_start = row_start->bottom;
        }while(_row);
    }
    CV_Error(Error::StsInternal,"cannot find corner");
    // return *top_left->top_left; // never reached
}

bool Chessboard::Board::isCellBlack(int row,int col)const
{
    return getCell(row,col)->black;
}

bool Chessboard::Board::hasCellMarker(int row,int col)
{
    return getCell(row,col)->marker;
}

int Chessboard::Board::detectMarkers(cv::InputArray image)
{
    cv::Mat img = image.getMat();
    CV_CheckTypeEQ(img.type(), CV_8UC1, "Unsupported source type");
    if(img.empty())
        CV_Error(Error::StsBadArg,"image is empty");
    if(isEmpty())
        CV_Error(Error::StsBadArg,"board is is empty");

    // get undistorted board
    cv::Mat board_image = warpImage(image);

    cv::Mat mask = cv::Mat::zeros(DUMMY_FIELD_SIZE,DUMMY_FIELD_SIZE,CV_8UC1);
    cv::circle(mask,cv::Point(DUMMY_FIELD_SIZE/2,DUMMY_FIELD_SIZE/2),DUMMY_FIELD_SIZE/7,cv::Scalar::all(255),-1);
    int signal_size = cv::countNonZero(mask);
    CV_Assert(signal_size > 0);

    cv::Mat mask2 = cv::Mat::zeros(DUMMY_FIELD_SIZE,DUMMY_FIELD_SIZE,CV_8UC1);
    cv::circle(mask2,cv::Point(DUMMY_FIELD_SIZE/2,DUMMY_FIELD_SIZE/2),DUMMY_FIELD_SIZE/2,cv::Scalar::all(255),-1);
    cv::circle(mask2,cv::Point(DUMMY_FIELD_SIZE/2,DUMMY_FIELD_SIZE/2),DUMMY_FIELD_SIZE/5,cv::Scalar::all(0),-1);
    int noise_size = cv::countNonZero(mask2);
    CV_Assert(noise_size > 0);

    std::vector<cv::Point2f> dst,src;
    dst.push_back(cv::Point2f(0.0F,0.0F));
    dst.push_back(cv::Point2f(float(DUMMY_FIELD_SIZE),0.0F));
    dst.push_back(cv::Point2f(float(DUMMY_FIELD_SIZE),float(DUMMY_FIELD_SIZE)));
    dst.push_back(cv::Point2f(0.0F,float(DUMMY_FIELD_SIZE)));
    src.resize(4);

    // check each field
    int icols = int(colCount()-1);
    int irows = int(rowCount()-1);
    int count = 0;
    cv::Mat temp;
    for(int y=1;y<irows;++y)
    {
        for(int x=1;x<icols;++x)
        {
            Cell *cell = getCell(y,x);

            // calculate homography for each field to avoid issues with
            // distorted images
            src[0] = *cell->top_left;
            src[1] = *cell->top_right;
            src[2] = *cell->bottom_right;
            src[3] = *cell->bottom_left;
            cv::Mat H = cv::findHomography(src,dst,cv::LMEDS);
            cv::Mat field;
            cv::warpPerspective(image,field,H,cv::Size(DUMMY_FIELD_SIZE,DUMMY_FIELD_SIZE));

            // calc signal and noise value
            cv::bitwise_and(field,mask,temp);
            double signal = cv::sum(temp)[0]/signal_size;
            cv::bitwise_and(field,mask2,temp);
            double noise= cv::sum(temp)[0]/noise_size;

            // calc refrence value
            Cell *cell2 = getCell(y,abs(x-1));
            src[0] = *cell2->top_left;
            src[1] = *cell2->top_right;
            src[2] = *cell2->bottom_right;
            src[3] = *cell2->bottom_left;
            H = cv::findHomography(src,dst,cv::LMEDS);
            cv::warpPerspective(image,field,H,cv::Size(DUMMY_FIELD_SIZE,DUMMY_FIELD_SIZE));
            cv::bitwise_and(field,mask2,temp);
            double reference = cv::sum(temp)[0]/noise_size;
            // check if marker is present
            if(cell->black)
                cell->marker = signal-noise > (reference-noise)*0.5;
            else
                cell->marker = noise-signal > (noise-reference)*0.5;
            if(cell->marker)
                count++;
            CV_LOG_DEBUG(NULL, "Cell: " << x << "/" << y << " signal " << signal << " noise " << noise << " reference " << reference  << " has marker " << int(cell->marker));
        }
    }
    return count;
}

bool Chessboard::Board::isCellEmpty(int row,int col)
{
    return getCell(row,col)->empty();
}

Chessboard::Board::Cell* Chessboard::Board::getCell(int row,int col)
{
    const Cell *cell = const_cast<const Board*>(this)->getCell(row,col);
    return const_cast<Cell*>(cell);
}

const Chessboard::Board::Cell* Chessboard::Board::getCell(int row,int col)const
{
    if(row > rows-1 || row < 0 || col > cols-1 || col < 0)
        CV_Error(Error::StsBadArg,"out of bound");
    PointIter p_iter(top_left,BOTTOM_RIGHT);
    for(int i=0; i< row; p_iter.bottom(),++i);
    for(int i=0; i< col; p_iter.right(),++i);
    return p_iter.getCell();
}


bool Chessboard::Board::isEmpty()const
{
    return cells.empty();
}

size_t Chessboard::Board::colCount()const
{
    return cols;
}

size_t Chessboard::Board::rowCount()const
{
    return rows;
}

cv::Size Chessboard::Board::getSize()const
{
    return cv::Size(int(colCount()),int(rowCount()));
}

void Chessboard::Board::drawEllipses(const std::vector<Ellipse> &ellipses)
{
    // currently there is no global image find way to store global image
    // without polluting namespace
    if(ellipses.empty())
        return;     //avoid compiler warning
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    cv::Mat img;
    draw(debug_image,img);
    std::vector<Ellipse>::const_iterator iter = ellipses.begin();
    for(;iter != ellipses.end();++iter)
        iter->draw(img);
    cv::imshow("chessboard",img);
    cv::waitKey(-1);
#endif
}


// TODO also delete points
bool Chessboard::Board::shrinkLeft()
{
    if(colCount() < 4)
        return false;

    // unlink cells on the left side and delete them
    top_left = top_left->right;
    PointIter iter(top_left,BOTTOM_RIGHT);
    do
    {
        auto cell = iter.getCell();
        auto citer = std::find(cells.begin(),cells.end(),cell->left);
        delete cell->left;
        cell->left= NULL;
        cells.erase(citer);
    }
    while(iter.bottom());
    --cols;
    return true;
}

bool Chessboard::Board::shrinkRight()
{
    if(colCount() < 4)
        return false;

    // unlink cells on the left side and delete them
    PointIter iter(top_left,BOTTOM_RIGHT);
    while(iter.right());
    iter.left();
    iter.left();
    do
    {
        auto cell = iter.getCell();
        auto citer = std::find(cells.begin(),cells.end(),cell->right);
        delete cell->right;
        cell->right= NULL;
        cells.erase(citer);
    }
    while(iter.bottom());
    --cols;
    return true;
}

bool Chessboard::Board::shrinkTop()
{
    if(rowCount() < 4)
        return false;

    // unlink cells on the left side and delete them
    top_left = top_left->bottom;
    PointIter iter(top_left,BOTTOM_RIGHT);
    do
    {
        auto cell = iter.getCell();
        auto citer = std::find(cells.begin(),cells.end(),cell->top);
        delete cell->top;
        cell->top= NULL;
        cells.erase(citer);
    }
    while(iter.right());
    --rows;
    return true;
}

bool Chessboard::Board::shrinkBottom()
{
    if(rowCount() < 4)
        return false;

    // unlink cells on the left side and delete them
    PointIter iter(top_left,BOTTOM_RIGHT);
    while(iter.bottom());
    iter.top();
    iter.top();
    do
    {
        auto cell = iter.getCell();
        auto citer = std::find(cells.begin(),cells.end(),cell->bottom);
        delete cell->bottom;
        cell->bottom= NULL;
        cells.erase(citer);
    }
    while(iter.right());
    --rows;
    return true;
}

void Chessboard::Board::growLeft()
{
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");
    PointIter iter(top_left,TOP_LEFT);
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.right();
        cv::Point2f *p1 = *iter2;
        iter2.right();
        cv::Point2f *p2 = *iter2;
        if(iter2.right())
            estimatePoint(**iter2,*p2,*p1,*p0,pt);
        else
            estimatePoint(*p2,*p1,*p0,pt);
        points.push_back(pt);
    }
    while(iter.bottom());
    addColumnLeft(points);
}

bool Chessboard::Board::growLeft(const cv::Mat &map,cv::flann::Index &flann_index)
{
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    std::vector<Ellipse> ellipses;
#endif
    if(isEmpty())
        CV_Error(Error::StsInternal,"growLeft: Board is empty");
    PointIter iter(top_left,TOP_LEFT);
    std::vector<cv::Point2f> points;
    int count = 0;
    Ellipse ellipse;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.right();
        cv::Point2f *p1 = *iter2;
        iter2.right();
        cv::Point2f *p2 = *iter2;
        cv::Point2f *p3 = NULL;
        if(iter2.right())
            p3 = *iter2;
        if(!estimateSearchArea(*p2,*p1,*p0,CORNERS_SEARCH,ellipse,p3))
            return false;
        float result = findMaxPoint(flann_index,map,ellipse,white_angle,black_angle,pt);
        if(pt == *p0)
        {
            ++count;
            points.push_back(ellipse.getCenter());
            if(points.back().x < 0 || points.back().y <0)
                return false;
        }
        else if(result != 0)
        {
            points.push_back(pt);
            if(result < 0)
                ++count;
        }
        else
        {
            ++count;
            if(pt.x != pt.x)    // NaN check
                points.push_back(ellipse.getCenter());
            else
                points.push_back(pt);
        }
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
        ellipses.push_back(ellipse);
#endif
    }
    while(iter.bottom());
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    drawEllipses(ellipses);
#endif
    if(points.size()-count <= 2)
        return false;
    if(count > points.size()*0.5 || !checkRowColumn(points))
        return false;
    addColumnLeft(points);
    return true;
}

void Chessboard::Board::growTop()
{
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");
    PointIter iter(top_left,TOP_LEFT);
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.bottom();
        cv::Point2f *p1 = *iter2;
        iter2.bottom();
        cv::Point2f *p2 = *iter2;
        if(iter2.bottom())
            estimatePoint(**iter2,*p2,*p1,*p0,pt);
        else
            estimatePoint(*p2,*p1,*p0,pt);
        points.push_back(pt);
    }
    while(iter.right());
    addRowTop(points);
}

bool Chessboard::Board::growTop(const cv::Mat &map,cv::flann::Index &flann_index)
{
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    std::vector<Ellipse> ellipses;
#endif
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");

    PointIter iter(top_left,TOP_LEFT);
    std::vector<cv::Point2f> points;
    int count = 0;
    Ellipse ellipse;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.bottom();
        cv::Point2f *p1 = *iter2;
        iter2.bottom();
        cv::Point2f *p2 = *iter2;
        cv::Point2f *p3 = NULL;
        if(iter2.bottom())
            p3 = *iter2;
        if(!estimateSearchArea(*p2,*p1,*p0,CORNERS_SEARCH,ellipse,p3))
            return false;
        float result = findMaxPoint(flann_index,map,ellipse,white_angle,black_angle,pt);
        if(pt == *p0)
        {
            ++count;
            points.push_back(ellipse.getCenter());
            if(points.back().x < 0 || points.back().y <0)
                return false;
        }
        else if(result != 0)
        {
            points.push_back(pt);
            if(result < 0)
                ++count;
        }
        else
        {
            ++count;
            if(pt.x != pt.x)    // NaN check
                points.push_back(ellipse.getCenter());
            else
                points.push_back(pt);
        }
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
        ellipses.push_back(ellipse);
#endif
    }
    while(iter.right());
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    drawEllipses(ellipses);
#endif
    if(count > points.size()*0.5 || !checkRowColumn(points))
        return false;
    addRowTop(points);
    return true;
}

void Chessboard::Board::growRight()
{
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");
    PointIter iter(top_left,TOP_RIGHT);
    while(iter.right());
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.left();
        cv::Point2f *p1 = *iter2;
        iter2.left();
        cv::Point2f *p2 = *iter2;
        if(iter2.left())
            estimatePoint(**iter2,*p2,*p1,*p0,pt);
        else
            estimatePoint(*p2,*p1,*p0,pt);
        points.push_back(pt);
    }
    while(iter.bottom());
    addColumnRight(points);
}

bool Chessboard::Board::growRight(const cv::Mat &map,cv::flann::Index &flann_index)
{
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    std::vector<Ellipse> ellipses;
#endif
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");

    PointIter iter(top_left,TOP_RIGHT);
    while(iter.right());
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    Ellipse ellipse;
    int count = 0;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.left();
        cv::Point2f *p1 = *iter2;
        iter2.left();
        cv::Point2f *p2 = *iter2;
        cv::Point2f *p3 = NULL;
        if(iter2.left())
            p3 = *iter2;
        if(!estimateSearchArea(*p2,*p1,*p0,CORNERS_SEARCH,ellipse,p3))
            return false;
        float result = findMaxPoint(flann_index,map,ellipse,white_angle,black_angle,pt);
        if(pt == *p0)
        {
            ++count;
            points.push_back(ellipse.getCenter());
            if(points.back().x < 0 || points.back().y <0)
                return false;
        }
        else if(result != 0)
        {
            points.push_back(pt);
            if(result < 0)
                ++count;
        }
        else
        {
            ++count;
            if(pt.x != pt.x)     // NaN check
                points.push_back(ellipse.getCenter());
            else
                points.push_back(pt);
        }
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
        ellipses.push_back(ellipse);
#endif
    }
    while(iter.bottom());
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    drawEllipses(ellipses);
#endif
    if(count > points.size()*0.5 || !checkRowColumn(points))
        return false;
    addColumnRight(points);
    return true;
}

void Chessboard::Board::growBottom()
{
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");

    PointIter iter(top_left,BOTTOM_LEFT);
    while(iter.bottom());
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.top();
        cv::Point2f *p1 = *iter2;
        iter2.top();
        cv::Point2f *p2 = *iter2;
        if(iter2.top())
            estimatePoint(**iter2,*p2,*p1,*p0,pt);
        else
            estimatePoint(*p2,*p1,*p0,pt);
        points.push_back(pt);
    }
    while(iter.right());
    addRowBottom(points);
}

bool Chessboard::Board::growBottom(const cv::Mat &map,cv::flann::Index &flann_index)
{
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    std::vector<Ellipse> ellipses;
#endif
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");

    PointIter iter(top_left,BOTTOM_LEFT);
    while(iter.bottom());
    std::vector<cv::Point2f> points;
    cv::Point2f pt;
    Ellipse ellipse;
    int count = 0;
    do
    {
        PointIter iter2(iter);
        cv::Point2f *p0 = *iter2;
        iter2.top();
        cv::Point2f *p1 = *iter2;
        iter2.top();
        cv::Point2f *p2 = *iter2;
        cv::Point2f *p3 = NULL;
        if(iter2.top())
            p3 = *iter2;
        if(!estimateSearchArea(*p2,*p1,*p0,CORNERS_SEARCH,ellipse,p3))
            return false;
        float result = findMaxPoint(flann_index,map,ellipse,white_angle,black_angle,pt);
        if(pt == *p0)
        {
            ++count;
            points.push_back(ellipse.getCenter());
            if(points.back().x < 0 || points.back().y <0)
                return false;
        }
        else if(result != 0)
        {
            points.push_back(pt);
            if(result < 0)
                ++count;
        }
        else
        {
            ++count;
            if(pt.x != pt.x)     // NaN check
                points.push_back(ellipse.getCenter());
            else
                points.push_back(pt);
        }
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
        ellipses.push_back(ellipse);
#endif
    }
    while(iter.right());
#ifdef CV_DETECTORS_CHESSBOARD_DEBUG
    drawEllipses(ellipses);
#endif
    if(count > points.size()*0.5 || !checkRowColumn(points))
        return false;
    addRowBottom(points);
    return true;
}

void Chessboard::Board::addColumnLeft(const std::vector<cv::Point2f> &points)
{
    if(points.empty() || points.size() != rowCount())
        CV_Error(Error::StsBadArg,"wrong number of points");

    int offset = int(cells.size());
    cells.resize(offset+points.size()-1);
    for(int i = offset;i < (int) cells.size();++i)
        cells[i] = new Cell();
    corners.push_back(new cv::Point2f(points.front()));

    Cell *cell = top_left;
    std::vector<cv::Point2f>::const_iterator iter = points.begin()+1;
    for(int pos=offset;iter != points.end();++iter,cell = cell->bottom,++pos)
    {
        cell->left = cells[pos];
        cells[pos]->black = !cell->black;
        if(pos != offset)
            cells[pos]->top = cells[pos-1];
        cells[pos]->right = cell;
        if(pos +1 < (int)cells.size())
            cells[pos]->bottom= cells[pos+1];
        cells[pos]->top_left = corners.back();
        corners.push_back(new cv::Point2f(*iter));
        cells[pos]->bottom_left = corners.back();
        cells[pos]->top_right=cell->top_left;
        cells[pos]->bottom_right=cell->bottom_left;
    }
    top_left = cells[offset];
    ++cols;
}

void Chessboard::Board::addRowTop(const std::vector<cv::Point2f> &points)
{
    if(points.empty() || points.size() != colCount())
        CV_Error(Error::StsBadArg,"wrong number of points");

    int offset = int(cells.size());
    cells.resize(offset+points.size()-1);
    for(int i = offset;i < (int) cells.size();++i)
        cells[i] = new Cell();
    corners.push_back(new cv::Point2f(points.front()));

    Cell *cell = top_left;
    std::vector<cv::Point2f>::const_iterator iter = points.begin()+1;
    for(int pos=offset;iter != points.end();++iter,cell = cell->right,++pos)
    {
        cell->top = cells[pos];
        cells[pos]->black = !cell->black;
        if(pos != offset)
            cells[pos]->left= cells[pos-1];
        cells[pos]->bottom= cell;
        if(pos +1 <(int) cells.size())
            cells[pos]->right= cells[pos+1];

        cells[pos]->top_left = corners.back();
        corners.push_back(new cv::Point2f(*iter));
        cells[pos]->top_right = corners.back();
        cells[pos]->bottom_left = cell->top_left;
        cells[pos]->bottom_right = cell->top_right;
    }
    top_left = cells[offset];
    ++rows;
}

void Chessboard::Board::addColumnRight(const std::vector<cv::Point2f> &points)
{
    if(points.empty() || points.size() != rowCount())
        CV_Error(Error::StsBadArg,"wrong number of points");

    int offset = int(cells.size());
    cells.resize(offset+points.size()-1);
    for(int i = offset;i < (int) cells.size();++i)
        cells[i] = new Cell();
    corners.push_back(new cv::Point2f(points.front()));

    Cell *cell = top_left;
    for(;cell->right;cell = cell->right);
    std::vector<cv::Point2f>::const_iterator iter = points.begin()+1;
    for(int pos=offset;iter != points.end();++iter,cell = cell->bottom,++pos)
    {
        cell->right = cells[pos];
        cells[pos]->black = !cell->black;
        if(pos != offset)
            cells[pos]->top= cells[pos-1];
        cells[pos]->left = cell;
        if(pos +1 <(int) cells.size())
            cells[pos]->bottom= cells[pos+1];

        cells[pos]->top_right = corners.back();
        corners.push_back(new cv::Point2f(*iter));
        cells[pos]->bottom_right = corners.back();
        cells[pos]->top_left =cell->top_right;
        cells[pos]->bottom_left =cell->bottom_right;
    }
    ++cols;
}

void Chessboard::Board::addRowBottom(const std::vector<cv::Point2f> &points)
{
    if(points.empty() || points.size() != colCount())
        CV_Error(Error::StsBadArg,"wrong number of points");

    int offset = int(cells.size());
    cells.resize(offset+points.size()-1);
    for(int i = offset;i < (int) cells.size();++i)
        cells[i] = new Cell();
    corners.push_back(new cv::Point2f(points.front()));

    Cell *cell = top_left;
    for(;cell->bottom;cell = cell->bottom);
    std::vector<cv::Point2f>::const_iterator iter = points.begin()+1;
    for(int pos=offset;iter != points.end();++iter,cell = cell->right,++pos)
    {
        cell->bottom = cells[pos];
        cells[pos]->black = !cell->black;
        if(pos != offset)
            cells[pos]->left = cells[pos-1];
        cells[pos]->top = cell;
        if(pos +1 < (int)cells.size())
            cells[pos]->right= cells[pos+1];

        cells[pos]->bottom_left = corners.back();
        corners.push_back(new cv::Point2f(*iter));
        cells[pos]->bottom_right = corners.back();
        cells[pos]->top_left = cell->bottom_left;
        cells[pos]->top_right = cell->bottom_right;
    }
    ++rows;
}

bool Chessboard::Board::checkUnique()const
{
    std::vector<cv::Point2f> points = getCorners(false);
    std::vector<cv::Point2f>::const_iterator iter = points.begin();
    for(;iter != points.end();++iter)
    {
        std::vector<cv::Point2f>::const_iterator iter2 = iter+1;
        for(;iter2 != points.end();++iter2)
        {
            if(*iter == *iter2)
                return false;
        }
    }
    return true;
}

int Chessboard::Board::validateCorners(const cv::Mat &data,cv::flann::Index &flann_index,const cv::Mat &h,float min_response)
{
    // TODO check input
    if(isEmpty() || h.empty())
        return 0;
    int count = 0; int icol = 0;
    // first row
    PointIter iter(top_left,TOP_LEFT);
    cv::Point2f point;
    do
    {
        if((*iter)->x == (*iter)->x)
            ++count;
        else
        {
            Ellipse ellipse = estimateSearchArea(h,0,icol,0.4F);
            float result = findMaxPoint(flann_index,data,ellipse,white_angle,black_angle,point);
            if(fabs(result) >= min_response)
            {
                ++count;
                **iter = point;
            }
        }
        ++icol;
    }while(iter.right());

    // all other rows
    int irow = 1;
    Cell *row = top_left;
    do
    {
        PointIter iter2(row,BOTTOM_LEFT);
        icol = 0;
        do
        {
            if((*iter2)->x == (*iter2)->x)
                ++count;
            else
            {
                Ellipse ellipse = estimateSearchArea(h,irow,icol,0.4F);
                if(min_response <= findMaxPoint(flann_index,data,ellipse,white_angle,black_angle,point))
                {
                    ++count;
                    **iter2 = point;
                }
            }
            ++icol;
        }while(iter2.right());
        row = row->bottom;
        ++irow;
    }while(row);

    // check that there are no points with the same coordinate
    std::vector<cv::Point2f> points = getCorners(false);
    std::vector<cv::Point2f>::const_iterator iter1 = points.begin();
    for(;iter1 != points.end();++iter1)
    {
        // we do not have to check for NaN because of getCorners(false)
        std::vector<cv::Point2f>::const_iterator iter2 = iter1+1;
        for(;iter2 != points.end();++iter2)
            if(*iter1 == *iter2)
                return -1;  // one corner is there twice -> not valid configuration
    }
    return count;
}

bool Chessboard::Board::validateContour()const
{
    std::vector<cv::Point2f> contour = getContour();
    if(contour.size() != 4)
    {
        return false;
    }
    cv::Point2f n1 = contour[1]-contour[0];
    cv::Point2f n2 = contour[2]-contour[1];
    cv::Point2f n3 = contour[3]-contour[2];
    cv::Point2f n4 = contour[0]-contour[3];
    n1 = n1/cv::norm(n1);
    n2 = n2/cv::norm(n2);
    n3 = n3/cv::norm(n3);
    n4 = n4/cv::norm(n4);
    // a > b => cos(a) < cos(b)
    if(fabs(n1.dot(n2)) > MIN_COS_ANGLE||
            fabs(n2.dot(n3)) > MIN_COS_ANGLE||
            fabs(n3.dot(n4)) > MIN_COS_ANGLE||
            fabs(n4.dot(n1)) > MIN_COS_ANGLE)
        return false;
    return true;
}

std::vector<cv::Point2f> Chessboard::Board::getContour()const
{
    std::vector<cv::Point2f> points;
    if(isEmpty())
        return points;

    //find start cell part of the contour
    Cell* start_cell = NULL;
    PointIter iter(top_left,TOP_LEFT);
    do
    {
        PointIter iter2(iter);
        do
        {
            if(!iter2.getCell()->empty())
            {
                start_cell = iter2.getCell();
                iter = iter2;
                break;
            }
        }while(iter2.right());
    }while(!start_cell && iter.bottom());
    if(start_cell == NULL)
        return points;

    // trace contour
    const cv::Point2f *start_pt = *iter;
    int mode = 2; int last = -1;
    do
    {
        PointIter current_iter(iter);
        switch(mode)
        {
        case 1: // top
            if(iter.top(true))
            {
                if(last != 1)
                    points.push_back(**current_iter);
                mode = 4;
                last = 1;
                break;
            }
            /* fallthrough */
        case 2: // right
            if(iter.right(true))
            {
                if(last != 2)
                    points.push_back(**current_iter);
                mode = 1;
                last = 2;
                break;
            }
            /* fallthrough */
        case 3: // bottom
            if(iter.bottom(true))
            {
                if(last != 3)
                    points.push_back(**current_iter);
                mode = 2;
                last = 3;
                break;
            }
            /* fallthrough */
        case 4: // left
            if(iter.left(true))
            {
                if(last != 4)
                    points.push_back(**current_iter);
                mode = 3;
                last = 4;
                break;
            }
            mode = 1;
            break;
        default:
            CV_Error(Error::StsInternal,"cannot retrieve contour");
        }
    }while(*iter != start_pt);
    return points;
}

void Chessboard::Board::maskImage(cv::InputOutputArray img,const cv::Scalar &color)const
{
    Chessboard::Board temp(*this);
    temp.growLeft();
    temp.growRight();
    temp.growTop();
    temp.growBottom();
    cv::Mat contour;
    cv::Mat(temp.getContour()).convertTo(contour,CV_32S);
    std::vector<cv::Mat> contours;
    contours.push_back(contour);
    cv::drawContours(img,contours,0,color,-1);
}

cv::Mat Chessboard::Board::estimateHomography(cv::Rect rect,int field_size)const
{
    int _rows = int(rowCount());
    int _cols = int(colCount());
    if(_rows < 3  || _cols < 3)
        return cv::Mat();
    if(rect.width <= 0)
        rect.width= _cols;
    if(rect.height <= 0)
        rect.height= _rows;

    int col_end = std::min(rect.x+rect.width,_cols);
    int row_end = std::min(rect.y+rect.height,_rows);
    std::vector<cv::Point2f> points = getCorners(true);

    // build src and dst
    std::vector<cv::Point2f> src,dst;
    for(int row =rect.y;row < row_end;++row)
    {
        for(int col=rect.x;col <col_end;++col)
        {
            const cv::Point2f &pt = points[row*_rows+col];
            if(pt.x != pt.x)    // NaN check
                continue;
            src.push_back(cv::Point2f(float(field_size)*(col+1),float(field_size)*(row+1)));
            dst.push_back(pt);
        }
    }
    if(dst.size() < 4)
        return cv::Mat();
    return cv::findHomography(src, dst,cv::LMEDS);
}

cv::Mat Chessboard::Board::estimateHomography(int field_size)const
{
    int _rows = int(rowCount());
    int _cols = int(colCount());
    if(_rows < 3  || _cols < 3)
        return cv::Mat();
    std::vector<cv::Point2f> src,dst;
    std::vector<cv::Point2f> points = getCorners(true);
    std::vector<cv::Point2f>::const_iterator iter = points.begin();
    for(int row =0;row < _rows;++row)
    {
        for(int col=0;col <_cols;++col,++iter)
        {
            const cv::Point2f &pt = *iter;
            if(pt.x == pt.x)
            {
                src.push_back(cv::Point2f(float(field_size)*(col+1),float(field_size)*(row+1)));
                dst.push_back(pt);
            }
        }
    }
    if(dst.size() < 4)
        return cv::Mat();
    return cv::findHomography(src, dst);
}

bool Chessboard::Board::findNextPoint(cv::flann::Index &index,const cv::Mat &data,
        const cv::Point2f &pt1,const cv::Point2f &pt2, const cv::Point2f &pt3,
        float white_angle,float black_angle,float min_response,cv::Point2f &point)
{
    Ellipse ellipse;
    if(!estimateSearchArea(pt1,pt2,pt3,0.4F,ellipse))
        return false;
    if(min_response > fabs(findMaxPoint(index,data,ellipse,white_angle,black_angle,point)))
        return false;
    return true;
}

int Chessboard::Board::grow(const cv::Mat &map,cv::flann::Index &flann_index)
{
    if(isEmpty())
        CV_Error(Error::StsInternal,"Board is empty");
    bool bleft = true;
    bool btop = true;
    bool bright = true;
    bool bbottom= true;
    int count = 0;
    do
    {
        if(btop)
        {
            btop= growTop(map,flann_index);
            if(btop)
            {
                ++count;
                continue;
            }
        }
        if(bbottom)
        {
            bbottom= growBottom(map,flann_index);
            if(bbottom)
            {
                ++count;
                continue;
            }
        }

        // grow to the left
        if(bleft)
        {
            bleft = growLeft(map,flann_index);
            if(bleft)
            {
                ++count;
                continue;
            }
        }
        if(bright)
        {
            bright= growRight(map,flann_index);
            if(bright)
            {
                ++count;
                continue;
            }
        }
    }while(bleft || btop || bright || bbottom );
    return count;
}

std::map<int,int> Chessboard::Board::getMapping()const
{
    std::map<int,int> map;
    std::vector<cv::Point2f>
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **CV_DETECTORS_CHESSBOARD_DEBUG()**: A function/method defined in this file
- **speeding()**: A function/method defined in this file
- **for()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `chessboard.hpp`
- `opencv2/highgui.hpp`
- `math.h`
- `precomp.hpp`
- `opencv2/flann.hpp`

**Python Imports:**
- `dummy`
- `the`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.


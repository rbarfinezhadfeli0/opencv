# Documentation for `modules/objdetect/src/qrcode.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/src/qrcode.cpp`
- **File Name**: `qrcode.cpp`
- **File Size**: 179,030 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/src/qrcode.cpp](../../../modules/objdetect/src/qrcode.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "precomp.hpp"
#include "opencv2/objdetect.hpp"
#include "opencv2/calib3d.hpp"
#include <opencv2/core/utils/logger.hpp>
#include "graphical_code_detector_impl.hpp"

#ifdef HAVE_QUIRC
#include "quirc.h"
#endif

#include <array>
#include <limits>
#include <cmath>
#include <queue>
#include <map>

namespace cv
{
using std::vector;
using std::pair;

static bool checkQRInputImage(InputArray img, Mat& gray)
{
    CV_Assert(!img.empty());
    CV_CheckDepthEQ(img.depth(), CV_8U, "");

    if (img.cols() <= 20 || img.rows() <= 20)
    {
        return false;  // image data is not enough for providing reliable results
    }
    int incn = img.channels();
    CV_Check(incn, incn == 1 || incn == 3 || incn == 4, "");
    if (incn == 3 || incn == 4)
    {
        cvtColor(img, gray, COLOR_BGR2GRAY);
    }
    else
    {
        gray = img.getMat();
    }
    return true;
}

static void updatePointsResult(OutputArray points_, const vector<Point2f>& points)
{
    if (points_.needed())
    {
        int N = int(points.size() / 4);
        if (N > 0)
        {
            Mat m_p(N, 4, CV_32FC2, (void*)&points[0]);
            int points_type = points_.fixedType() ? points_.type() : CV_32FC2;
            m_p.reshape(2, points_.rows()).convertTo(points_, points_type);  // Mat layout: N x 4 x 2cn
        }
        else
        {
            points_.release();
        }
    }
}

static Point2f intersectionLines(Point2f a1, Point2f a2, Point2f b1, Point2f b2)
{
    // Try to solve a two lines intersection (a1, a2) and (b1, b2) as a system of equations:
    // a2 + u * (a1 - a2) = b2 + v * (b1 - b2)
    const float divisor = (a1.x - a2.x) * (b1.y - b2.y) - (a1.y - a2.y) * (b1.x - b2.x);
    const float eps = 0.001f;
    if (abs(divisor) < eps)
        return a2;
    const float u = ((b2.x - a2.x) * (b1.y - b2.y) + (b1.x - b2.x) * (a2.y - b2.y)) / divisor;
    return a2 + u * (a1 - a2);
}

//      / | b
//     /  |
//    /   |
//  a/    | c

static inline double getCosVectors(Point2f a, Point2f b, Point2f c)
{
    return ((a - b).x * (c - b).x + (a - b).y * (c - b).y) / (norm(a - b) * norm(c - b));
}

static bool arePointsNearest(Point2f a, Point2f b, float delta = 0.0)
{
    if ((abs(a.x - b.x) < delta) && (abs(a.y - b.y) < delta))
        return true;
    else
        return false;
}

class QRDetect
{
public:
    void init(const Mat& src, double eps_vertical_ = 0.2, double eps_horizontal_ = 0.1);
    bool localization();
    bool computeTransformationPoints();
    Mat getBinBarcode() { return bin_barcode; }
    Mat getStraightBarcode() { return straight_barcode; }
    vector<Point2f> getTransformationPoints() { return transformation_points; }
protected:
    vector<Vec3d> searchHorizontalLines();
    vector<Point2f> separateVerticalLines(const vector<Vec3d> &list_lines);
    vector<Point2f> extractVerticalLines(const vector<Vec3d> &list_lines, double eps);
    void fixationPoints(vector<Point2f> &local_point);
    vector<Point2f> getQuadrilateral(vector<Point2f> angle_list);
    bool testByPassRoute(vector<Point2f> hull, int start, int finish);

    Mat barcode, bin_barcode, resized_barcode, resized_bin_barcode, straight_barcode;
    vector<Point2f> localization_points, transformation_points;
    double eps_vertical, eps_horizontal, coeff_expansion;
    enum resize_direction { ZOOMING, SHRINKING, UNCHANGED } purpose;
};


void QRDetect::init(const Mat& src, double eps_vertical_, double eps_horizontal_)
{
    CV_TRACE_FUNCTION();
    CV_Assert(!src.empty());
    barcode = src.clone();
    const double min_side = std::min(src.size().width, src.size().height);
    if (min_side < 512.0)
    {
        purpose = ZOOMING;
        coeff_expansion = 512.0 / min_side;
        const int width  = cvRound(src.size().width  * coeff_expansion);
        const int height = cvRound(src.size().height  * coeff_expansion);
        Size new_size(width, height);
        resize(src, barcode, new_size, 0, 0, INTER_LINEAR_EXACT);
    }
    else if (min_side > 512.0)
    {
        purpose = SHRINKING;
        coeff_expansion = min_side / 512.0;
        const int width  = cvRound(src.size().width  / coeff_expansion);
        const int height = cvRound(src.size().height  / coeff_expansion);
        Size new_size(width, height);
        resize(src, resized_barcode, new_size, 0, 0, INTER_AREA);
    }
    else
    {
        purpose = UNCHANGED;
        coeff_expansion = 1.0;
    }

    eps_vertical   = eps_vertical_;
    eps_horizontal = eps_horizontal_;

    if (!barcode.empty())
        adaptiveThreshold(barcode, bin_barcode, 255, ADAPTIVE_THRESH_GAUSSIAN_C, THRESH_BINARY, 83, 2);
    else
        bin_barcode.release();

    if (!resized_barcode.empty())
        adaptiveThreshold(resized_barcode, resized_bin_barcode, 255, ADAPTIVE_THRESH_GAUSSIAN_C, THRESH_BINARY, 83, 2);
    else
        resized_bin_barcode.release();
}

vector<Vec3d> QRDetect::searchHorizontalLines()
{
    CV_TRACE_FUNCTION();
    vector<Vec3d> result;
    const int height_bin_barcode = bin_barcode.rows;
    const int width_bin_barcode  = bin_barcode.cols;
    const size_t test_lines_size = 5;
    double test_lines[test_lines_size];
    vector<size_t> pixels_position;

    for (int y = 0; y < height_bin_barcode; y++)
    {
        pixels_position.clear();
        const uint8_t *bin_barcode_row = bin_barcode.ptr<uint8_t>(y);

        int pos = 0;
        for (; pos < width_bin_barcode; pos++) { if (bin_barcode_row[pos] == 0) break; }
        if (pos == width_bin_barcode) { continue; }

        pixels_position.push_back(pos);
        pixels_position.push_back(pos);
        pixels_position.push_back(pos);

        uint8_t future_pixel = 255;
        for (int x = pos; x < width_bin_barcode; x++)
        {
            if (bin_barcode_row[x] == future_pixel)
            {
                future_pixel = static_cast<uint8_t>(~future_pixel);
                pixels_position.push_back(x);
            }
        }
        pixels_position.push_back(width_bin_barcode - 1);
        for (size_t i = 2; i < pixels_position.size() - 3; i+=2)
        {
            test_lines[0] = static_cast<double>(pixels_position[i - 1] - pixels_position[i - 2]);
            test_lines[1] = static_cast<double>(pixels_position[i    ] - pixels_position[i - 1]);
            test_lines[2] = static_cast<double>(pixels_position[i + 1] - pixels_position[i    ]);
            test_lines[3] = static_cast<double>(pixels_position[i + 2] - pixels_position[i + 1]);
            test_lines[4] = static_cast<double>(pixels_position[i + 3] - pixels_position[i + 2]);

            double length = 0.0, weight = 0.0;  // TODO avoid 'double' calculations

            for (size_t j = 0; j < test_lines_size; j++) { length += test_lines[j]; }

            if (length == 0) { continue; }
            for (size_t j = 0; j < test_lines_size; j++)
            {
                if (j != 2) { weight += fabs((test_lines[j] / length) - 1.0/7.0); }
                else        { weight += fabs((test_lines[j] / length) - 3.0/7.0); }
            }

            if (weight < eps_vertical)
            {
                Vec3d line;
                line[0] = static_cast<double>(pixels_position[i - 2]);
                line[1] = y;
                line[2] = length;
                result.push_back(line);
            }
        }
    }
    return result;
}

vector<Point2f> QRDetect::separateVerticalLines(const vector<Vec3d> &list_lines)
{
    CV_TRACE_FUNCTION();
    const double min_dist_between_points = 10.0;
    const double max_ratio = 1.0;
    for (int coeff_epsilon_i = 1; coeff_epsilon_i < 101; ++coeff_epsilon_i)
    {
        const float coeff_epsilon = coeff_epsilon_i * 0.1f;
        vector<Point2f> point2f_result = extractVerticalLines(list_lines, eps_horizontal * coeff_epsilon);
        if (!point2f_result.empty())
        {
            vector<Point2f> centers;
            Mat labels;
            double compactness = kmeans(
                    point2f_result, 3, labels,
                    TermCriteria(TermCriteria::EPS + TermCriteria::COUNT, 10, 0.1),
                    3, KMEANS_PP_CENTERS, centers);
            double min_dist = std::numeric_limits<double>::max();
            for (size_t i = 0; i < centers.size(); i++)
            {
                double dist = norm(centers[i] - centers[(i+1) % centers.size()]);
                if (dist < min_dist)
                {
                    min_dist = dist;
                }
            }
            if (min_dist < min_dist_between_points)
            {
                continue;
            }
            double mean_compactness = compactness / point2f_result.size();
            double ratio = mean_compactness / min_dist;

            if (ratio < max_ratio)
            {
                return point2f_result;
            }
        }
    }
    return vector<Point2f>();  // nothing
}

vector<Point2f> QRDetect::extractVerticalLines(const vector<Vec3d> &list_lines, double eps)
{
    CV_TRACE_FUNCTION();
    vector<Vec3d> result;
    vector<double> test_lines; test_lines.reserve(6);

    for (size_t pnt = 0; pnt < list_lines.size(); pnt++)
    {
        const int x = cvRound(list_lines[pnt][0] + list_lines[pnt][2] * 0.5);
        const int y = cvRound(list_lines[pnt][1]);

        // --------------- Search vertical up-lines --------------- //

        test_lines.clear();
        uint8_t future_pixel_up = 255;

        int temp_length_up = 0;
        for (int j = y; j < bin_barcode.rows - 1; j++)
        {
            uint8_t next_pixel = bin_barcode.ptr<uint8_t>(j + 1)[x];
            temp_length_up++;
            if (next_pixel == future_pixel_up)
            {
                future_pixel_up = static_cast<uint8_t>(~future_pixel_up);
                test_lines.push_back(temp_length_up);
                temp_length_up = 0;
                if (test_lines.size() == 3)
                    break;
            }
        }

        // --------------- Search vertical down-lines --------------- //

        int temp_length_down = 0;
        uint8_t future_pixel_down = 255;
        for (int j = y; j >= 1; j--)
        {
            uint8_t next_pixel = bin_barcode.ptr<uint8_t>(j - 1)[x];
            temp_length_down++;
            if (next_pixel == future_pixel_down)
            {
                future_pixel_down = static_cast<uint8_t>(~future_pixel_down);
                test_lines.push_back(temp_length_down);
                temp_length_down = 0;
                if (test_lines.size() == 6)
                    break;
            }
        }

        // --------------- Compute vertical lines --------------- //

        if (test_lines.size() == 6)
        {
            double length = 0.0, weight = 0.0;  // TODO avoid 'double' calculations

            for (size_t i = 0; i < test_lines.size(); i++)
                length += test_lines[i];

            CV_Assert(length > 0);
            for (size_t i = 0; i < test_lines.size(); i++)
            {
                if (i % 3 != 0)
                {
                    weight += fabs((test_lines[i] / length) - 1.0/ 7.0);
                }
                else
                {
                    weight += fabs((test_lines[i] / length) - 3.0/14.0);
                }
            }

            if (weight < eps)
            {
                result.push_back(list_lines[pnt]);
            }
        }
    }

    vector<Point2f> point2f_result;
    if (result.size() > 2)
    {
        for (size_t i = 0; i < result.size(); i++)
        {
            point2f_result.push_back(
                  Point2f(static_cast<float>(result[i][0] + result[i][2] * 0.5),
                          static_cast<float>(result[i][1])));
        }
    }
    return point2f_result;
}

void QRDetect::fixationPoints(vector<Point2f> &local_point)
{
    CV_TRACE_FUNCTION();
    double cos_angles[3], norm_triangl[3];

    norm_triangl[0] = norm(local_point[1] - local_point[2]);
    norm_triangl[1] = norm(local_point[0] - local_point[2]);
    norm_triangl[2] = norm(local_point[1] - local_point[0]);

    cos_angles[0] = (norm_triangl[1] * norm_triangl[1] + norm_triangl[2] * norm_triangl[2]
                  -  norm_triangl[0] * norm_triangl[0]) / (2 * norm_triangl[1] * norm_triangl[2]);
    cos_angles[1] = (norm_triangl[0] * norm_triangl[0] + norm_triangl[2] * norm_triangl[2]
                  -  norm_triangl[1] * norm_triangl[1]) / (2 * norm_triangl[0] * norm_triangl[2]);
    cos_angles[2] = (norm_triangl[0] * norm_triangl[0] + norm_triangl[1] * norm_triangl[1]
                  -  norm_triangl[2] * norm_triangl[2]) / (2 * norm_triangl[0] * norm_triangl[1]);

    const double angle_barrier = 0.85;
    if (fabs(cos_angles[0]) > angle_barrier || fabs(cos_angles[1]) > angle_barrier || fabs(cos_angles[2]) > angle_barrier)
    {
        local_point.clear();
        return;
    }

    size_t i_min_cos =
       (cos_angles[0] < cos_angles[1] && cos_angles[0] < cos_angles[2]) ? 0 :
       (cos_angles[1] < cos_angles[0] && cos_angles[1] < cos_angles[2]) ? 1 : 2;

    size_t index_max = 0;
    double max_area = std::numeric_limits<double>::min();
    for (size_t i = 0; i < local_point.size(); i++)
    {
        const size_t current_index = i % 3;
        const size_t left_index  = (i + 1) % 3;
        const size_t right_index = (i + 2) % 3;

        const Point2f current_point(local_point[current_index]),
            left_point(local_point[left_index]), right_point(local_point[right_index]),
            central_point(intersectionLines(current_point,
                              Point2f(static_cast<float>((local_point[left_index].x + local_point[right_index].x) * 0.5),
                                      static_cast<float>((local_point[left_index].y + local_point[right_index].y) * 0.5)),
                              Point2f(0, static_cast<float>(bin_barcode.rows - 1)),
                              Point2f(static_cast<float>(bin_barcode.cols - 1),
                                      static_cast<float>(bin_barcode.rows - 1))));

        vector<Point2f> list_area_pnt;
        list_area_pnt.push_back(current_point);

        vector<LineIterator> list_line_iter;
        list_line_iter.push_back(LineIterator(bin_barcode, current_point, left_point));
        list_line_iter.push_back(LineIterator(bin_barcode, current_point, central_point));
        list_line_iter.push_back(LineIterator(bin_barcode, current_point, right_point));

        for (size_t k = 0; k < list_line_iter.size(); k++)
        {
            LineIterator& li = list_line_iter[k];
            uint8_t future_pixel = 255, count_index = 0;
            for(int j = 0; j < li.count; j++, ++li)
            {
                const Point p = li.pos();
                if (p.x >= bin_barcode.cols ||
                    p.y >= bin_barcode.rows)
                {
                    break;
                }

                const uint8_t value = bin_barcode.at<uint8_t>(p);
                if (value == future_pixel)
                {
                    future_pixel = static_cast<uint8_t>(~future_pixel);
                    count_index++;
                    if (count_index == 3)
                    {
                        list_area_pnt.push_back(p);
                        break;
                    }
                }
            }
        }

        const double temp_check_area = contourArea(list_area_pnt);
        if (temp_check_area > max_area)
        {
            index_max = current_index;
            max_area = temp_check_area;
        }

    }
    if (index_max == i_min_cos) { std::swap(local_point[0], local_point[index_max]); }
    else { local_point.clear(); return; }

    const Point2f rpt = local_point[0], bpt = local_point[1], gpt = local_point[2];
    Matx22f m(rpt.x - bpt.x, rpt.y - bpt.y, gpt.x - rpt.x, gpt.y - rpt.y);
    if( determinant(m) > 0 )
    {
        std::swap(local_point[1], local_point[2]);
    }
}

bool QRDetect::localization()
{
    CV_TRACE_FUNCTION();
    Point2f begin, end;
    vector<Vec3d> list_lines_x = searchHorizontalLines();
    vector<Point2f> list_lines_y;
    Mat labels;
    if (!list_lines_x.empty())
    {
        list_lines_y = separateVerticalLines(list_lines_x);
        if (!list_lines_y.empty())
        {
            kmeans(list_lines_y, 3, labels,
                TermCriteria( TermCriteria::EPS + TermCriteria::COUNT, 10, 0.1),
                3, KMEANS_PP_CENTERS, localization_points);

            fixationPoints(localization_points);
        }
    }

    if (labels.empty())
    {
        localization_points.clear();
    }

    bool square_flag = false, local_points_flag = false;
    double triangle_sides[3];
    double triangle_perim, square_area, img_square_area;
    if (localization_points.size() == 3)
    {
        triangle_sides[0] = norm(localization_points[0] - localization_points[1]);
        triangle_sides[1] = norm(localization_points[1] - localization_points[2]);
        triangle_sides[2] = norm(localization_points[2] - localization_points[0]);

        triangle_perim = (triangle_sides[0] + triangle_sides[1] + triangle_sides[2]) / 2;

        square_area = sqrt((triangle_perim * (triangle_perim - triangle_sides[0])
                                           * (triangle_perim - triangle_sides[1])
                                           * (triangle_perim - triangle_sides[2]))) * 2;
        img_square_area = bin_barcode.cols * bin_barcode.rows;

        if (square_area > (img_square_area * 0.2))
        {
            square_flag = true;
        }
    }
    else
    {
        local_points_flag = true;
    }
    if ((square_flag || local_points_flag) && purpose == SHRINKING)
    {
        localization_points.clear();
        bin_barcode = resized_bin_barcode.clone();
        list_lines_x = searchHorizontalLines();
        if( list_lines_x.empty() ) { return false; }
        list_lines_y = separateVerticalLines(list_lines_x);
        if( list_lines_y.empty() ) { return false; }

        kmeans(list_lines_y, 3, labels,
               TermCriteria( TermCriteria::EPS + TermCriteria::COUNT, 10, 0.1),
               3, KMEANS_PP_CENTERS, localization_points);

        fixationPoints(localization_points);
        if (localization_points.size() != 3) { return false; }

        const int width  = cvRound(bin_barcode.size().width  * coeff_expansion);
        const int height = cvRound(bin_barcode.size().height * coeff_expansion);
        Size new_size(width, height);
        Mat intermediate;
        resize(bin_barcode, intermediate, new_size, 0, 0, INTER_LINEAR_EXACT);
        bin_barcode = intermediate.clone();
        for (size_t i = 0; i < localization_points.size(); i++)
        {
            localization_points[i] *= coeff_expansion;
        }
    }
    if (purpose == ZOOMING)
    {
        const int width  = cvRound(bin_barcode.size().width  / coeff_expansion);
        const int height = cvRound(bin_barcode.size().height / coeff_expansion);
        Size new_size(width, height);
        Mat intermediate;
        resize(bin_barcode, intermediate, new_size, 0, 0, INTER_LINEAR_EXACT);
        bin_barcode = intermediate.clone();
        for (size_t i = 0; i < localization_points.size(); i++)
        {
            localization_points[i] /= coeff_expansion;
        }
    }

    for (size_t i = 0; i < localization_points.size(); i++)
    {
        for (size_t j = i + 1; j < localization_points.size(); j++)
        {
            if (norm(localization_points[i] - localization_points[j]) < 10)
            {
                return false;
            }
        }
    }
    return true;

}

bool QRDetect::computeTransformationPoints()
{
    CV_TRACE_FUNCTION();
    if (localization_points.size() != 3) { return false; }

    vector<Point> locations, non_zero_elem[3], newHull;
    vector<Point2f> new_non_zero_elem[3];
    for (size_t i = 0; i < 3; i++)
    {
        Mat mask = Mat::zeros(bin_barcode.rows + 2, bin_barcode.cols + 2, CV_8UC1);
        uint8_t next_pixel, future_pixel = 255;
        int count_test_lines = 0, index_c = max(0, min(cvRound(localization_points[i].x), bin_barcode.cols - 1));
        const int index_r = max(0, min(cvRound(localization_points[i].y), bin_barcode.rows - 1));
        for (; index_c < bin_barcode.cols - 1; index_c++)
        {
            next_pixel = bin_barcode.ptr<uint8_t>(index_r)[index_c + 1];
            if (next_pixel == future_pixel)
            {
                future_pixel = static_cast<uint8_t>(~future_pixel);
                count_test_lines++;
                if (count_test_lines == 2)
                {
                    floodFill(bin_barcode, mask,
                              Point(index_c + 1, index_r), 255,
                              0, Scalar(), Scalar(), FLOODFILL_MASK_ONLY);
                    break;
                }
            }
        }
        Mat mask_roi = mask(Range(1, bin_barcode.rows - 1), Range(1, bin_barcode.cols - 1));
        findNonZero(mask_roi, non_zero_elem[i]);
        newHull.insert(newHull.end(), non_zero_elem[i].begin(), non_zero_elem[i].end());
    }
    convexHull(newHull, locations);
    for (size_t i = 0; i < locations.size(); i++)
    {
        for (size_t j = 0; j < 3; j++)
        {
            for (size_t k = 0; k < non_zero_elem[j].size(); k++)
            {
                if (locations[i] == non_zero_elem[j][k])
                {
                    new_non_zero_elem[j].push_back(locations[i]);
                }
            }
        }
    }

    double pentagon_diag_norm = -1;
    Point2f down_left_edge_point, up_right_edge_point, up_left_edge_point;
    for (size_t i = 0; i < new_non_zero_elem[1].size(); i++)
    {
        for (size_t j = 0; j < new_non_zero_elem[2].size(); j++)
        {
            double temp_norm = norm(new_non_zero_elem[1][i] - new_non_zero_elem[2][j]);
            if (temp_norm > pentagon_diag_norm)
            {
                down_left_edge_point = new_non_zero_elem[1][i];
                up_right_edge_point  = new_non_zero_elem[2][j];
                pentagon_diag_norm = temp_norm;
            }
        }
    }

    if (down_left_edge_point == Point2f(0, 0) ||
        up_right_edge_point  == Point2f(0, 0) ||
        new_non_zero_elem[0].size() == 0) { return false; }

    double max_area = -1;
    up_left_edge_point = new_non_zero_elem[0][0];

    for (size_t i = 0; i < new_non_zero_elem[0].size(); i++)
    {
        vector<Point2f> list_edge_points;
        list_edge_points.push_back(new_non_zero_elem[0][i]);
        list_edge_points.push_back(down_left_edge_point);
        list_edge_points.push_back(up_right_edge_point);

        double temp_area = fabs(contourArea(list_edge_points));
        if (max_area < temp_area)
        {
            up_left_edge_point = new_non_zero_elem[0][i];
            max_area = temp_area;
        }
    }

    Point2f down_max_delta_point, up_max_delta_point;
    double norm_down_max_delta = -1, norm_up_max_delta = -1;
    for (size_t i = 0; i < new_non_zero_elem[1].size(); i++)
    {
        double temp_norm_delta = norm(up_left_edge_point - new_non_zero_elem[1][i])
                               + norm(down_left_edge_point - new_non_zero_elem[1][i]);
        if (norm_down_max_delta < temp_norm_delta)
        {
            down_max_delta_point = new_non_zero_elem[1][i];
            norm_down_max_delta = temp_norm_delta;
        }
    }


    for (size_t i = 0; i < new_non_zero_elem[2].size(); i++)
    {
        double temp_norm_delta = norm(up_left_edge_point - new_non_zero_elem[2][i])
                               + norm(up_right_edge_point - new_non_zero_elem[2][i]);
        if (norm_up_max_delta < temp_norm_delta)
        {
            up_max_delta_point = new_non_zero_elem[2][i];
            norm_up_max_delta = temp_norm_delta;
        }
    }

    transformation_points.push_back(down_left_edge_point);
    transformation_points.push_back(up_left_edge_point);
    transformation_points.push_back(up_right_edge_point);
    transformation_points.push_back(
        intersectionLines(down_left_edge_point, down_max_delta_point,
                          up_right_edge_point, up_max_delta_point));
    vector<Point2f> quadrilateral = getQuadrilateral(transformation_points);
    transformation_points = quadrilateral;

    int width = bin_barcode.size().width;
    int height = bin_barcode.size().height;
    for (size_t i = 0; i < transformation_points.size(); i++)
    {
        if ((cvRound(transformation_points[i].x) > width) ||
            (cvRound(transformation_points[i].y) > height)) { return false; }
    }
    return true;
}

// test function (if true then ------> else <------ )
bool QRDetect::testByPassRoute(vector<Point2f> hull, int start, int finish)
{
    CV_TRACE_FUNCTION();
    int index_hull = start, next_index_hull, hull_size = (int)hull.size();
    double test_length[2] = { 0.0, 0.0 };
    do
    {
        next_index_hull = index_hull + 1;
        if (next_index_hull == hull_size) { next_index_hull = 0; }
        test_length[0] += norm(hull[index_hull] - hull[next_index_hull]);
        index_hull = next_index_hull;
    }
    while(index_hull != finish);

    index_hull = start;
    do
    {
        next_index_hull = index_hull - 1;
        if (next_index_hull == -1) { next_index_hull = hull_size - 1; }
        test_length[1] += norm(hull[index_hull] - hull[next_index_hull]);
        index_hull = next_index_hull;
    }
    while(index_hull != finish);

    if (test_length[0] < test_length[1]) { return true; } else { return false; }
}

vector<Point2f> QRDetect::getQuadrilateral(vector<Point2f> angle_list)
{
    CV_TRACE_FUNCTION();
    size_t angle_size = angle_list.size();
    uint8_t value, mask_value;
    Mat mask = Mat::zeros(bin_barcode.rows + 2, bin_barcode.cols + 2, CV_8UC1);
    Mat fill_bin_barcode = bin_barcode.clone();
    for (size_t i = 0; i < angle_size; i++)
    {
        LineIterator line_iter(bin_barcode, angle_list[ i      % angle_size],
                                            angle_list[(i + 1) % angle_size]);
        for(int j = 0; j < line_iter.count; j++, ++line_iter)
        {
            Point p = line_iter.pos();
            value = bin_barcode.at<uint8_t>(p);
            mask_value = mask.at<uint8_t>(p + Point(1, 1));
            if (value == 0 && mask_value == 0)
            {
                floodFill(fill_bin_barcode, mask, p, 255,
                          0, Scalar(), Scalar(), FLOODFILL_MASK_ONLY);
            }
        }
    }
    vector<Point> locations;
    Mat mask_roi = mask(Range(1, bin_barcode.rows - 1), Range(1, bin_barcode.cols - 1));

    findNonZero(mask_roi, locations);

    for (size_t i = 0; i < angle_list.size(); i++)
    {
        int x = cvRound(angle_list[i].x);
        int y = cvRound(angle_list[i].y);
        locations.push_back(Point(x, y));
    }

    vector<Point> integer_hull;
    convexHull(locations, integer_hull);
    int hull_size = (int)integer_hull.size();
    vector<Point2f> hull(hull_size);
    for (int i = 0; i < hull_size; i++)
    {
        float x = saturate_cast<float>(integer_hull[i].x);
        float y = saturate_cast<float>(integer_hull[i].y);
        hull[i] = Point2f(x, y);
    }

    const double experimental_area = fabs(contourArea(hull));

    vector<Point2f> result_hull_point(angle_size);
    vector<bool> used_hull_point(hull_size, false);
    double min_norm;
    for (size_t i = 0; i < angle_size; i++)
    {
        min_norm = std::numeric_limits<double>::max();
        int closest_pnt_idx = -1;
        for (int j = 0; j < hull_size; j++)
        {
            if (used_hull_point[j])
            {
                continue;
            }
            double temp_norm = norm(hull[j] - angle_list[i]);
            if (min_norm > temp_norm)
            {
                min_norm = temp_norm;
                closest_pnt_idx = j;
            }
        }
        result_hull_point[i] = hull[closest_pnt_idx];
        used_hull_point[closest_pnt_idx] = true;
    }

    int start_line[2] = { 0, 0 }, finish_line[2] = { 0, 0 }, unstable_pnt = 0;
    for (int i = 0; i < hull_size; i++)
    {
        if (result_hull_point[2] == hull[i]) { start_line[0] = i; }
        if (result_hull_point[1] == hull[i]) { finish_line[0] = start_line[1] = i; }
        if (result_hull_point[0] == hull[i]) { finish_line[1] = i; }
        if (result_hull_point[3] == hull[i]) { unstable_pnt = i; }
    }

    int index_hull, extra_index_hull, next_index_hull, extra_next_index_hull;
    Point result_side_begin[4], result_side_end[4];

    bool bypass_orientation = testByPassRoute(hull, start_line[0], finish_line[0]);

    min_norm = std::numeric_limits<double>::max();
    index_hull = start_line[0];
    do
    {
        if (bypass_orientation) { next_index_hull = index_hull + 1; }
        else { next_index_hull = index_hull - 1; }

        if (next_index_hull == hull_size) { next_index_hull = 0; }
        if (next_index_hull == -1) { next_index_hull = hull_size - 1; }

        Point angle_closest_pnt =  norm(hull[index_hull] - angle_list[1]) >
        norm(hull[index_hull] - angle_list[2]) ? angle_list[2] : angle_list[1];

        Point intrsc_line_hull =
        intersectionLines(hull[index_hull], hull[next_index_hull],
                          angle_list[1], angle_list[2]);
        double temp_norm = getCosVectors(hull[index_hull], intrsc_line_hull, angle_closest_pnt);
        if (min_norm > temp_norm &&
            norm(hull[index_hull] - hull[next_index_hull]) >
            norm(angle_list[1] - angle_list[2]) * 0.1)
        {
            min_norm = temp_norm;
            result_side_begin[0] = hull[index_hull];
            result_side_end[0]   = hull[next_index_hull];
        }


        index_hull = next_index_hull;
    }
    while(index_hull != finish_line[0]);

    if (min_norm == std::numeric_limits<double>::max())
    {
        result_side_begin[0] = angle_list[1];
        result_side_end[0]   = angle_list[2];
    }

    min_norm = std::numeric_limits<double>::max();
    index_hull = start_line[1];
    bypass_orientation = testByPassRoute(hull, start_line[1], finish_line[1]);
    do
    {
        if (bypass_orientation) { next_index_hull = index_hull + 1; }
        else { next_index_hull = index_hull - 1; }

        if (next_index_hull == hull_size) { next_index_hull = 0; }
        if (next_index_hull == -1) { next_index_hull = hull_size - 1; }

        Point angle_closest_pnt =  norm(hull[index_hull] - angle_list[0]) >
        norm(hull[index_hull] - angle_list[1]) ? angle_list[1] : angle_list[0];

        Point intrsc_line_hull =
        intersectionLines(hull[index_hull], hull[next_index_hull],
                          angle_list[0], angle_list[1]);
        double temp_norm = getCosVectors(hull[index_hull], intrsc_line_hull, angle_closest_pnt);
        if (min_norm > temp_norm &&
            norm(hull[index_hull] - hull[next_index_hull]) >
            norm(angle_list[0] - angle_list[1]) * 0.05)
        {
            min_norm = temp_norm;
            result_side_begin[1] = hull[index_hull];
            result_side_end[1]   = hull[next_index_hull];
        }

        index_hull = next_index_hull;
    }
    while(index_hull != finish_line[1]);

    if (min_norm == std::numeric_limits<double>::max())
    {
        result_side_begin[1] = angle_list[0];
        result_side_end[1]   = angle_list[1];
    }

    bypass_orientation = testByPassRoute(hull, start_line[0], unstable_pnt);
    const bool extra_bypass_orientation = testByPassRoute(hull, finish_line[1], unstable_pnt);

    vector<Point2f> result_angle_list(4), test_result_angle_list(4);
    double min_diff_area = std::numeric_limits<double>::max();
    index_hull = start_line[0];
    const double standart_norm = std::max(
        norm(result_side_begin[0] - result_side_end[0]),
        norm(result_side_begin[1] - result_side_end[1]));
    do
    {
        if (bypass_orientation) { next_index_hull = index_hull + 1; }
        else { next_index_hull = index_hull - 1; }

        if (next_index_hull == hull_size) { next_index_hull = 0; }
        if (next_index_hull == -1) { next_index_hull = hull_size - 1; }

        if (norm(hull[index_hull] - hull[next_index_hull]) < standart_norm * 0.1)
        { index_hull = next_index_hull; continue; }

        extra_index_hull = finish_line[1];
        do
        {
            if (extra_bypass_orientation) { extra_next_index_hull = extra_index_hull + 1; }
            else { extra_next_index_hull = extra_index_hull - 1; }

            if (extra_next_index_hull == hull_size) { extra_next_index_hull = 0; }
            if (extra_next_index_hull == -1) { extra_next_index_hull = hull_size - 1; }

            if (norm(hull[extra_index_hull] - hull[extra_next_index_hull]) < standart_norm * 0.1)
            { extra_index_hull = extra_next_index_hull; continue; }

            test_result_angle_list[0]
            = intersectionLines(result_side_begin[0], result_side_end[0],
                                result_side_begin[1], result_side_end[1]);
            test_result_angle_list[1]
            = intersectionLines(result_side_begin[1], result_side_end[1],
                                hull[extra_index_hull], hull[extra_next_index_hull]);
            test_result_angle_list[2]
            = intersectionLines(hull[extra_index_hull], hull[extra_next_index_hull],
                                hull[index_hull], hull[next_index_hull]);
            test_result_angle_list[3]
            = intersectionLines(hull[index_hull], hull[next_index_hull],
                                result_side_begin[0], result_side_end[0]);

            const double test_diff_area
                = fabs(fabs(contourArea(test_result_angle_list)) - experimental_area);
            if (min_diff_area > test_diff_area)
            {
                min_diff_area = test_diff_area;
                for (size_t i = 0; i < test_result_angle_list.size(); i++)
                {
                    result_angle_list[i] = test_result_angle_list[i];
                }
            }

            extra_index_hull = extra_next_index_hull;
        }
        while(extra_index_hull != unstable_pnt);

        index_hull = next_index_hull;
    }
    while(index_hull != unstable_pnt);

    // check label points
    if (norm(result_angle_list[0] - angle_list[1]) > 2) { result_angle_list[0] = angle_list[1]; }
    if (norm(result_angle_list[1] - angle_list[0]) > 2) { result_angle_list[1] = angle_list[0]; }
    if (norm(result_angle_list[3] - angle_list[2]) > 2) { result_angle_list[3] = angle_list[2]; }

    // check calculation point
    if (norm(result_angle_list[2] - angle_list[3]) >
       (norm(result_angle_list[0] - result_angle_list[1]) +
        norm(result_angle_list[0] - result_angle_list[3])) * 0.5 )
    { result_angle_list[2] = angle_list[3]; }

    return result_angle_list;
}

struct ImplContour : public GraphicalCodeDetector::Impl
{
public:
    ImplContour(): epsX(0.2), epsY(0.1)  {}

    double epsX, epsY;
    mutable vector<vector<Point2f>> alignmentMarkers;
    mutable vector<Point2f> updateQrCorners;
    mutable vector<QRCodeEncoder::ECIEncodings> encodings;
    bool useAlignmentMarkers = true;

    bool detect(InputArray in, OutputArray points) const override;
    std::string decode(InputArray img, InputArray points, OutputArray straight_qrcode) const override;
    std::string detectAndDecode(InputArray img, OutputArray points, OutputArray straight_qrcode) const override;

    bool detectMulti(InputArray img, OutputArray points) const override;
    bool decodeMulti(InputArray img, InputArray points, std::vector<cv::String>& decoded_info,
                     OutputArrayOfArrays straight_qrcode) const override;
    bool detectAndDecodeMulti(InputArray img, std::vector<cv::String>& decoded_info, OutputArray points,
                              OutputArrayOfArrays straight_qrcode) const override;

    String decodeCurved(InputArray in, InputArray points, OutputArray straight_qrcode);

    std::string detectAndDecodeCurved(InputArray in, OutputArray points, OutputArray straight_qrcode);

    QRCodeEncoder::ECIEncodings getEncoding(int codeIdx);
};

QRCodeDetector::QRCodeDetector() {
    p = makePtr<ImplContour>();
}

QRCodeDetector& QRCodeDetector::setEpsX(double epsX) {
    std::dynamic_pointer_cast<ImplContour>(p)->epsX = epsX;
    return *this;
}

QRCodeDetector& QRCodeDetector::setEpsY(double epsY) {
    std::dynamic_pointer_cast<ImplContour>(p)->epsY = epsY;
    return *this;
}

QRCodeEncoder::ECIEncodings QRCodeDetector::getEncoding(int codeIdx) {
    auto& encodings = std::dynamic_pointer_cast<ImplContour>(p)->encodings;
    CV_Assert(codeIdx >= 0);
    CV_Assert(codeIdx < static_cast<int>(encodings.size()));
    return encodings[codeIdx];
}

bool ImplContour::detect(InputArray in, OutputArray points) const
{
    Mat inarr;
    if (!checkQRInputImage(in, inarr))
        return false;

    QRDetect qrdet;
    qrdet.init(inarr, epsX, epsY);
    if (!qrdet.localization()) { return false; }
    if (!qrdet.computeTransformationPoints()) { return false; }
    vector<Point2f> pnts2f = qrdet.getTransformationPoints();
    updatePointsResult(points, pnts2f);
    return true;
}

class QRDecode
{
public:
    QRDecode(bool useAlignmentMarkers);
    void init(const Mat &src, const vector<Point2f> &points);
    Mat getIntermediateBarcode() { return intermediate; }
    Mat getStraightBarcode() { return straight; }
    size_t getVersion() { return version; }
    std::string getDecodeInformation() { return result_info; }
    bool straightDecodingProcess();
    bool curvedDecodingProcess();
    vector<Point2f> alignment_coords;
    float coeff_expansion = 1.f;
    vector<Point2f> getOriginalPoints() {return original_points;}
    bool useAlignmentMarkers;

    // Structured Append mode generates a sequence of QR codes.
    // Final message is restored according to the index of the code in sequence.
    // Different QR codes are grouped by a parity value.
    bool isStructured() { return mode == QRCodeEncoder::EncodeMode::MODE_STRUCTURED_APPEND; }
    struct {
        uint8_t parity = 0;
        uint8_t sequence_num = 0;
        uint8_t total_num = 1;
    } structure_info;

    QRCodeEncoder::ECIEncodings eci;

protected:
    double getNumModules();
    Mat getHomography() {
        CV_TRACE_FUNCTION();
        vector<Point2f> perspective_points = {{0.f, 0.f}, {test_perspective_size, 0.f},
                                              {test_perspective_size, test_perspective_size},
                                              {0.f, test_perspective_size}};
        vector<Point2f> pts = original_points;
        return findHomography(pts, perspective_points);
    }
    bool updatePerspective(const Mat& H);
    bool versionDefinition();
    void detectAlignment();
    bool samplingForVersion();
    bool decodingProcess();
    inline double pointPosition(Point2f a, Point2f b , Point2f c);
    float distancePointToLine(Point2f a, Point2f b , Point2f c);
    void getPointsInsideQRCode(const vector<Point2f> &angle_list);
    bool computeClosestPoints(const vector<Point> &result_integer_hull);
    bool computeSidesPoints(const vector<Point> &result_integer_hull);
    vector<Point> getPointsNearUnstablePoint(const vector<Point> &side, int start, int end, int step);
    bool findAndAddStablePoint();
    bool findIndexesCurvedSides();
    bool findIncompleteIndexesCurvedSides();
    Mat getPatternsMask();
    Point findClosestZeroPoint(Point2f original_point);
    bool findPatternsContours(vector<vector<Point> > &patterns_contours);
    bool findPatternsVerticesPoints(vector<vector<Point> > &patterns_vertices_points);
    bool findTempPatternsAddingPoints(vector<std::pair<int, vector<Point> > > &temp_patterns_add_points);
    bool computePatternsAddingPoints(std::map<int, vector<Point> > &patterns_add_points);
    bool addPointsToSides();
    void completeAndSortSides();
    vector<vector<float> > computeSpline(const vector<int> &x_arr, const vector<int> &y_arr);
    bool createSpline(vector<vector<Point2f> > &spline_lines);
    bool divideIntoEvenSegments(vector<vector<Point2f> > &segments_points);
    bool straightenQRCodeInParts();
    bool preparingCurvedQRCodes();

    const static int NUM_SIDES = 2;
    Mat original, bin_barcode, no_border_intermediate, intermediate, straight, curved_to_straight, test_image;
    vector<Point2f> original_points;
    Mat homography;
    vector<Point2f> original_curved_points;
    vector<Point> qrcode_locations;
    vector<std::pair<size_t, Point> > closest_points;
    vector<vector<Point> > sides_points;
    std::pair<size_t, Point> unstable_pair;
    vector<int> curved_indexes, curved_incomplete_indexes;
    std::map<int, vector<Point> > complete_curved_sides;
    std::string result_info;
    uint8_t version, version_size;
    float test_perspective_size;
    QRCodeEncoder::EncodeMode mode;

    struct sortPairAsc
    {
        bool operator()(const std::pair<size_t, double> &a,
                        const std::pair<size_t, double> &b) const
        {
            return a.second < b.second;
        }
    };
    struct sortPairDesc
    {
        bool operator()(const std::pair<size_t, double> &a,
                        const std::pair<size_t, double> &b) const
        {
            return a.second > b.second;
        }
    };
    struct sortPointsByX
    {
        bool operator()(const Point &a, const Point &b) const
        {
            return a.x < b.x;
        }
    };
    struct sortPointsByY
    {
        bool operator()(const Point &a, const Point &b) const
        {
            return a.y < b.y;
        }
    };
};

float static getMinSideLen(const vector<Point2f> &points) {
    CV_Assert(points.size() == 4ull);
    double res = norm(points[1]-points[0]);
    for (size_t i = 1ull; i < points.size(); i++) {
        res = min(res, norm(points[i]-points[(i+1ull) % points.size()]));
    }
    return static_cast<float>(res);
}


void QRDecode::init(const Mat &src, const vector<Point2f> &points)
{
    CV_TRACE_FUNCTION();
    vector<Point2f> bbox = points;
    original = src.clone();
    test_image = src.clone();
    adaptiveThreshold(original, bin_barcode, 255, ADAPTIVE_THRESH_GAUSSIAN_C, THRESH_BINARY, 83, 2);
    intermediate = Mat::zeros(original.size(), CV_8UC1);
    original_points = bbox;
    version = 0;
    version_size = 0;
    test_perspective_size = max(getMinSideLen(points)+1.f, 251.f);
    result_info = "";
}

inline double QRDecode::pointPosition(Point2f a, Point2f b , Point2f c)
{
    return (a.x - b.x) * (c.y - b.y) - (c.x - b.x) * (a.y - b.y);
}

float QRDecode::distancePointToLine(Point2f a, Point2f b , Point2f c)
{
    float A, B, C, result;
    A = c.y - b.y;
    B = c.x - b.x;
    C = c.x * b.y - b.x * c.y;
    float dist = sqrt(A*A + B*B);
    if (dist == 0) return 0;
    result = abs((A * a.x - B * a.y + C)) / dist;

    return result;
}

void QRDecode::getPointsInsideQRCode(const vector<Point2f> &angle_list)
{
    CV_TRACE_FUNCTION();
    size_t angle_size = angle_list.size();
    Mat contour_mask = Mat::zeros(bin_barcode.size(), CV_8UC1);
    for (size_t i = 0; i < angle_size; i++)
    {
        LineIterator line_iter(bin_barcode, angle_list[ i      % angle_size],
                                            angle_list[(i + 1) % angle_size]);
        for(int j = 0; j < line_iter.count; j++, ++line_iter)
        {
            Point p = line_iter.pos();
            contour_mask.at<uint8_t>(p + Point(1, 1)) = 255;
        }
    }
    Point2f center_point = intersectionLines(angle_list[0], angle_list[2],
                                             angle_list[1], angle_list[3]);
    floodFill(contour_mask, center_point, 255, 0, Scalar(), Scalar(), FLOODFILL_FIXED_RANGE);

    vector<Point> locations;
    findNonZero(contour_mask, locations);

    Mat fill_bin_barcode = bin_barcode.clone();
    Mat qrcode_mask = Mat::zeros(bin_barcode.rows + 2, bin_barcode.cols + 2, CV_8UC1);
    uint8_t value, mask_value;
    for(size_t i = 0; i < locations.size(); i++)
    {
        value = bin_barcode.at<uint8_t>(locations[i]);
        mask_value = qrcode_mask.at<uint8_t>(locations[i] + Point(1, 1));
        if (value == 0 && mask_value == 0)
        {
            floodFill(fill_bin_barcode, qrcode_mask, locations[i], 255,
                      0, Scalar(), Scalar(), FLOODFILL_MASK_ONLY);
        }
    }
    Mat qrcode_mask_roi = qrcode_mask(Range(1, qrcode_mask.rows - 1), Range(1, qrcode_mask.cols - 1));
    findNonZero(qrcode_mask_roi, qrcode_locations);
}

bool QRDecode::computeClosestPoints(const vector<Point> &result_integer_hull)
{
    CV_TRACE_FUNCTION();
    double min_norm, max_norm = 0.0;
    size_t idx_min = (size_t)-1;
    for (size_t i = 0; i < original_points.size(); i++)
    {
        min_norm = std::numeric_limits<double>::max();

        Point closest_pnt;
        for (size_t j = 0; j < result_integer_hull.size(); j++)
        {
            Point integer_original_point = original_points[i];
            double temp_norm = norm(integer_original_point - result_integer_hull[j]);
            if (temp_norm < min_norm)
            {
                min_norm = temp_norm;
                closest_pnt = result_integer_hull[j];
                idx_min = j;
            }
        }
        if (min_norm > max_norm)
        {
            max_norm = min_norm;
            unstable_pair = std::pair<size_t,Point>(i, closest_pnt);
        }
        CV_Assert(idx_min != (size_t)-1);
        closest_points.push_back(std::pair<size_t,Point>(idx_min, closest_pnt));
    }

    if (closest_points.size() != 4)
    {
        return false;
    }

    return true;
}

bool QRDecode::computeSidesPoints(const vector<Point> &result_integer_hull)
{
    size_t num_closest_points = closest_points.size();
    vector<Point> points;

    for(size_t i = 0; i < num_closest_points; i++)
    {
        points.clear();
        size_t start = closest_points[i].first,
               end   = closest_points[(i + 1) % num_closest_points].first;
        if (start < end)
        {
            points.insert(points.end(),
                          result_integer_hull.begin() + start,
                          result_integer_hull.begin() + end + 1);
        }
        else
        {
            points.insert(points.end(),
                          result_integer_hull.begin() + start,
                          result_integer_hull.end());
            points.insert(points.end(),
                          result_integer_hull.begin(),
                          result_integer_hull.begin() + end + 1);
        }
        if (abs(result_integer_hull[start].x - result_integer_hull[end].x) >
            abs(result_integer_hull[start].y - result_integer_hull[end].y))
        {
            if (points.front().x > points.back().x)
            {
                std::reverse(points.begin(), points.end());
            }
        }
        else
        {
            if (points.front().y > points.back().y)
            {
                std::reverse(points.begin(), points.end());
            }
        }
        if (points.empty())
        {
            return false;
        }
        sides_points.push_back(points);
    }

    return true;
}

vector<Point> QRDecode::getPointsNearUnstablePoint(const vector<Point> &side, int start, int end, int step)
{
    vector<Point> points;
    Point p1, p2, p3;

    double max_neighbour_angle = 1.0;
    int index_max_angle = start + step;
    bool enough_points = true;

    if(side.size() < 3)
    {
        points.insert(points.end(), side.begin(), side.end());
        return points;
    }
    const double cos_angle_threshold = -0.97;
    for (int i = start + step; i != end; i+= step)
    {
        p1 = side[i + step];
        if (norm(p1 - side[i])        < 5) { continue; }
        p2 = side[i];
        if (norm(p2 - side[i - step]) < 5) { continue; }
        p3 = side[i - step];

        double neighbour_angle = getCosVectors(p1, p2, p3);
        neighbour_angle = floor(neighbour_angle*1000)/1000;

        if ((neighbour_angle <= max_neighbour_angle) && (neighbour_angle < cos_angle_threshold))
        {
            max_neighbour_angle = neighbour_angle;
            index_max_angle = i;
        }
        else if (i == end - step)
        {
            enough_points = false;
            index_max_angle = i;
        }
    }

    if (enough_points)
    {
        p1 = side[index_max_angle + step];
        p2 = side[index_max_angle];
        p3 = side[index_max_angle - step];

        points.push_back(p1);
        points.push_back(p2);
        points.push_back(p3);
    }
    else
    {
        p1 = side[index_max_angle];
        p2 = side[index_max_angle - step];

        points.push_back(p1);
        points.push_back(p2);
    }

    return points;
}

bool QRDecode::findAndAddStablePoint()
{
    size_t idx_unstable_point = unstable_pair.first;
    Point unstable_point = unstable_pair.second;

    vector<Point> current_side_points, next_side_points;
    Point a1, a2, b1, b2;
    int start_current, end_current, step_current, start_next, end_next, step_next;
    vector<Point>::iterator it_a, it_b;

    vector<Point> &current_side = sides_points[(idx_unstable_point + 3) % 4];
    vector<Point> &next_side    = sides_points[idx_unstable_point];

    if(current_side.size() < 2 || next_side.size() < 2)
    {
        return false;
    }

    if(arePointsNearest(unstable_point, current_side.front(), 3.0))
    {
        start_current = (int)current_side.size() - 1;
        end_current = 0;
        step_current = -1;
        it_a = current_side.begin();
    }
    else if(arePointsNearest(unstable_point, current_side.back(), 3.0))
    {
        start_current = 0;
        end_current = (int)current_side.size() - 1;
        step_current = 1;
        it_a = current_side.end() - 1;
    }
    else
    {
        return false;
    }
    if(arePointsNearest(unstable_point, next_side.front(), 3.0))
    {
        start_next = (int)next_side.size() - 1;
        end_next = 0;
        step_next = -1;
        it_b = next_side.begin();
    }
    else if(arePointsNearest(unstable_point, next_side.back(), 3.0))
    {
        start_next = 0;
        end_next = (int)next_side.size() - 1;
        step_next = 1;
        it_b = next_side.end() - 1;
    }
    else
    {
        return false;
    }
    current_side_points = getPointsNearUnstablePoint(current_side, start_current, end_current, step_current);
    next_side_points    = getPointsNearUnstablePoint(next_side, start_next, end_next, step_next);

    if (current_side_points.size() < 2 || next_side_points.size() < 2)
    {
        return false;
    }

    a1 = current_side_points[0];
    a2 = current_side_points[1];

    b1 = next_side_points[0];
    b2 = next_side_points[1];

    if(norm(a1 - b1) < 10 && next_side_points.size() > 2)
    {
        b1 = next_side_points[1];
        b2 = next_side_points[2];
    }

    Point stable_point = intersectionLines(a1, a2, b1, b2);

    const double max_side = std::max(bin_barcode.size().width, bin_barcode.size().height);
    if ((abs(stable_point.x) > max_side) || (abs(stable_point.y) > max_side))
    {
        return false;
    }

    while (*it_a != a1)
    {
        it_a = current_side.erase(it_a);
        if (it_a == current_side.end())
        {
            it_a -= step_current;
        }
        Point point_to_remove_from_current = *it_a;
        if (point_to_remove_from_current.x > max_side || point_to_remove_from_current.y > max_side)
        {
            break;
        }
    }
    while (*it_b != b1)
    {
        it_b = next_side.erase(it_b);
        if (it_b == next_side.end())
        {
            it_b -= step_next;
        }
        Point point_to_remove_from_next = *it_b;
        if (point_to_remove_from_next.x > max_side || point_to_remove_from_next.y > max_side)
        {
            break;
        }
    }

    bool add_stable_point = true;

    for (size_t i = 0; i < original_points.size(); i++)
    {
        if(arePointsNearest(stable_point, original_points[i], 3.0))
        {
            add_stable_point = false;
            break;
        }
    }

    if(add_stable_point)
    {
        current_side.insert(it_a, stable_point);
        next_side.insert(it_b, stable_point);
        closest_points[unstable_pair.first].second = stable_point;
    }
    else
    {
        stable_point = original_points[unstable_pair.first];
        closest_points[unstable_pair.first].second = stable_point;
        current_side.insert(it_a, stable_point);
        next_side.insert(it_b, stable_point);
    }

    return true;
}

bool QRDecode::findIndexesCurvedSides()
{
    double max_dist_to_arc_side = 0.0;
    size_t num_closest_points = closest_points.size();
    int idx_curved_current = -1, idx_curved_opposite = -1;

    for (size_t i = 0; i < num_closest_points; i++)
    {
        double dist_to_arc = 0.0;

        Point arc_start = closest_points[i].second;
        Point arc_end   = closest_points[(i + 1) % num_closest_points].second;

        for (size_t j = 0; j < sides_points[i].size(); j++)
        {
            Point arc_point = sides_points[i][j];
            double dist = distancePointToLine(arc_point, arc_start, arc_end);
            dist_to_arc += dist;
        }
        dist_to_arc /= sides_points[i].size();

        if (dist_to_arc > max_dist_to_arc_side)
        {
            max_dist_to_arc_side = dist_to_arc;
            idx_curved_current = (int)i;
            idx_curved_opposite = (int)(i + 2) % num_closest_points;
        }
    }
    if (idx_curved_current == -1 || idx_curved_opposite == -1)
    {
        return false;
    }

    curved_indexes.push_back(idx_curved_current);
    curved_indexes.push_back(idx_curved_opposite);

    return true;
}

bool QRDecode::findIncompleteIndexesCurvedSides()
{
    int num_closest_points = (int)closest_points.size();

    for (int i = 0; i < NUM_SIDES; i++)
    {
        int idx_side = curved_indexes[i];
        int side_size = (int)sides_points[idx_side].size();

        double max_norm = norm(closest_points[idx_side].second -
                               closest_points[(idx_side + 1) % num_closest_points].second);
        double real_max_norm = 0;

        for (int j = 0; j < side_size - 1; j++)
        {
            double temp_norm = norm(sides_points[idx_side][j] -
                                    sides_points[idx_side][j + 1]);
            if (temp_norm > real_max_norm)
            {
                real_max_norm = temp_norm;
            }
        }
        if (real_max_norm > (0.5 * max_norm))
        {
            curved_incomplete_indexes.push_back(curved_indexes[i]);
        }

    }

    if (curved_incomplete_indexes.size() == 0)
    {
        return false;
    }
    return true;
}

Point QRDecode::findClosestZeroPoint(Point2f original_point)
{
    int orig_x = static_cast<int>(original_point.x);
    int orig_y = static_cast<int>(original_point.y);
    uint8_t value;
    Point zero_point;

    const int step = 2;
    for (int i = std::max(orig_x - step, 0); i >= 0 && i <= std::min(orig_x + step, bin_barcode.cols - 1); i++)
    {
        for (int j = std::max(orig_y - step, 0); j >= 0 && j <= std::min(orig_y + step, bin_barcode.rows - 1); j++)
        {
            Point p(i, j);
            value = bin_barcode.at<uint8_t>(p);
            if (value == 0) zero_point = p;
        }
    }

    return zero_point;
}

Mat QRDecode::getPatternsMask()
{
    Mat mask(bin_barcode.rows + 2, bin_barcode.cols + 2, CV_8UC1, Scalar(0));
    Mat patterns_mask(bin_barcode.rows + 2, bin_barcode.cols + 2, CV_8UC1, Scalar(0));
    Mat fill_bin_barcode = bin_barcode.clone();
    for (size_t i = 0; i < original_points.size(); i++)
    {
        if (i == 2) continue;
        Point p = findClosestZeroPoint(original_points[i]);
        floodFill(fill_bin_barcode, mask, p, 255,
                        0, Scalar(), Scalar(), FLOODFILL_MASK_ONLY);
        patterns_mask += mask;
    }
    Mat mask_roi = patterns_mask(Range(1, bin_barcode.rows - 1), Range(1, bin_barcode.cols - 1));

    return mask_roi;
}

bool QRDecode::findPatternsContours(vector<vector<Point> > &patterns_contours)
{
    Mat patterns_mask = getPatternsMask();
    findContours(patterns_mask, patterns_contours, RETR_EXTERNAL, CHAIN_APPROX_NONE, Point(0, 0));
    if (patterns_contours.size() != 3) {  return false; }
    return true;
}

bool QRDecode::findPatternsVerticesPoints(vector<vector<Point> > &patterns_vertices_points)
{
    vector<vector<Point> > patterns_contours;
    if(!findPatternsContours(patterns_contours))
    {
        return false;
    }
    const int num_vertices = 4;
    for(size_t i = 0; i < patterns_contours.size(); i++)
    {
        vector<Point> convexhull_contours, new_convexhull_contours;
        convexHull(patterns_contours[i], convexhull_contours);

        size_t number_pnts_in_hull = convexhull_contours.size();
        vector<std::pair<size_t, double> > cos_angles_in_hull;
        vector<size_t> min_angle_pnts_indexes;

        for(size_t j = 1; j < number_pnts_in_hull + 1; j++)
        {
            double cos_angle = getCosVectors(convexhull_contours[(j - 1) % number_pnts_in_hull],
                                             convexhull_contours[ j      % number_pnts_in_hull],
                                             convexhull_contours[(j + 1) % number_pnts_in_hull]);
            cos_angles_in_hull.push_back(std::pair<size_t, double>(j, cos_angle));
        }

        sort(cos_angles_in_hull.begin(), cos_angles_in_hull.end(), sortPairDesc());

        for (size_t j = 0; j < cos_angles_in_hull.size(); j++)
        {
            bool add_edge = true;
            for(size_t k = 0; k < min_angle_pnts_indexes.size(); k++)
            {
                if(norm(convexhull_contours[cos_angles_in_hull[j].first % number_pnts_in_hull] -
                        convexhull_contours[min_angle_pnts_indexes[k]   % number_pnts_in_hull]) < 3)
                {
                    add_edge = false;
                }
            }
            if (add_edge)
            {
                min_angle_pnts_indexes.push_back(cos_angles_in_hull[j].first % number_pnts_in_hull);
            }
            if ((int)min_angle_pnts_indexes.size() == num_vertices) { break; }
        }
        std::sort(min_angle_pnts_indexes.begin(), min_angle_pnts_indexes.end());

        vector<Point> contour_vertices_points;

        for (size_t k = 0; k < min_angle_pnts_indexes.size(); k++)
        {
            contour_vertices_points.push_back(convexhull_contours[min_angle_pnts_indexes[k]]);
        }
        patterns_vertices_points.push_back(contour_vertices_points);
    }
    if (patterns_vertices_points.size() != 3)
    {
        return false;
    }

    return true;
}

bool QRDecode::findTempPatternsAddingPoints(vector<std::pair<int, vector<Point> > > &temp_patterns_add_points)
{
    vector<vector<Point> >patterns_contours, patterns_vertices_points;
    if(!findPatternsVerticesPoints(patterns_vertices_points))
    {
        return false;
    }
    if(!findPatternsContours(patterns_contours))
    {
        return false;
    }

    for (size_t i = 0; i < curved_incomplete_indexes.size(); i++)
    {
        int idx_curved_side = curved_incomplete_indexes[i];
        Point close_transform_pnt_curr = original_points[idx_curved_side];
        Point close_transform_pnt_next = original_points[(idx_curved_side + 1) % 4];

        vector<size_t> patterns_indexes;

        for (size_t j = 0; j < patterns_vertices_points.size(); j++)
        {
            for (size_t k = 0; k < patterns_vertices_points[j].size(); k++)
            {
                if (norm(close_transform_pnt_curr - patterns_vertices_points[j][k]) < 5)
                {
                    patterns_indexes.push_back(j);
                    break;
                }
                if (norm(close_transform_pnt_next - patterns_vertices_points[j][k]) < 5)
                {
                    patterns_indexes.push_back(j);
                    break;
                }
            }
        }
        for (size_t j = 0; j < patterns_indexes.size(); j++)
        {
            vector<Point> vertices = patterns_vertices_points[patterns_indexes[j]];
            vector<std::pair<int, double> > vertices_dist_pair;
            vector<Point> points;
            for (size_t k = 0; k < vertices.size(); k++)
            {
                double dist_to_side = distancePointToLine(vertices[k], close_transform_pnt_curr,
                                                                       close_transform_pnt_next);
                vertices_dist_pair.push_back(std::pair<int, double>((int)k, dist_to_side));
            }
            if (vertices_dist_pair.size() == 0)
            {
                return false;
            }
            sort(vertices_dist_pair.begin(), vertices_dist_pair.end(), sortPairAsc());
            Point p1, p2;
            int index_p1_in_vertices = 0, index_p2_in_vertices = 0;
            for (int k = 4; k > 0; k--)
            {
                if((vertices_dist_pair[0].first == k % 4) && (vertices_dist_pair[1].first == (k - 1) % 4))
                {
                    index_p1_in_vertices = vertices_dist_pair[0].first;
                    index_p2_in_vertices = vertices_dist_pair[1].first;
                }
                else if((vertices_dist_pair[1].first == k % 4) && (vertices_dist_pair[0].first == (k - 1) % 4))
                {
                    index_p1_in_vertices = vertices_dist_pair[1].first;
                    index_p2_in_vertices = vertices_dist_pair[0].first;
                }
            }
            if (index_p1_in_vertices == index_p2_in_vertices) return false;

            p1 = vertices[index_p1_in_vertices];
            p2 = vertices[index_p2_in_vertices];

            size_t index_p1_in_contour = 0, index_p2_in_contour = 0;
            vector<Point> add_points = patterns_contours[patterns_indexes[j]];

            for(size_t k = 0; k < add_points.size(); k++)
            {
                if (add_points[k] == p1)
                {
                    index_p1_in_contour = k;
                }
                if (add_points[k] == p2)
                {
                    index_p2_in_contour = k;
                }
            }

            if (index_p1_in_contour > index_p2_in_contour)
            {
                for (size_t k = index_p1_in_contour; k < add_points.size(); k++)
                {
                    points.push_back(add_points[k]);
                }
                for (size_t k = 0; k <= index_p2_in_contour; k++)
                {
                    points.push_back(add_points[k]);
                }
            }
            else if (index_p1_in_contour < index_p2_in_contour)
            {
                for (size_t k = index_p1_in_contour; k <= index_p2_in_contour; k++)
                {
                    points.push_back(add_points[k]);
                }
            }
            else
            {
                return false;
            }
            if (abs(p1.x - p2.x) > abs(p1.y - p2.y))
            {
                std::sort(points.begin(), points.end(), sortPointsByX());
            }
            else
            {
                std::sort(points.begin(), points.end(), sortPointsByY());
            }

            temp_patterns_add_points.push_back(std::pair<int, vector<Point> >(idx_curved_side,points));
        }
    }

    return true;
}

bool QRDecode::computePatternsAddingPoints(std::map<int, vector<Point> > &patterns_add_points)
{
    vector<std::pair<int, vector<Point> > > temp_patterns_add_points;
    if(!findTempPatternsAddingPoints(temp_patterns_add_points))
    {
        return false;
    }

    const int num_points_in_pattern = 3;
    for(size_t i = 0; i < temp_patterns_add_points.size(); i++)
    {
        int idx_side = temp_patterns_add_points[i].first;
        int size = (int)temp_patterns_add_points[i].second.size();

        float step = static_cast<float>(size) / num_points_in_pattern;
        vector<Point> temp_points;
        for (int j = 0; j < num_points_in_pattern; j++)
        {
            float val = j * step;
            int idx = cvRound(val) >= size ? size - 1 : cvRound(val);
            temp_points.push_back(temp_patterns_add_points[i].second[idx]);
        }
        temp_points.push_back(temp_patterns_add_points[i].second.back());
        if(patterns_add_points.count(idx_side) == 1)
        {
            patterns_add_points[idx_side].insert(patterns_add_points[idx_side].end(),
                                                temp_points.begin(), temp_points.end());
        }
        patterns_add_points.insert(std::pair<int, vector<Point> >(idx_side, temp_points));

    }
    if (patterns_add_points.size() == 0)
    {
        return false;
    }

    return true;
}

bool QRDecode::addPointsToSides()
{
    if(!computePatternsAddingPoints(complete_curved_sides))
    {
        return false;
    }
    std::map<int, vector<Point> >::iterator it;
    double mean_step = 0.0;
    size_t num_points_at_side = 0;
    for (it = complete_curved_sides.begin(); it != complete_curved_sides.end(); ++it)
    {
        int count = -1;
        const size_t num_points_at_pattern = it->second.size();
        for(size_t j = 0; j < num_points_at_pattern - 1; j++, count++)
        {
            if (count == 3) continue;
            double temp_norm = norm(it->second[j] -
                                    it->second[j + 1]);
            mean_step += temp_norm;
        }
        num_points_at_side += num_points_at_pattern;
    }
    if (num_points_at_side == 0)
    {
        return false;
    }
    mean_step /= num_points_at_side;

    const size_t num_incomplete_sides = curved_incomplete_indexes.size();
    for (size_t i = 0; i < num_incomplete_sides; i++)
    {
        int idx = curved_incomplete_indexes[i];
        vector<int> sides_points_indexes;

        const int num_points_at_side_to_add = (int)sides_points[idx].size();
        for (int j = 0; j < num_points_at_side_to_add; j++)
        {
            bool not_too_close = true;
            const size_t num_points_at_side_exist = complete_curved_sides[idx].size();
            for (size_t k = 0; k < num_points_at_side_exist; k++)
            {
                double temp_norm = norm(sides_points[idx][j] - complete_curved_sides[idx][k]);
                if (temp_norm < mean_step)
                {
                    not_too_close = false;
                    break;
                }
            }
            if (not_too_close)
            {
                sides_points_indexes.push_back(j);
            }
        }

        for (size_t j = 0; j < sides_points_indexes.size(); j++)
        {
            bool not_equal = true;
            for (size_t k = 0; k < complete_curved_sides[idx].size(); k++)
            {
                if (sides_points[idx][sides_points_indexes[j]] ==
                    complete_curved_sides[idx][k])
                {
                    not_equal = false;
                }
            }
            if (not_equal)
            {
                complete_curved_sides[idx].push_back(sides_points[idx][sides_points_indexes[j]]);
            }
        }
    }

    return true;
}

void QRDecode::completeAndSortSides()
{
    if (complete_curved_sides.size() < 2)
    {
        for (int i = 0; i < NUM_SIDES; i++)
        {
            if(complete_curved_sides.count(curved_indexes[i]) == 0)
            {
                int idx_second_cur_side = curved_indexes[i];
                complete_curved_sides.insert(std::pair<int,vector<Point> >(idx_second_cur_side, sides_points[idx_second_cur_side]));
            }
        }
    }
    std::map<int,vector<Point> >::iterator it;
    for (it = complete_curved_sides.begin(); it != complete_curved_sides.end(); ++it)
    {
        Point p1 = it->second.front();
        Point p2 = it->second.back();
        if (abs(p1.x - p2.x) > abs(p1.y - p2.y))
        {
            std::sort(it->second.begin(), it->second.end(), sortPointsByX());
        }
        else
        {
            std::sort(it->second.begin(), it->second.end(), sortPointsByY());
        }
    }
}

vector<vector<float> > QRDecode::computeSpline(const vector<int> &x_arr, const vector<int> &y_arr)
{
    const int n = (int)x_arr.size();
    vector<float> a, b(n - 1), d(n - 1), h(n - 1), alpha(n - 1), c(n), l(n), mu(n), z(n);

    for (int i = 0; i < (int)y_arr.size(); i++)
    {
        a.push_back(static_cast<float>(x_arr[i]));
    }
    for (int i = 0; i < n - 1; i++)
    {
        h[i] = static_cast<float>(y_arr[i + 1] - y_arr[i]) + std::numeric_limits<float>::epsilon();
    }
    for (int i = 1; i < n - 1; i++)
    {
        alpha[i] = 3 / h[i] * (a[i + 1] - a[i]) - 3 / (h[i - 1]) * (a[i] - a[i - 1]);
    }
    l[0] = 1;
    mu[0] = 0;
    z[0] = 0;

    for (int i = 1; i < n - 1; i++)
    {
        l[i] = 2 * (y_arr[i + 1] - y_arr[i - 1]) - h[i - 1] * mu[i - 1];
        mu[i] = h[i] / l[i];
        z[i] = (alpha[i] - h[i - 1] * z[i - 1]) / l[i];
    }
    l[n - 1] = 1;
    z[n - 1] = 0;
    c[n - 1] = 0;

    for(int j = n - 2; j >= 0; j--)
    {
        c[j] = z[j] - mu[j] * c[j + 1];
        b[j] = (a[j + 1] - a[j]) / h[j] - (h[j] * (c[j + 1] + 2 * c[j])) / 3;
        d[j] = (c[j + 1] - c[j]) / (3 * h[j]);
    }

    vector<vector<float> > S(n - 1);
    for (int i = 0; i < n - 1; i++)
    {
        S[i].push_back(a[i]);
        S[i].push_back(b[i]);
        S[i].push_back(c[i]);
        S[i].push_back(d[i]);
    }

    return S;
}

bool QRDecode::createSpline(vector<vector<Point2f> > &spline_lines)
{
    int start, end;
    vector<vector<float> > S;

    for (int idx = 0; idx < NUM_SIDES; idx++)
    {
        int idx_curved_side = curved_indexes[idx];

        vector<Point> spline_points = complete_curved_sides.find(idx_curved_side)->second;
        vector<int> x_arr, y_arr;

        for (size_t j = 0; j < spline_points.size(); j++)
        {
            x_arr.push_back(cvRound(spline_points[j].x));
            y_arr.push_back(cvRound(spline_points[j].y));
        }

        bool horizontal_order = abs(x_arr.front() - x_arr.back()) > abs(y_arr.front() - y_arr.back());
        vector<int>& second_arr = horizontal_order ? x_arr : y_arr;
        vector<int>& first_arr  = horizontal_order ? y_arr : x_arr;

        S = computeSpline(first_arr, second_arr);

        int closest_point_first  = horizontal_order ? closest_points[idx_curved_side].second.x
                                                    : closest_points[idx_curved_side].second.y;
        int closest_point_second = horizontal_order ? closest_points[(idx_curved_side + 1) % 4].second.x
                                                    : closest_points[(idx_curved_side + 1) % 4].second.y;

        start = idx_curved_side;
        end = (idx_curved_side + 1) % 4;
        if(closest_point_first > closest_point_second)
        {
            start = (idx_curved_side + 1) % 4;
            end = idx_curved_side;
        }

        int closest_point_start = horizontal_order ? closest_points[start].second.x : closest_points[start].second.y;
        int closest_point_end   = horizontal_order ? closest_points[end].second.x   : closest_points[end].second.y;

        for (int index = closest_point_start; index <= closest_point_end; index++)
        {
            if (index == second_arr.front())
            {
                spline_lines[idx].push_back(closest_points[start].second);
            }
            for (size_t i = 0; i < second_arr.size() - 1; i++)
            {
                if ((index > second_arr[i]) && (index <= second_arr[i + 1]))
                {
                    float val = S[i][0] + S[i][1] * (index - second_arr[i]) + S[i][2] * (index - second_arr[i]) * (index - second_arr[i])
                                                                            + S[i][3] * (index - second_arr[i]) * (index - second_arr[i]) * (index - second_arr[i]);
                    spline_lines[idx].push_back(horizontal_order ? Point2f(static_cast<float>(index), val) : Point2f(val, static_cast<float>(index)));
                }
            }
        }
    }
    for (int i = 0; i < NUM_SIDES; i++)
    {
        if (spline_lines[i].size() == 0)
        {
            return false;
        }
    }
    return true;
}

bool QRDecode::divideIntoEvenSegments(vector<vector<Point2f> > &segments_points)
{
    vector<vector<Point2f> > spline_lines(NUM_SIDES);
    if (!createSpline(spline_lines))
    {
        return false;
    }
    float mean_num_points_in_line = 0.0;
    for (int i = 0; i < NUM_SIDES; i++)
    {
        mean_num_points_in_line += spline_lines[i].size();
    }
    mean_num_points_in_line /= NUM_SIDES;
    const int min_num_points = 1, max_num_points = cvRound(mean_num_points_in_line / 2.0);
    float linear_threshold = 0.5f;
    for (int num = min_num_points; num < max_num_points; num++)
    {
        for (int i = 0; i < NUM_SIDES; i++)
        {
            segments_points[i].clear();

            int size = (int)spline_lines[i].size();
            float step = static_cast<float>(size) / num;
            for (int j = 0; j < num; j++)
            {
                float val = j * step;
                int idx = cvRound(val) >= size ? size - 1 : cvRound(val);
                segments_points[i].push_back(spline_lines[i][idx]);
            }
            segments_points[i].push_back(spline_lines[i].back());
        }
        float mean_of_two_sides = 0.0;
        for (int i = 0; i < NUM_SIDES; i++)
        {
            float mean_dist_in_segment = 0.0;
            for (size_t j = 0; j < segments_points[i].size() - 1; j++)
            {
                Point2f segment_start = segments_points[i][j];
                Point2f segment_end   = segments_points[i][j + 1];
                vector<Point2f>::iterator it_start, it_end, it;
                it_start = std::find(spline_lines[i].begin(), spline_lines[i].end(), segment_start);
                it_end   = std::find(spline_lines[i].begin(), spline_lines[i].end(), segment_end);
                float max_dist_to_line = 0.0;
                for (it = it_start; it != it_end; it++)
                {
                    float temp_dist = distancePointToLine(*it, segment_start, segment_end);
                    if (temp_dist > max_dist_to_line)
                    {
                        max_dist_to_line = temp_dist;
                    }
                }
                mean_dist_in_segment += max_dist_to_line;
            }
            mean_dist_in_segment /= segments_points[i].size();
            mean_of_two_sides    += mean_dist_in_segment;
        }
        mean_of_two_sides /= NUM_SIDES;
        if (mean_of_two_sides < linear_threshold)
        {
            break;
        }
    }

    return true;
}

bool QRDecode::straightenQRCodeInParts()
{
    vector<vector<Point2f> > segments_points(NUM_SIDES);
    if (!divideIntoEvenSegments(segments_points))
    {
        return false;
    }
    vector<Point2f> current_curved_side, opposite_curved_side;

    for (int i = 0; i < NUM_SIDES; i++)
    {
        Point2f temp_point_start = segments_points[i].front();
        Point2f temp_point_end   = segments_points[i].back();
        bool horizontal_order = (abs(temp_point_start.x - temp_point_end.x) >
                                 abs(temp_point_start.y - temp_point_end.y));
        float compare_point_current  = horizontal_order ? segments_points[i].front().y
                                                        : segments_points[(i + 1) % 2].front().x;
        float compare_point_opposite = horizontal_order ? segments_points[(i + 1) % 2].front().y
                                                        : segments_points[i].front().x;

        if (compare_point_current > compare_point_opposite)
        {
            current_curved_side  = segments_points[i];
            opposite_curved_side = segments_points[(i + 1) % 2];
        }
    }
    if (current_curved_side.size() != opposite_curved_side.size())
    {
        return false;
    }
    size_t number_pnts_to_cut = current_curved_side.size();
    if (number_pnts_to_cut == 0)
    {
        return false;
    }
    float perspective_curved_size = max(getMinSideLen(original_points)+1.f, 251.f);;
    const Size temporary_size(cvRound(perspective_curved_size), cvRound(perspective_curved_size));

    float dist = perspective_curved_size / (number_pnts_to_cut - 1);
    Mat perspective_result = Mat::zeros(temporary_size, CV_8UC1);
    vector<Point2f> curved_parts_points;

    float start_cut = 0.0;
    vector<Point2f> temp_closest_points(4);

    for (size_t i = 1; i < number_pnts_to_cut; i++)
    {
        curved_parts_points.clear();
        Mat test_mask = Mat::zeros(bin_barcode.size(), CV_8UC1);

        Point2f start_point = current_curved_side[i];
        Point2f prev_start_point = current_curved_side[i - 1];
        Point2f finish_point = opposite_curved_side[i];
        Point2f prev_finish_point = opposite_curved_side[i - 1];

        for (size_t j = 0; j < qrcode_locations.size(); j++)
        {
            if ((pointPosition(start_point, finish_point, qrcode_locations[j]) >= 0) &&
                (pointPosition(prev_start_point, prev_finish_point, qrcode_locations[j]) <= 0))
            {
                test_mask.at<uint8_t>(qrcode_locations[j]) = 255;
            }
        }

        vector<Point2f> perspective_points;

        perspective_points.push_back(Point2f(0.0, start_cut));
        perspective_points.push_back(Point2f(perspective_curved_size, start_cut));

        perspective_points.push_back(Point2f(perspective_curved_size, start_cut + dist));
        perspective_points.push_back(Point2f(0.0, start_cut+dist));

        perspective_points.push_back(Point2f(perspective_curved_size * 0.5f, start_cut + dist * 0.5f));

        if (i == 1)
        {
            for (size_t j = 0; j < closest_points.size(); j++)
            {
                if (arePointsNearest(closest_points[j].second, prev_start_point, 3.0))
                {
                    temp_closest_points[j] = perspective_points[0];
                }
                else if (arePointsNearest(closest_points[j].second, prev_finish_point, 3.0))
                {
                    temp_closest_points[j] = perspective_points[1];
                }
            }
        }
        if (i == number_pnts_to_cut - 1)
        {
            for (size_t j = 0; j < closest_points.size(); j++)
            {
                if (arePointsNearest(closest_points[j].second, finish_point, 3.0))
                {
                    temp_closest_points[j] = perspective_points[2];
                }
                else if (arePointsNearest(closest_points[j].second, start_point, 3.0))
                {
                    temp_closest_points[j] = perspective_points[3];
                }
            }
        }
        start_cut += dist;

        curved_parts_points.push_back(prev_start_point);
        curved_parts_points.push_back(prev_finish_point);
        curved_parts_points.push_back(finish_point);
        curved_parts_points.push_back(start_point);

        Point2f center_point = intersectionLines(curved_parts_points[0], curved_parts_points[2],
                                                 curved_parts_points[1], curved_parts_points[3]);
        if (cvIsNaN(center_point.x) || cvIsNaN(center_point.y))
            return false;

        vector<Point2f> pts = curved_parts_points;
        pts.push_back(center_point);

        Mat H = findHomography(pts, perspective_points);
        if (H.empty())
            return false;
        Mat temp_intermediate(temporary_size, CV_8UC1);
        warpPerspective(test_mask, temp_intermediate, H, temporary_size, INTER_NEAREST);
        perspective_result += temp_intermediate;

    }
    Mat white_mask = Mat(temporary_size, CV_8UC1, Scalar(255));
    Mat inversion = white_mask - perspective_result;
    Mat temp_result;

    original_curved_points = temp_closest_points;

    Point2f original_center_point = intersectionLines(original_curved_points[0], original_curved_points[2],
                                                      original_curved_points[1], original_curved_points[3]);

    original_curved_points.push_back(original_center_point);

    for (size_t i = 0; i < original_curved_points.size(); i++)
    {
        if (cvIsNaN(original_curved_points[i].x) || cvIsNaN(original_curved_points[i].y))
            return false;
    }

    vector<Point2f> perspective_straight_points;
    perspective_straight_points.push_back(Point2f(0.f, 0.f));
    perspective_straight_points.push_back(Point2f(perspective_curved_size, 0.f));

    perspective_straight_points.push_back(Point2f(perspective_curved_size, perspective_curved_size));
    perspective_straight_points.push_back(Point2f(0.f, perspective_curved_size));

    perspective_straight_points.push_back(Point2f(perspective_curved_size * 0.5f, perspective_curved_size * 0.5f));

    Mat H = findHomography(original_curved_points, perspective_straight_points);
    if (H.empty())
        return false;
    warpPerspective(inversion, temp_result, H, temporary_size, INTER_NEAREST, BORDER_REPLICATE);

    no_border_intermediate = temp_result(Range(1, temp_result.rows), Range(1, temp_result.cols));
    const int border = cvRound(0.1 * perspective_curved_size);
    const int borderType = BORDER_CONSTANT;
    copyMakeBorder(no_border_intermediate, curved_to_straight, border, border, border, border, borderType, Scalar(255));
    intermediate = curved_to_straight;

    return true;
}

bool QRDecode::preparingCurvedQRCodes()
{
    vector<Point> result_integer_hull;
    getPointsInsideQRCode(original_points);
    if (qrcode_locations.size() == 0)
        return false;
    convexHull(qrcode_locations, result_integer_hull);
    if (!computeClosestPoints(result_integer_hull))
        return false;
    if (!computeSidesPoints(result_integer_hull))
        return false;
    if (!findAndAddStablePoint())
        return false;
    if (!findIndexesCurvedSides())
        return false;
    if (findIncompleteIndexesCurvedSides())
    {
        if(!addPointsToSides())
            return false;
    }
    completeAndSortSides();
    if (!straightenQRCodeInParts())
        return false;

    return true;
}

/**
 * @param finderPattern 4 points of finder pattern markers, calculated by findPatternsVerticesPoints()
 * @return true if the pattern has the correct side lengths
 */
static inline bool checkFinderPatternByAspect(const vector<Point> &finderPattern) {
    if (finderPattern.size() != 4ull)
        return false;
    float sidesLen[4];
    for (size_t i = 0; i < finderPattern.size(); i++) {
        sidesLen[i] = (sqrt(normL2Sqr<float>(Point2f(finderPattern[i] - finderPattern[(i+1ull)%finderPattern.size()]))));
    }
    const float maxSide = max(max(sidesLen[0], sidesLen[1]), max(sidesLen[2], sidesLen[3]));
    const float minSide = min(min(sidesLen[0], sidesLen[1]), min(sidesLen[2], sidesLen[3]));

    const float patternMaxRelativeLen = .3f;
    if (1.f - minSide / maxSide > patternMaxRelativeLen)
        return false;
    return true;
}

/**
 * @param finderPattern - 4 points of finder pattern markers, calculated by findPatternsVerticesPoints()
 * @param cornerPointsQR - 4 corner points of QR code
 * @return pair<int, int> first - the index in points of finderPattern closest to the corner of the QR code,
 *                        second - the index in points of cornerPointsQR closest to the corner of finderPattern
 *
 * This function matches finder patterns to the corners of the QR code. Points of finder pattern calculated by
 * findPatternsVerticesPoints() may be erroneous, so they are checked.
 */
static inline std::pair<int, int> matchPatternPoints(const vector<Point> &finderPattern,
                                                     const vector<Point2f>& cornerPointsQR) {
    if (!checkFinderPatternByAspect(finderPattern))
        return std::make_pair(-1, -1);

    float distanceToOrig = normL2Sqr<float>(Point2f(finderPattern[0]) - cornerPointsQR[0]);
    int closestFinderPatternV = 0;
    int closetOriginalV = 0;

    for (size_t i = 0ull; i < finderPattern.size(); i++) {
        for (size_t j = 0ull; j < cornerPointsQR.size(); j++) {
            const float tmp = normL2Sqr<float>(Point2f(finderPattern[i]) - cornerPointsQR[j]);
            if (tmp < distanceToOrig) {
                distanceToOrig = tmp;
                closestFinderPatternV = (int)i;
                closetOriginalV = (int)j;
            }
        }
    }
    distanceToOrig = sqrt(distanceToOrig);

    // check that the distance from the QR pattern to the corners of the QR code is small
    const float originalQrSide = sqrt(normL2Sqr<float>(cornerPointsQR[0] - cornerPointsQR[1]))*0.5f +
                                 sqrt(normL2Sqr<float>(cornerPointsQR[0] - cornerPointsQR[3]))*0.5f;
    const float maxRelativeDistance = .1f;

    if (distanceToOrig/originalQrSide > maxRelativeDistance)
        return std::make_pair(-1, -1);
    return std::make_pair(closestFinderPatternV, closetOriginalV);
}

double QRDecode::getNumModules() {
    vector<vector<Point>> finderPatterns;
    double numModulesX = 0., numModulesY = 0.;
    if (findPatternsVerticesPoints(finderPatterns)) {
        double pattern_distance[4] = {0.,0.,0.,0.};
        for (auto& pattern : finderPatterns) {
            auto indexes = matchPatternPoints(pattern, original_points);
            if (indexes == std::make_pair(-1, -1))
                return 0.;
            Point2f vf[4] = {pattern[indexes.first % 4], pattern[(1+indexes.first) % 4],
                                  pattern[(2+indexes.first) % 4], pattern[(3+indexes.first) % 4]};
            for (int i = 1; i < 4; i++) {
                pattern_distance[indexes.second] += (norm(vf[i] - vf[i-1]));
            }
            pattern_distance[indexes.second] += norm(vf[3] - vf[0]);
            pattern_distance[indexes.second] /= 4.;
        }
        const double moduleSizeX = (pattern_distance[0] + pattern_distance[1])/(2.*7.);
        const double moduleSizeY = (pattern_distance[0] + pattern_distance[3])/(2.*7.);
        numModulesX = norm(original_points[1] - original_points[0])/moduleSizeX;
        numModulesY = norm(original_points[3] - original_points[0])/moduleSizeY;
    }
    return (numModulesX + numModulesY)/2.;
}

// use code from https://stackoverflow.com/questions/13238704/calculating-the-position-of-qr-code-alignment-patterns
static inline vector<pair<int, int>> getAlignmentCoordinates(int version) {
    if (version <= 1) return {};
    int intervals = (version / 7) + 1; // Number of gaps between alignment patterns
    int distance = 4 * version + 4; // Distance between first and last alignment pattern
    int step = cvRound((double)distance / (double)intervals); // Round equal spacing to nearest integer
    step += step & 0b1; // Round step to next even number
    vector<int> coordinates((size_t)intervals + 1ull);
    coordinates[0] = 6; // First coordinate is always 6 (can't be calculated with step)
    for (int i = 1; i <= intervals; i++) {
        coordinates[i] = (6 + distance - step * (intervals - i));  // Start right/bottom and go left/up by step*k
    }
    if (version >= 7) {
        return {std::make_pair(coordinates.back(), coordinates.back()),
                std::make_pair(coordinates.back(), coordinates[coordinates.size()-2]),
                std::make_pair(coordinates[coordinates.size()-2], coordinates.back()),
                std::make_pair(coordinates[coordinates.size()-2], coordinates[coordinates.size()-2]),
                std::make_pair(coordinates[0], coordinates[1]),
                std::make_pair(coordinates[1], coordinates[0]),
               };
    }
    return {std::make_pair(coordinates.back(), coordinates.back())};
}


bool QRDecode::updatePerspective(const Mat& H)
{
    if (H.empty())
        return false;
    homography = H;
    Mat temp_intermediate;
    const Size temporary_size(cvRound(test_perspective_size), cvRound(test_perspective_size));
    warpPerspective(bin_barcode, temp_intermediate, H, temporary_size, INTER_NEAREST);
    no_border_intermediate = temp_intermediate(Range(1, temp_intermediate.rows), Range(1, temp_intermediate.cols));

    const int border = cvRound(0.1 * test_perspective_size);
    const int borderType = BORDER_CONSTANT;
    copyMakeBorder(no_border_intermediate, intermediate, border, border, border, border, borderType, Scalar(255));
    return true;
}

static inline Point computeOffset(const vector<Point>& v)
{
    // compute the width/height of convex hull
    Rect areaBox = boundingRect(v);

    // compute the good offset
    // the box is consisted by 7 steps
    // to pick the middle of the stripe, it needs to be 1/14 of the size
    const int cStep = 7 * 2;
    Point offset = Point(areaBox.width, areaBox.height);
    offset /= cStep;
    return offset;
}

// QR code with version 7 or higher has a special 18 bit version number code.
// @return std::pair<double, int> first - distance to estimatedVersion, second - version
/**
 * @param numModules - estimated numModules
 * @param estimatedVersion
 * @return pair<double, int>, first - Hamming distance to 18 bit code, second - closest version
 *
 * QR code with version 7 or higher has a special 18 bit version number code:
 * https://www.thonky.com/qr-code-tutorial/format-version-information
 */
static inline std::pair<double, int> getVersionByCode(double numModules, Mat qr, int estimatedVersion) {
    const double moduleSize = qr.rows / numModules;
    Point2d startVersionInfo1 = Point2d((numModules-8.-3.)*moduleSize, 0.);
    Point2d endVersionInfo1 = Point2d((numModules-8.)*moduleSize, moduleSize*6.);
    Point2d startVersionInfo2 = Point2d(0., (numModules-8.-3.)*moduleSize);
    Point2d endVersionInfo2 = Point2d(moduleSize*6., (numModules-8.)*moduleSize);
    Mat v1(qr, Rect2d(startVersionInfo1, endVersionInfo1));
    Mat v2(qr, Rect2d(startVersionInfo2, endVersionInfo2));
    const double thresh = 127.;
    resize(v1, v1, Size(3, 6), 0., 0., INTER_AREA);
    threshold(v1, v1, thresh, 255, THRESH_BINARY);
    resize(v2, v2, Size(6, 3), 0., 0., INTER_AREA);
    threshold(v2, v2, thresh, 255, THRESH_BINARY);

    Mat version1, version2;
    // convert version1 (top right version information block) and
    // version2 (bottom left version information block) to version table format
    // https://www.thonky.com/qr-code-tutorial/format-version-tables
    rotate((255-v1)/255, version1, ROTATE_180), rotate(((255-v2)/255).t(), version2, ROTATE_180);

    static uint8_t versionCodes[][18] = {{0,0,0,1,1,1,1,1,0,0,1,0,0,1,0,1,0,0},{0,0,1,0,0,0,0,1,0,1,1,0,1,1,1,1,0,0},
                                         {0,0,1,0,0,1,1,0,1,0,1,0,0,1,1,0,0,1},{0,0,1,0,1,0,0,1,0,0,1,1,0,1,0,0,1,1},
                                         {0,0,1,0,1,1,1,0,1,1,1,1,1,1,0,1,1,0},{0,0,1,1,0,0,0,1,1,1,0,1,1,0,0,0,1,0},
                                         {0,0,1,1,0,1,1,0,0,0,0,1,0,0,0,1,1,1},{0,0,1,1,1,0,0,1,1,0,0,0,0,0,1,1,0,1},
                                         {0,0,1,1,1,1,1,0,0,1,0,0,1,0,1,0,0,0},{0,1,0,0,0,0,1,0,1,1,0,1,1,1,1,0,0,0},
                                         {0,1,0,0,0,1,0,1,0,0,0,1,0,1,1,1,0,1},{0,1,0,0,1,0,1,0,1,0,0,0,0,1,0,1,1,1},
                                         {0,1,0,0,1,1,0,1,0,1,0,0,1,1,0,0,1,0},{0,1,0,1,0,0,1,0,0,1,1,0,1,0,0,1,1,0},
                                         {0,1,0,1,0,1,0,1,1,0,1,0,0,0,0,0,1,1},{0,1,0,1,1,0,1,0,0,0,1,1,0,0,1,0,0,1},
                                         {0,1,0,1,1,1,0,1,1,1,1,1,1,0,1,1,0,0},{0,1,1,0,0,0,1,1,1,0,1,1,0,0,0,1,0,0},
                                         {0,1,1,0,0,1,0,0,0,1,1,1,1,0,0,0,0,1},{0,1,1,0,1,0,1,1,1,1,1,0,1,0,1,0,1,1},
                                         {0,1,1,0,1,1,0,0,0,0,1,0,0,0,1,1,1,0},{0,1,1,1,0,0,1,1,0,0,0,0,0,1,1,0,1,0},
                                         {0,1,1,1,0,1,0,0,1,1,0,0,1,1,1,1,1,1},{0,1,1,1,1,0,1,1,0,1,0,1,1,1,0,1,0,1},
                                         {0,1,1,1,1,1,0,0,1,0,0,1,0,1,0,0,0,0},{1,0,0,0,0,0,1,0,0,1,1,1,0,1,0,1,0,1},
                                         {1,0,0,0,0,1,0,1,1,0,1,1,1,1,0,0,0,0},{1,0,0,0,1,0,1,0,0,0,1,0,1,1,1,0,1,0},
                                         {1,0,0,0,1,1,0,1,1,1,1,0,0,1,1,1,1,1},{1,0,0,1,0,0,1,0,1,1,0,0,0,0,1,0,1,1},
                                         {1,0,0,1,0,1,0,1,0,0,0,0,1,0,1,1,1,0},{1,0,0,1,1,0,1,0,1,0,0,1,1,0,0,1,0,0},
                                         {1,0,0,1,1,1,0,1,0,1,0,1,0,0,0,0,0,1},{1,0,1,0,0,0,1,1,0,0,0,1,1,0,1,0,0,1}
    };
    double minDist = 19.;
    int bestVersion = -1;
    const double penaltyFactor = 0.8;

    for (int i = 0; i < (int)(sizeof(versionCodes)/sizeof(versionCodes[0])); i++) {
        Mat currVers(Size(3, 6), CV_8UC1, versionCodes[i]);
        // minimum hamming distance between version = 8
        double tmp = norm(currVers, version1, NORM_HAMMING) + penaltyFactor*abs(estimatedVersion-i-7);
        if (tmp < minDist) {
            bestVersion = i+7;
            minDist = tmp;
        }
        tmp = norm(currVers, version2, NORM_HAMMING) + penaltyFactor*abs(estimatedVersion-i-7);
        if (tmp < minDist) {
            bestVersion = i+7;
            minDist = tmp;
        }
    }
    return std::make_pair(minDist, bestVersion);
}

bool QRDecode::versionDefinition()
{
    CV_TRACE_FUNCTION();
    CV_LOG_DEBUG(NULL, "QR corners: " << original_points[0] << " " << original_points[1] << " " << original_points[2] <<
                      " " << original_points[3]);
    LineIterator line_iter(intermediate, Point2f(0, 0), Point2f(test_perspective_size, test_perspective_size));
    Point black_point = Point(0, 0);
    for(int j = 0; j < line_iter.count; j++, ++line_iter)
    {
        const uint8_t value = intermediate.at<uint8_t>(line_iter.pos());
        if (value == 0)
        {
            black_point = line_iter.pos();
            break;
        }
    }

    Mat mask = Mat::zeros(intermediate.rows + 2, intermediate.cols + 2, CV_8UC1);
    floodFill(intermediate, mask, black_point, 255, 0, Scalar(), Scalar(), FLOODFILL_MASK_ONLY);

    vector<Point> locations, non_zero_elem;
    Mat mask_roi = mask(Range(1, intermediate.rows - 1), Range(1, intermediate.cols - 1));
    findNonZero(mask_roi, non_zero_elem);
    convexHull(non_zero_elem, locations);
    Point offset = computeOffset(locations);

    Point temp_remote = locations[0], remote_point;
    const Point delta_diff = offset;
    for (size_t i = 0; i < locations.size(); i++)
    {
        if (norm(black_point - temp_remote) <= norm(black_point - locations[i]))
        {
            const uint8_t value = intermediate.at<uint8_t>(temp_remote - delta_diff);
            temp_remote = locations[i];
            if (value == 0) { remote_point = temp_remote - delta_diff; }
            else { remote_point = temp_remote - (delta_diff / 2); }
        }
    }

    size_t transition_x = 0 , transition_y = 0;

    uint8_t future_pixel = 255;
    const uint8_t *intermediate_row = intermediate.ptr<uint8_t>(remote_point.y);
    for(int i = remote_point.x; i < intermediate.cols; i++)
    {
        if (intermediate_row[i] == future_pixel)
        {
            future_pixel = static_cast<uint8_t>(~future_pixel);
            transition_x++;
        }
    }

    future_pixel = 255;
    for(int j = remote_point.y; j < intermediate.rows; j++)
    {
        const uint8_t value = intermediate.at<uint8_t>(Point(j, remote_point.x));
        if (value == future_pixel)
        {
            future_pixel = static_cast<uint8_t>(~future_pixel);
            transition_y++;
        }
    }

    const int versionByTransition = saturate_cast<uint8_t>((std::min(transition_x, transition_y) - 1) * 0.25 - 1);
    const int numModulesByTransition = 21 + (versionByTransition - 1) * 4;

    const double numModulesByFinderPattern = getNumModules();
    const double versionByFinderPattern = (numModulesByFinderPattern - 21.) * .25 + 1.;
    bool useFinderPattern = false;
    const double thresholdFinderPattern = 0.2;
    const double roundingError = abs(numModulesByFinderPattern - cvRound(numModulesByFinderPattern));

    if (cvRound(versionByFinderPattern) >= 1 && versionByFinderPattern <= 6. &&
        transition_x != transition_y && roundingError < thresholdFinderPattern) {
            useFinderPattern = true;
    }

    bool useCode = false;
    int versionByCode = 7;
    if (cvRound(versionByFinderPattern) >= 7 || versionByTransition >= 7) {
        vector<std::pair<double, int>> versionAndDistances;
        if (cvRound(versionByFinderPattern) >= 7) {
            versionAndDistances.push_back(getVersionByCode(numModulesByFinderPattern, no_border_intermediate,
                                                           cvRound(versionByFinderPattern)));
        }
        if (versionByTransition >= 7) {
            versionAndDistances.push_back(getVersionByCode(numModulesByTransition, no_border_intermediate,
                                                           versionByTransition));
        }
        const auto& bestVersion = min(versionAndDistances.front(), versionAndDistances.back());
        double distanceByCode = bestVersion.first;
        versionByCode = bestVersion.second;
        if (distanceByCode < 5.) {
            useCode = true;
        }
    }

    if (useCode) {
        CV_LOG_DEBUG(NULL, "Version type: useCode");
        version = (uint8_t)versionByCode;
    }
    else if (useFinderPattern ) {
        CV_LOG_DEBUG(NULL, "Version type: useFinderPattern");
        version = (uint8_t)cvRound(versionByFinderPattern);
    }
    else {
        CV_LOG_DEBUG(NULL, "Version type: useTransition");
        version = (uint8_t)versionByTransition;
    }
    version_size = 21 + (version - 1) * 4;
    if ( !(0 < version && version <= 40) ) { return false; }
    CV_LOG_DEBUG(NULL, "QR version: " << (int)version);
    return true;
}

void QRDecode::detectAlignment() {
    vector<pair<int, int>> alignmentPositions = getAlignmentCoordinates(version);
    if (alignmentPositions.size() > 0) {
        vector<Point2f> perspective_points = {{0.f, 0.f}, {test_perspective_size, 0.f}, {0.f, test_perspective_size}};
        vector<Point2f> object_points = {original_points[0], original_points[1], original_points[3]};

        // create alignment image
        static uint8_t alignmentMarker[25] = {
            0, 0,   0,   0,   0,
            0, 255, 255, 255, 0,
            0, 255, 0,   255, 0,
            0, 255, 255, 255, 0,
            0, 0,   0,   0,   0
        };
        Mat alignmentMarkerMat(5, 5, CV_8UC1, alignmentMarker);
        const float module_size = test_perspective_size / version_size;
        Mat resizedAlignmentMarker;
        resize(alignmentMarkerMat, resizedAlignmentMarker,
               Size(cvRound(module_size * 5.f), cvRound(module_size * 5.f)), 0, 0, INTER_AREA);
        const float module_offset = 1.9f;
        const float offset = (module_size * (5 + module_offset * 2)); // 5 modules in alignment marker, 2 x module_offset modules in offset
        for (const pair<int, int>& alignmentPos : alignmentPositions) {
            const float left_top_x = (module_size * (alignmentPos.first - 2.f - module_offset)); // add offset
            const float left_top_y = (module_size * (alignmentPos.second - 2.f - module_offset)); // add offset
            Mat subImage(no_border_intermediate, Rect(cvRound(left_top_x), cvRound(left_top_y), cvRound(offset), cvRound(offset)));
            Mat resTemplate;
            matchTemplate(subImage, resizedAlignmentMarker, resTemplate, TM_CCOEFF_NORMED);
            double minVal = 0., maxVal = 0.;
            Point minLoc, maxLoc, 
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

### Classes and Structures

- **QRDetect**: A class/struct defined in this file
- **PimplQRAruco**: A class/struct defined in this file
- **ImplContour**: A class/struct defined in this file
- **sortPairAsc**: A class/struct defined in this file
- **compareDistanse_y**: A class/struct defined in this file
- **compareSquare**: A class/struct defined in this file
- **sortPointsByX**: A class/struct defined in this file
- **sortPairDesc**: A class/struct defined in this file
- **ParallelDecodeProcess**: A class/struct defined in this file
- **FinderPatternInfo**: A class/struct defined in this file
- **sortPointsByY**: A class/struct defined in this file
- **BWCounter**: A class/struct defined in this file
- **QRDetectMulti**: A class/struct defined in this file
- **QRDecode**: A class/struct defined in this file
- **ParallelSearch**: A class/struct defined in this file
- **QRCode**: A class/struct defined in this file

### Functions and Methods

- **matches()**: A function/method defined in this file
- **HAVE_QUIRC()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cmath`
- `graphical_code_detector_impl.hpp`
- `opencv2/core/utils/logger.hpp`
- `array`
- `queue`
- `quirc.h`
- `limits`
- `precomp.hpp`
- `opencv2/objdetect.hpp`
- `opencv2/calib3d.hpp`
- `map`

**Python Imports:**
- `the`
- `https`


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


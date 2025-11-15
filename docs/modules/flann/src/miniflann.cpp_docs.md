# Documentation for `modules/flann/src/miniflann.cpp`

## File Metadata

- **Full Path**: `modules/flann/src/miniflann.cpp`
- **File Name**: `miniflann.cpp`
- **File Size**: 28,846 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/flann/src/miniflann.cpp](../../../modules/flann/src/miniflann.cpp)

## Purpose and Role

This file is located in the `modules/flann/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "precomp.hpp"

#define MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES 0

static cvflann::IndexParams& get_params(const cv::flann::IndexParams& p)
{
    return *(cvflann::IndexParams*)(p.params);
}

cv::flann::IndexParams::~IndexParams()
{
    delete &get_params(*this);
}

namespace cv
{

namespace flann
{

using namespace cvflann;

IndexParams::IndexParams()
{
    params = new ::cvflann::IndexParams();
}

template<typename T>
T getParam(const IndexParams& _p, const String& key, const T& defaultVal=T())
{
    ::cvflann::IndexParams& p = get_params(_p);
    ::cvflann::IndexParams::const_iterator it = p.find(key);
    if( it == p.end() )
        return defaultVal;
    return it->second.cast<T>();
}

template<typename T>
void setParam(IndexParams& _p, const String& key, const T& value)
{
    ::cvflann::IndexParams& p = get_params(_p);
    p[key] = value;
}

String IndexParams::getString(const String& key, const String& defaultVal) const
{
    return getParam(*this, key, defaultVal);
}

int IndexParams::getInt(const String& key, int defaultVal) const
{
    return getParam(*this, key, defaultVal);
}

double IndexParams::getDouble(const String& key, double defaultVal) const
{
    return getParam(*this, key, defaultVal);
}


void IndexParams::setString(const String& key, const String& value)
{
    setParam(*this, key, value);
}

void IndexParams::setInt(const String& key, int value)
{
    setParam(*this, key, value);
}

void IndexParams::setDouble(const String& key, double value)
{
    setParam(*this, key, value);
}

void IndexParams::setFloat(const String& key, float value)
{
    setParam(*this, key, value);
}

void IndexParams::setBool(const String& key, bool value)
{
    setParam(*this, key, value);
}

void IndexParams::setAlgorithm(int value)
{
    setParam(*this, "algorithm", (cvflann::flann_algorithm_t)value);
}

void IndexParams::getAll(std::vector<String>& names,
            std::vector<FlannIndexType>& types,
            std::vector<String>& strValues,
            std::vector<double>& numValues) const
{
    names.clear();
    types.clear();
    strValues.clear();
    numValues.clear();

    ::cvflann::IndexParams& p = get_params(*this);
    ::cvflann::IndexParams::const_iterator it = p.begin(), it_end = p.end();

    for( ; it != it_end; ++it )
    {
        names.push_back(it->first);
        try
        {
            String val = it->second.cast<String>();
            types.push_back(FLANN_INDEX_TYPE_STRING);
            strValues.push_back(val);
            numValues.push_back(-1);
        continue;
        }
        catch (...) {}

        strValues.push_back(it->second.type().name());

        try
        {
            double val = it->second.cast<double>();
            types.push_back(FLANN_INDEX_TYPE_64F);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            float val = it->second.cast<float>();
            types.push_back(FLANN_INDEX_TYPE_32F);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            int val = it->second.cast<int>();
            types.push_back(FLANN_INDEX_TYPE_32S);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            short val = it->second.cast<short>();
            types.push_back(FLANN_INDEX_TYPE_16S);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            ushort val = it->second.cast<ushort>();
            types.push_back(FLANN_INDEX_TYPE_16U);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            char val = it->second.cast<char>();
            types.push_back(FLANN_INDEX_TYPE_8S);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            uchar val = it->second.cast<uchar>();
            types.push_back(FLANN_INDEX_TYPE_8U);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            bool val = it->second.cast<bool>();
            types.push_back(FLANN_INDEX_TYPE_BOOL);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}
        try
        {
            cvflann::flann_algorithm_t val = it->second.cast<cvflann::flann_algorithm_t>();
            types.push_back(FLANN_INDEX_TYPE_ALGORITHM);
            numValues.push_back(val);
        continue;
        }
        catch (...) {}


        types.push_back((FlannIndexType)-1); // unknown type
        numValues.push_back(-1);
    }
}


KDTreeIndexParams::KDTreeIndexParams(int trees)
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_KDTREE;
    p["trees"] = trees;
}

LinearIndexParams::LinearIndexParams()
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_LINEAR;
}

CompositeIndexParams::CompositeIndexParams(int trees, int branching, int iterations,
                             flann_centers_init_t centers_init, float cb_index )
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_KMEANS;
    // number of randomized trees to use (for kdtree)
    p["trees"] = trees;
    // branching factor
    p["branching"] = branching;
    // max iterations to perform in one kmeans clustering (kmeans tree)
    p["iterations"] = iterations;
    // algorithm used for picking the initial cluster centers for kmeans tree
    p["centers_init"] = centers_init;
    // cluster boundary index. Used when searching the kmeans tree
    p["cb_index"] = cb_index;
}

AutotunedIndexParams::AutotunedIndexParams(float target_precision, float build_weight,
                                           float memory_weight, float sample_fraction)
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_AUTOTUNED;
    // precision desired (used for autotuning, -1 otherwise)
    p["target_precision"] = target_precision;
    // build tree time weighting factor
    p["build_weight"] = build_weight;
    // index memory weighting factor
    p["memory_weight"] = memory_weight;
    // what fraction of the dataset to use for autotuning
    p["sample_fraction"] = sample_fraction;
}


KMeansIndexParams::KMeansIndexParams(int branching, int iterations,
                  flann_centers_init_t centers_init, float cb_index )
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_KMEANS;
    // branching factor
    p["branching"] = branching;
    // max iterations to perform in one kmeans clustering (kmeans tree)
    p["iterations"] = iterations;
    // algorithm used for picking the initial cluster centers for kmeans tree
    p["centers_init"] = centers_init;
    // cluster boundary index. Used when searching the kmeans tree
    p["cb_index"] = cb_index;
}

HierarchicalClusteringIndexParams::HierarchicalClusteringIndexParams(int branching ,
                                      flann_centers_init_t centers_init,
                                      int trees, int leaf_size)
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_HIERARCHICAL;
    // The branching factor used in the hierarchical clustering
    p["branching"] = branching;
    // Algorithm used for picking the initial cluster centers
    p["centers_init"] = centers_init;
    // number of parallel trees to build
    p["trees"] = trees;
    // maximum leaf size
    p["leaf_size"] = leaf_size;
}

LshIndexParams::LshIndexParams(int table_number, int key_size, int multi_probe_level)
{
    ::cvflann::IndexParams& p = get_params(*this);
    p["algorithm"] = FLANN_INDEX_LSH;
    // The number of hash tables to use
    p["table_number"] = table_number;
    // The length of the key in the hash tables
    p["key_size"] = key_size;
    // Number of levels to use in multi-probe (0 for standard LSH)
    p["multi_probe_level"] = multi_probe_level;
}

SavedIndexParams::SavedIndexParams(const String& _filename)
{
    String filename = _filename;
    ::cvflann::IndexParams& p = get_params(*this);

    p["algorithm"] = FLANN_INDEX_SAVED;
    p["filename"] = filename;
}

SearchParams::SearchParams( int checks, float eps, bool sorted, bool explore_all_trees )
{
    ::cvflann::IndexParams& p = get_params(*this);

    // how many leafs to visit when searching for neighbours (-1 for unlimited)
    p["checks"] = checks;
    // search for eps-approximate neighbours (default: 0)
    p["eps"] = eps;
    // only for radius search, require neighbours sorted by distance (default: true)
    p["sorted"] = sorted;
    // if false, search stops at the tree reaching the number of  max checks (original behavior).
    // When true, we do a descent in each tree and. Like before the alternative paths
    // stored in the heap are not be processed further when max checks is reached.
    p["explore_all_trees"] = explore_all_trees;
}


SearchParams::SearchParams( int checks, float eps, bool sorted )
{
    ::cvflann::IndexParams& p = get_params(*this);

    // how many leafs to visit when searching for neighbours (-1 for unlimited)
    p["checks"] = checks;
    // search for eps-approximate neighbours (default: 0)
    p["eps"] = eps;
    // only for radius search, require neighbours sorted by distance (default: true)
    p["sorted"] = sorted;
    // if false, search stops at the tree reaching the number of  max checks (original behavior).
    // When true, we do a descent in each tree and. Like before the alternative paths
    // stored in the heap are not be processed further when max checks is reached.
    p["explore_all_trees"] = false;
}


template<typename Distance, typename IndexType> void
buildIndex_(void*& index, const Mat& data, const IndexParams& params, const Distance& dist = Distance())
{
    typedef typename Distance::ElementType ElementType;
    if(DataType<ElementType>::type != data.type())
        CV_Error_(Error::StsUnsupportedFormat, ("type=%d\n", data.type()));
    if(!data.isContinuous())
        CV_Error(Error::StsBadArg, "Only continuous arrays are supported");

    ::cvflann::Matrix<ElementType> dataset((ElementType*)data.data, data.rows, data.cols);
    IndexType* _index = new IndexType(dataset, get_params(params), dist);

    try
    {
        _index->buildIndex();
    }
    catch (...)
    {
        delete _index;
        _index = NULL;

        throw;
    }

    index = _index;
}

template<typename Distance> void
buildIndex(void*& index, const Mat& data, const IndexParams& params, const Distance& dist = Distance())
{
    buildIndex_<Distance, ::cvflann::Index<Distance> >(index, data, params, dist);
}

#if CV_NEON
typedef ::cvflann::Hamming<uchar> HammingDistance;
#else
typedef ::cvflann::HammingLUT HammingDistance;
#endif
typedef ::cvflann::DNAmming2<uchar> DNAmmingDistance;

Index::Index()
{
    index = 0;
    featureType = CV_32F;
    algo = FLANN_INDEX_LINEAR;
    distType = FLANN_DIST_L2;
}

Index::Index(InputArray _data, const IndexParams& params, flann_distance_t _distType)
{
    index = 0;
    featureType = CV_32F;
    algo = FLANN_INDEX_LINEAR;
    distType = FLANN_DIST_L2;
    build(_data, params, _distType);
}

void Index::build(InputArray _data, const IndexParams& params, flann_distance_t _distType)
{
    CV_INSTRUMENT_REGION();

    release();

    // Index may reuse 'data' during search, need to keep it alive
    features_clone = _data.getMat().clone();
    Mat data = features_clone;

    algo = getParam<flann_algorithm_t>(params, "algorithm", FLANN_INDEX_LINEAR);
    if( algo == FLANN_INDEX_SAVED )
    {
        load_(getParam<String>(params, "filename", String()));
        return;
    }

    index = 0;
    featureType = data.type();
    distType = _distType;

    if ( algo == FLANN_INDEX_LSH)
    {
        distType = FLANN_DIST_HAMMING;
    }

    switch( distType )
    {
    case FLANN_DIST_HAMMING:
        buildIndex< HammingDistance >(index, data, params);
        break;
    case FLANN_DIST_L2:
        buildIndex< ::cvflann::L2<float> >(index, data, params);
        break;
    case FLANN_DIST_L1:
        buildIndex< ::cvflann::L1<float> >(index, data, params);
        break;
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
    case FLANN_DIST_DNAMMING:
        buildIndex< DNAmmingDistance >(index, data, params);
        break;
    case FLANN_DIST_MAX:
        buildIndex< ::cvflann::MaxDistance<float> >(index, data, params);
        break;
    case FLANN_DIST_HIST_INTERSECT:
        buildIndex< ::cvflann::HistIntersectionDistance<float> >(index, data, params);
        break;
    case FLANN_DIST_HELLINGER:
        buildIndex< ::cvflann::HellingerDistance<float> >(index, data, params);
        break;
    case FLANN_DIST_CHI_SQUARE:
        buildIndex< ::cvflann::ChiSquareDistance<float> >(index, data, params);
        break;
    case FLANN_DIST_KL:
        buildIndex< ::cvflann::KL_Divergence<float> >(index, data, params);
        break;
#endif
    default:
        CV_Error(Error::StsBadArg, "Unknown/unsupported distance type");
    }
}

template<typename IndexType> void deleteIndex_(void* index)
{
    delete (IndexType*)index;
}

template<typename Distance> void deleteIndex(void* index)
{
    deleteIndex_< ::cvflann::Index<Distance> >(index);
}

Index::~Index()
{
    release();
}

void Index::release()
{
    CV_INSTRUMENT_REGION();

    features_clone.release();

    if( !index )
        return;

    switch( distType )
    {
        case FLANN_DIST_HAMMING:
            deleteIndex< HammingDistance >(index);
            break;
        case FLANN_DIST_L2:
            deleteIndex< ::cvflann::L2<float> >(index);
            break;
        case FLANN_DIST_L1:
            deleteIndex< ::cvflann::L1<float> >(index);
            break;
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
        case FLANN_DIST_DNAMMING:
            deleteIndex< DNAmmingDistance >(index);
            break;
        case FLANN_DIST_MAX:
            deleteIndex< ::cvflann::MaxDistance<float> >(index);
            break;
        case FLANN_DIST_HIST_INTERSECT:
            deleteIndex< ::cvflann::HistIntersectionDistance<float> >(index);
            break;
        case FLANN_DIST_HELLINGER:
            deleteIndex< ::cvflann::HellingerDistance<float> >(index);
            break;
        case FLANN_DIST_CHI_SQUARE:
            deleteIndex< ::cvflann::ChiSquareDistance<float> >(index);
            break;
        case FLANN_DIST_KL:
            deleteIndex< ::cvflann::KL_Divergence<float> >(index);
            break;
#endif
        default:
            CV_Error(Error::StsBadArg, "Unknown/unsupported distance type");
    }
    index = 0;
}

template<typename Distance, typename IndexType>
void runKnnSearch_(void* index, const Mat& query, Mat& indices, Mat& dists,
                  int knn, const SearchParams& params)
{
    typedef typename Distance::ElementType ElementType;
    typedef typename Distance::ResultType DistanceType;
    int type = DataType<ElementType>::type;
    int dtype = DataType<DistanceType>::type;
    IndexType* index_ = (IndexType*)index;

    CV_Assert((size_t)knn <= index_->size());
    CV_Assert(query.type() == type && indices.type() == CV_32S && dists.type() == dtype);
    CV_Assert(query.isContinuous() && indices.isContinuous() && dists.isContinuous());

    ::cvflann::Matrix<ElementType> _query((ElementType*)query.data, query.rows, query.cols);
    ::cvflann::Matrix<int> _indices(indices.ptr<int>(), indices.rows, indices.cols);
    ::cvflann::Matrix<DistanceType> _dists(dists.ptr<DistanceType>(), dists.rows, dists.cols);

    index_->knnSearch(_query, _indices, _dists, knn,
                      (const ::cvflann::SearchParams&)get_params(params));
}

template<typename Distance>
void runKnnSearch(void* index, const Mat& query, Mat& indices, Mat& dists,
                  int knn, const SearchParams& params)
{
    runKnnSearch_<Distance, ::cvflann::Index<Distance> >(index, query, indices, dists, knn, params);
}

template<typename Distance, typename IndexType>
int runRadiusSearch_(void* index, const Mat& query, Mat& indices, Mat& dists,
                    double radius, const SearchParams& params)
{
    typedef typename Distance::ElementType ElementType;
    typedef typename Distance::ResultType DistanceType;
    int type = DataType<ElementType>::type;
    int dtype = DataType<DistanceType>::type;
    CV_Assert(query.type() == type && indices.type() == CV_32S && dists.type() == dtype);
    CV_Assert(query.isContinuous() && indices.isContinuous() && dists.isContinuous());

    ::cvflann::Matrix<ElementType> _query((ElementType*)query.data, query.rows, query.cols);
    ::cvflann::Matrix<int> _indices(indices.ptr<int>(), indices.rows, indices.cols);
    ::cvflann::Matrix<DistanceType> _dists(dists.ptr<DistanceType>(), dists.rows, dists.cols);

    return ((IndexType*)index)->radiusSearch(_query, _indices, _dists,
                                            saturate_cast<float>(radius),
                                            (const ::cvflann::SearchParams&)get_params(params));
}

template<typename Distance>
int runRadiusSearch(void* index, const Mat& query, Mat& indices, Mat& dists,
                     double radius, const SearchParams& params)
{
    return runRadiusSearch_<Distance, ::cvflann::Index<Distance> >(index, query, indices, dists, radius, params);
}


static void createIndicesDists(OutputArray _indices, OutputArray _dists,
                               Mat& indices, Mat& dists, int rows,
                               int minCols, int maxCols, int dtype)
{
    if( _indices.needed() )
    {
        indices = _indices.getMat();
        if( !indices.isContinuous() || indices.type() != CV_32S ||
            indices.rows != rows || indices.cols < minCols || indices.cols > maxCols )
        {
            if( !indices.isContinuous() )
               _indices.release();
            _indices.create( rows, minCols, CV_32S );
            indices = _indices.getMat();
        }
    }
    else
        indices.create( rows, minCols, CV_32S );

    if( _dists.needed() )
    {
        dists = _dists.getMat();
        if( !dists.isContinuous() || dists.type() != dtype ||
           dists.rows != rows || dists.cols < minCols || dists.cols > maxCols )
        {
            if( !_dists.isContinuous() )
                _dists.release();
            _dists.create( rows, minCols, dtype );
            dists = _dists.getMat();
        }
    }
    else
        dists.create( rows, minCols, dtype );
}


void Index::knnSearch(InputArray _query, OutputArray _indices,
               OutputArray _dists, int knn, const SearchParams& params)
{
    CV_INSTRUMENT_REGION();

    Mat query = _query.getMat(), indices, dists;
    int dtype = (distType == FLANN_DIST_HAMMING)
                || (distType == FLANN_DIST_DNAMMING) ? CV_32S : CV_32F;

    createIndicesDists( _indices, _dists, indices, dists, query.rows, knn, knn, dtype );

    switch( distType )
    {
    case FLANN_DIST_HAMMING:
        runKnnSearch<HammingDistance>(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_L2:
        runKnnSearch< ::cvflann::L2<float> >(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_L1:
        runKnnSearch< ::cvflann::L1<float> >(index, query, indices, dists, knn, params);
        break;
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
    case FLANN_DIST_DNAMMING:
        runKnnSearch<DNAmmingDistance>(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_MAX:
        runKnnSearch< ::cvflann::MaxDistance<float> >(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_HIST_INTERSECT:
        runKnnSearch< ::cvflann::HistIntersectionDistance<float> >(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_HELLINGER:
        runKnnSearch< ::cvflann::HellingerDistance<float> >(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_CHI_SQUARE:
        runKnnSearch< ::cvflann::ChiSquareDistance<float> >(index, query, indices, dists, knn, params);
        break;
    case FLANN_DIST_KL:
        runKnnSearch< ::cvflann::KL_Divergence<float> >(index, query, indices, dists, knn, params);
        break;
#endif
    default:
        CV_Error(Error::StsBadArg, "Unknown/unsupported distance type");
    }
}

int Index::radiusSearch(InputArray _query, OutputArray _indices,
                        OutputArray _dists, double radius, int maxResults,
                        const SearchParams& params)
{
    CV_INSTRUMENT_REGION();

    Mat query = _query.getMat(), indices, dists;
    int dtype = (distType == FLANN_DIST_HAMMING)
                || (distType == FLANN_DIST_DNAMMING) ? CV_32S : CV_32F;
    CV_Assert( maxResults > 0 );
    createIndicesDists( _indices, _dists, indices, dists, query.rows, maxResults, INT_MAX, dtype );

    if( algo == FLANN_INDEX_LSH )
        CV_Error( Error::StsNotImplemented, "LSH index does not support radiusSearch operation" );

    switch( distType )
    {
    case FLANN_DIST_HAMMING:
        return runRadiusSearch< HammingDistance >(index, query, indices, dists, radius, params);

    case FLANN_DIST_L2:
        return runRadiusSearch< ::cvflann::L2<float> >(index, query, indices, dists, radius, params);
    case FLANN_DIST_L1:
        return runRadiusSearch< ::cvflann::L1<float> >(index, query, indices, dists, radius, params);
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
    case FLANN_DIST_DNAMMING:
        return runRadiusSearch< DNAmmingDistance >(index, query, indices, dists, radius, params);
    case FLANN_DIST_MAX:
        return runRadiusSearch< ::cvflann::MaxDistance<float> >(index, query, indices, dists, radius, params);
    case FLANN_DIST_HIST_INTERSECT:
        return runRadiusSearch< ::cvflann::HistIntersectionDistance<float> >(index, query, indices, dists, radius, params);
    case FLANN_DIST_HELLINGER:
        return runRadiusSearch< ::cvflann::HellingerDistance<float> >(index, query, indices, dists, radius, params);
    case FLANN_DIST_CHI_SQUARE:
        return runRadiusSearch< ::cvflann::ChiSquareDistance<float> >(index, query, indices, dists, radius, params);
    case FLANN_DIST_KL:
        return runRadiusSearch< ::cvflann::KL_Divergence<float> >(index, query, indices, dists, radius, params);
#endif
    default:
        CV_Error(Error::StsBadArg, "Unknown/unsupported distance type");
    }
    return -1;
}

flann_distance_t Index::getDistance() const
{
    return distType;
}

flann_algorithm_t Index::getAlgorithm() const
{
    return algo;
}

template<typename IndexType> void saveIndex_(const Index* index0, const void* index, FILE* fout)
{
    IndexType* _index = (IndexType*)index;
    ::cvflann::save_header(fout, *_index);
    // some compilers may store short enumerations as bytes,
    // so make sure we always write integers (which are 4-byte values in any modern C compiler)
    int idistType = (int)index0->getDistance();
    ::cvflann::save_value<int>(fout, idistType);
    _index->saveIndex(fout);
}

template<typename Distance> void saveIndex(const Index* index0, const void* index, FILE* fout)
{
    saveIndex_< ::cvflann::Index<Distance> >(index0, index, fout);
}

void Index::save(const String& filename) const
{
    CV_INSTRUMENT_REGION();

    FILE* fout = fopen(filename.c_str(), "wb");
    if (fout == NULL)
        CV_Error_( Error::StsError, ("Can not open file %s for writing FLANN index\n", filename.c_str()) );

    switch( distType )
    {
    case FLANN_DIST_HAMMING:
        saveIndex< HammingDistance >(this, index, fout);
        break;
    case FLANN_DIST_L2:
        saveIndex< ::cvflann::L2<float> >(this, index, fout);
        break;
    case FLANN_DIST_L1:
        saveIndex< ::cvflann::L1<float> >(this, index, fout);
        break;
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
    case FLANN_DIST_DNAMMING:
        saveIndex< DNAmmingDistance >(this, index, fout);
        break;
    case FLANN_DIST_MAX:
        saveIndex< ::cvflann::MaxDistance<float> >(this, index, fout);
        break;
    case FLANN_DIST_HIST_INTERSECT:
        saveIndex< ::cvflann::HistIntersectionDistance<float> >(this, index, fout);
        break;
    case FLANN_DIST_HELLINGER:
        saveIndex< ::cvflann::HellingerDistance<float> >(this, index, fout);
        break;
    case FLANN_DIST_CHI_SQUARE:
        saveIndex< ::cvflann::ChiSquareDistance<float> >(this, index, fout);
        break;
    case FLANN_DIST_KL:
        saveIndex< ::cvflann::KL_Divergence<float> >(this, index, fout);
        break;
#endif
    default:
        fclose(fout);
        fout = 0;
        CV_Error(Error::StsBadArg, "Unknown/unsupported distance type");
    }
    if( fout )
        fclose(fout);
}


template<typename Distance, typename IndexType>
bool loadIndex_(Index* index0, void*& index, const Mat& data, FILE* fin, const Distance& dist=Distance())
{
    typedef typename Distance::ElementType ElementType;
    CV_Assert(DataType<ElementType>::type == data.type() && data.isContinuous());

    ::cvflann::Matrix<ElementType> dataset((ElementType*)data.data, data.rows, data.cols);

    ::cvflann::IndexParams params;
    params["algorithm"] = index0->getAlgorithm();
    IndexType* _index = new IndexType(dataset, params, dist);
    _index->loadIndex(fin);
    index = _index;
    return true;
}

template<typename Distance>
bool loadIndex(Index* index0, void*& index, const Mat& data, FILE* fin, const Distance& dist=Distance())
{
    return loadIndex_<Distance, ::cvflann::Index<Distance> >(index0, index, data, fin, dist);
}

bool Index::load(InputArray _data, const String& filename)
{
    release();

    // Index may reuse 'data' during search, need to keep it alive
    features_clone = _data.getMat().clone();
    Mat data = features_clone;

    return load_(filename);
}

bool Index::load_(const String& filename)
{
    Mat data = features_clone;
    bool ok = true;

    FILE* fin = fopen(filename.c_str(), "rb");
    if (fin == NULL) {
        return false;
    }
    FILEScopeGuard fscgd(fin);

    ::cvflann::IndexHeader header = ::cvflann::load_header(fin);

    algo = header.index_type;
    featureType = header.data_type == FLANN_UINT8 ? CV_8U :
                  header.data_type == FLANN_INT8 ? CV_8S :
                  header.data_type == FLANN_UINT16 ? CV_16U :
                  header.data_type == FLANN_INT16 ? CV_16S :
                  header.data_type == FLANN_INT32 ? CV_32S :
                  header.data_type == FLANN_FLOAT32 ? CV_32F :
                  header.data_type == FLANN_FLOAT64 ? CV_64F : -1;

    if( (int)header.rows != data.rows || (int)header.cols != data.cols ||
        featureType != data.type() )
    {
        fprintf(stderr, "Reading FLANN index error: the saved data size (%d, %d) or type (%d) is different from the passed one (%d, %d), %d\n",
                (int)header.rows, (int)header.cols, featureType, data.rows, data.cols, data.type());
        return false;
    }

    int idistType = 0;
    ::cvflann::load_value(fin, idistType);
    distType = (flann_distance_t)idistType;

    if( !((distType == FLANN_DIST_HAMMING && featureType == CV_8U) ||
          (distType == FLANN_DIST_DNAMMING && featureType == CV_8U) ||
          (distType != FLANN_DIST_HAMMING && featureType == CV_32F)) )
    {
        fprintf(stderr, "Reading FLANN index error: unsupported feature type %d for the index type %d\n", featureType, algo);
        return false;
    }

    switch( distType )
    {
    case FLANN_DIST_HAMMING:
        loadIndex< HammingDistance >(this, index, data, fin);
        break;
    case FLANN_DIST_L2:
        loadIndex< ::cvflann::L2<float> >(this, index, data, fin);
        break;
    case FLANN_DIST_L1:
        loadIndex< ::cvflann::L1<float> >(this, index, data, fin);
        break;
#if MINIFLANN_SUPPORT_EXOTIC_DISTANCE_TYPES
    case FLANN_DIST_DNAMMING:
        loadIndex< DNAmmingDistance >(this, index, data, fin);
        break;
    case FLANN_DIST_MAX:
        loadIndex< ::cvflann::MaxDistance<float> >(this, index, data, fin);
        break;
    case FLANN_DIST_HIST_INTERSECT:
        loadIndex< ::cvflann::HistIntersectionDistance<float> >(this, index, data, fin);
        break;
    case FLANN_DIST_HELLINGER:
        loadIndex< ::cvflann::HellingerDistance<float> >(this, index, data, fin);
        break;
    case FLANN_DIST_CHI_SQUARE:
        loadIndex< ::cvflann::ChiSquareDistance<float> >(this, index, data, fin);
        break;
    case FLANN_DIST_KL:
        loadIndex< ::cvflann::KL_Divergence<float> >(this, index, data, fin);
        break;
#endif
    default:
        fprintf(stderr, "Reading FLANN index error: unsupported distance type %d\n", distType);
        ok = false;
    }

    return ok;
}

}

}
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

- **typename()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`

**Python Imports:**
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


# Documentation for `samples/cpp/travelsalesman.cpp`

## File Metadata

- **Full Path**: `samples/cpp/travelsalesman.cpp`
- **File Name**: `travelsalesman.cpp`
- **File Size**: 2,829 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/travelsalesman.cpp](../../samples/cpp/travelsalesman.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/core.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/ml.hpp>

using namespace cv;

class TravelSalesman
{
private :
    const std::vector<Point>& posCity;
    std::vector<int>& next;
    RNG rng;
    int d0,d1,d2,d3;

public:
    TravelSalesman(std::vector<Point> &p, std::vector<int> &n) :
        posCity(p), next(n)
    {
        rng = theRNG();
    }
    /** Give energy value for a state of system.*/
    double energy() const;
    /** Function which change the state of system (random perturbation).*/
    void changeState();
    /** Function to reverse to the previous state.*/
    void reverseState();

};

void TravelSalesman::changeState()
{
    d0 = rng.uniform(0,static_cast<int>(posCity.size()));
    d1 = next[d0];
    d2 = next[d1];
    d3 = next[d2];

    next[d0] = d2;
    next[d2] = d1;
    next[d1] = d3;
}


void TravelSalesman::reverseState()
{
    next[d0] = d1;
    next[d1] = d2;
    next[d2] = d3;
}

double TravelSalesman::energy() const
{
    double e = 0;
    for (size_t i = 0; i < next.size(); i++)
    {
        e += norm(posCity[i]-posCity[next[i]]);
    }
    return e;
}


static void DrawTravelMap(Mat &img, std::vector<Point> &p, std::vector<int> &n)
{
    for (size_t i = 0; i < n.size(); i++)
    {
        circle(img,p[i],5,Scalar(0,0,255),2);
        line(img,p[i],p[n[i]],Scalar(0,255,0),2);
    }
}
int main(void)
{
    int nbCity=40;
    Mat img(500,500,CV_8UC3,Scalar::all(0));
    RNG rng(123456);
    int radius=static_cast<int>(img.cols*0.45);
    Point center(img.cols/2,img.rows/2);

    std::vector<Point> posCity(nbCity);
    std::vector<int> next(nbCity);
    for (size_t i = 0; i < posCity.size(); i++)
    {
        double theta = rng.uniform(0., 2 * CV_PI);
        posCity[i].x = static_cast<int>(radius*cos(theta)) + center.x;
        posCity[i].y = static_cast<int>(radius*sin(theta)) + center.y;
        next[i]=(i+1)%nbCity;
    }
    TravelSalesman ts_system(posCity, next);

    DrawTravelMap(img,posCity,next);
    imshow("Map",img);
    waitKey(10);
    double currentTemperature = 100.0;
    for (int i = 0, zeroChanges = 0; zeroChanges < 10; i++)
    {
        int changesApplied = ml::simulatedAnnealingSolver(ts_system, currentTemperature, currentTemperature*0.97, 0.99, 10000*nbCity, &currentTemperature, rng);
        img.setTo(Scalar::all(0));
        DrawTravelMap(img, posCity, next);
        imshow("Map", img);
        int k = waitKey(10);
        std::cout << "i=" << i << " changesApplied=" << changesApplied << " temp=" << currentTemperature << " result=" << ts_system.energy() << std::endl;
        if (k == 27 || k == 'q' || k == 'Q')
            return 0;
        if (changesApplied == 0)
            zeroChanges++;
    }
    std::cout << "Done" << std::endl;
    waitKey(0);
    return 0;
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

### Classes and Structures

- **TravelSalesman**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/ml.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/highgui.hpp`


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


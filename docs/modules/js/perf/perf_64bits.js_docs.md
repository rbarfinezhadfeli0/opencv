# Documentation for `modules/js/perf/perf_64bits.js`

## File Metadata

- **Full Path**: `modules/js/perf/perf_64bits.js`
- **File Name**: `perf_64bits.js`
- **File Size**: 5,089 bytes
- **File Type**: .js
- **Link to Source**: [modules/js/perf/perf_64bits.js](../../../modules/js/perf/perf_64bits.js)

## Purpose and Role

This file is located in the `modules/js/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
var isNodeJs = (typeof window) === 'undefined'? true : false;

if (isNodeJs) {
  var Benchmark = require('benchmark');
  var cv = require('../../opencv');
} else {
  var paramsElement = document.getElementById('params');
  var runButton = document.getElementById('runButton');
  var logElement = document.getElementById('log');
}

function perf() {

  console.log('opencv.js loaded');
  if (isNodeJs) {
    global.cv = cv;
  } else {
    runButton.removeAttribute('disabled');
    runButton.setAttribute('class', 'btn btn-primary');
    runButton.innerHTML = 'Run';
  }
  let totalCaseNum, currentCaseId;


  function addCountNonZeroCase(suite) {
    suite.add('countNonZero', function() {
      cv.countNonZero(mat);
    }, {
      'setup': function() {
        let size = this.params.size;
        let mat = cv.Mat.eye(size[0], size[1], cv.CV_64F);
      }, 'teardown': function() {
        mat.delete();
      }
    });
  }

  function addMatDotCase(suite) {
    suite.add('Mat::dot', function() {
      mat.dot(matT);
    }, {
      'setup': function() {
        let size = this.params.size;
        let mat = cv.Mat.ones(size[0], size[1], cv.CV_64FC1);
        let matT = mat.t();
      }, 'teardown': function() {
        mat.delete();
        matT.delete();
      }
    });
  }

  function addSplitCase(suite) {
    suite.add('Split', function() {
      cv.split(mat, planes);
    }, {
      'setup': function() {
        let size = this.params.size;
        let mat = cv.Mat.ones(size[0], size[1], cv.CV_64FC3);
        let planes = new cv.MatVector();
      }, 'teardown': function() {
        mat.delete();
        planes.delete();
      }
    });
  }

  function addMergeCase(suite) {
    suite.add('Merge', function() {
      cv.merge(planes, mat);
    }, {
      'setup': function() {
        let size = this.params.size;
        let mat = new cv.Mat();
        let mat1 = cv.Mat.ones(size[0], size[1], cv.CV_64FC3);
        let planes = new cv.MatVector();
        cv.split(mat1, planes);
      }, 'teardown': function() {
        mat.delete();
        mat1.delete();
        planes.delete();
      }
    });
  }

  function setInitParams(suite, sizeArray) {
    for( let i =0; i < suite.length; i++) {
      suite[i].params = {
        size: sizeArray
      };
    }
  }

  function log(message) {
    console.log(message);
    if (!isNodeJs) {
      logElement.innerHTML += `\n${'\t' + message}`;
    }
  }

  function setBenchmarkSuite(suite) {
    suite
    // add listeners
    .on('cycle', function(event) {
      ++currentCaseId;
      let size = event.target.params.size;
      log(`=== ${event.target.name} ${currentCaseId} ===`);
      log(`params: (${parseInt(size[0])}x${parseInt(size[1])})`);
      log('elapsed time:' +String(event.target.times.elapsed*1000)+' ms');
      log('mean time:' +String(event.target.stats.mean*1000)+' ms');
      log('stddev time:' +String(event.target.stats.deviation*1000)+' ms');
      log(String(event.target));
    })
    .on('error', function(event) { log(`test case ${event.target.name} failed`); })
    .on('complete', function(event) {
      log(`\n ###################################`)
      log(`Finished testing ${event.currentTarget.length} cases \n`);
      if (!isNodeJs) {
        runButton.removeAttribute('disabled');
        runButton.setAttribute('class', 'btn btn-primary');
        runButton.innerHTML = 'Run';
      }
    });
  }

  function genBenchmarkCase(paramsContent) {
    let suite = new Benchmark.Suite;
    var sizeArray;
    totalCaseNum = 4;
    currentCaseId = 0;
    if (/\([0-9]+x[0-9]+\)/g.test(paramsContent.toString())) {
      let params = paramsContent.toString().match(/\([0-9]+x[0-9]+\)/g)[0];
      let sizeStrs = (params.match(/[0-9]+/g) || []).slice(0, 2).toString().split(",");
      sizeArray = sizeStrs.map(Number);
    } else {
      log("no getting invalid params, run all the cases with Mat of shape (1000 x 1000)");
      sizeArray = [1000, 1000];
    }
    addCountNonZeroCase(suite);
    addMatDotCase(suite);
    addSplitCase(suite);
    addMergeCase(suite);
    setInitParams(suite, sizeArray)
    setBenchmarkSuite(suite);
    log(`Running ${totalCaseNum} tests from 64-bit intrinsics`);
    suite.run({ 'async': true }); // run the benchmark
  }


  // set test filter params
  if (isNodeJs) {
    const args = process.argv.slice(2);
    let paramsContent = '';
    if (/--test_param_filter=\([0-9]+x[0-9]+,[\ ]*\w+\)/g.test(args.toString())) {
      paramsContent = args.toString().match(/\([0-9]+x[0-9]+,[\ ]*\w+\)/g)[0];
    }
    genBenchmarkCase(paramsContent);
  } else {
    runButton.onclick = function()　{
      let paramsContent = paramsElement.value;
      genBenchmarkCase(paramsContent);
      if (totalCaseNum !== 0) {
        runButton.setAttribute("disabled", "disabled");
        runButton.setAttribute('class', 'btn btn-primary disabled');
        runButton.innerHTML = "Running";
      }
    }
  }
};

async function main() {
  if (cv instanceof Promise) {
    cv = await cv;
    perf();
  } else {
    cv.onRuntimeInitialized = perf;
  }
}

main();```

## High-Level Overview

This is a .js source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **addCountNonZeroCase()**: A function/method defined in this file
- **addMatDotCase()**: A function/method defined in this file
- **perf()**: A function/method defined in this file
- **addSplitCase()**: A function/method defined in this file
- **addMergeCase()**: A function/method defined in this file
- **setBenchmarkSuite()**: A function/method defined in this file
- **genBenchmarkCase()**: A function/method defined in this file
- **log()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **setInitParams()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `64`


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


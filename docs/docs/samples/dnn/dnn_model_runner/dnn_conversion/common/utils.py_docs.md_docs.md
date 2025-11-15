# Documentation for `docs/samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py_docs.md`
- **File Name**: `utils.py_docs.md`
- **File Size**: 9,107 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py_docs.md](../../../../../../docs/samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn/dnn_model_runner/dnn_conversion/common` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py`

## File Metadata

- **Full Path**: `samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py`
- **File Name**: `utils.py`
- **File Size**: 5,086 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py](../../../../../samples/dnn/dnn_model_runner/dnn_conversion/common/utils.py)

## Purpose and Role

This file is located in the `samples/dnn/dnn_model_runner/dnn_conversion/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import argparse
import importlib.util
import os
import random

import matplotlib.pyplot as plt
import numpy as np
import tensorflow as tf
import torch

from .test.configs.test_config import CommonConfig

SEED_VAL = 42
DNN_LIB = "DNN"
# common path for model savings
MODEL_PATH_ROOT = os.path.join(CommonConfig().output_data_root_dir, "{}/models")


def get_full_model_path(lib_name, model_full_name):
    model_path = MODEL_PATH_ROOT.format(lib_name)
    return {
        "path": model_path,
        "full_path": os.path.join(model_path, model_full_name)
    }


def plot_acc(data_list, experiment_name):
    plt.figure(figsize=[8, 6])
    plt.plot(data_list[:, 0], "r", linewidth=2.5, label="Original Model")
    plt.plot(data_list[:, 1], "b", linewidth=2.5, label="Converted DNN Model")
    plt.xlabel("Iterations ", fontsize=15)
    plt.ylabel("Time (ms)", fontsize=15)
    plt.title(experiment_name, fontsize=15)
    plt.legend()
    full_path_to_fig = os.path.join(CommonConfig().output_data_root_dir, experiment_name + ".png")
    plt.savefig(full_path_to_fig, bbox_inches="tight")


def get_final_summary_info(general_quality_metric, general_inference_time, metric_name):
    general_quality_metric = np.array(general_quality_metric)
    general_inference_time = np.array(general_inference_time)
    summary_line = "===== End of processing. General results:\n"
    "\t* mean {} for the original model: {}\t"
    "\t* mean time (min) for the original model inferences: {}\n"
    "\t* mean {} for the DNN model: {}\t"
    "\t* mean time (min) for the DNN model inferences: {}\n".format(
        metric_name, np.mean(general_quality_metric[:, 0]),
        np.mean(general_inference_time[:, 0]) / 60000,
        metric_name, np.mean(general_quality_metric[:, 1]),
        np.mean(general_inference_time[:, 1]) / 60000,
    )
    return summary_line


def set_common_reproducibility():
    random.seed(SEED_VAL)
    np.random.seed(SEED_VAL)


def set_pytorch_env():
    set_common_reproducibility()
    torch.manual_seed(SEED_VAL)
    torch.set_printoptions(precision=10)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(SEED_VAL)
        torch.backends.cudnn_benchmark_enabled = False
        torch.backends.cudnn.deterministic = True


def set_tf_env(is_use_gpu=True):
    set_common_reproducibility()
    tf.random.set_seed(SEED_VAL)
    os.environ["TF_DETERMINISTIC_OPS"] = "1"

    if tf.config.list_physical_devices("GPU") and is_use_gpu:
        gpu_devices = tf.config.list_physical_devices("GPU")
        tf.config.experimental.set_visible_devices(gpu_devices[0], "GPU")
        tf.config.experimental.set_memory_growth(gpu_devices[0], True)
        os.environ["TF_USE_CUDNN"] = "1"
    else:
        os.environ["CUDA_VISIBLE_DEVICES"] = "-1"


def str_bool(input_val):
    if input_val.lower() in ('yes', 'true', 't', 'y', '1'):
        return True
    elif input_val.lower() in ('no', 'false', 'f', 'n', '0'):
        return False
    else:
        raise argparse.ArgumentTypeError('Boolean value was expected')


def get_formatted_model_list(model_list):
    note_line = 'Please, choose the model from the below list:\n'
    spaces_to_set = ' ' * (len(note_line) - 2)
    return note_line + ''.join([spaces_to_set, '{} \n'] * len(model_list)).format(*model_list)


def model_str(model_list):
    def type_model_list(input_val):
        if input_val.lower() in model_list:
            return input_val.lower()
        else:
            raise argparse.ArgumentTypeError(
                'The model is currently unavailable for test.\n' +
                get_formatted_model_list(model_list)
            )

    return type_model_list


def get_test_module(test_module_name, test_module_path):
    module_spec = importlib.util.spec_from_file_location(test_module_name, test_module_path)
    test_module = importlib.util.module_from_spec(module_spec)
    module_spec.loader.exec_module(test_module)
    module_spec.loader.exec_module(test_module)
    return test_module


def create_parser():
    parser = argparse.ArgumentParser(formatter_class=argparse.RawTextHelpFormatter)
    parser.add_argument(
        "--test",
        type=str_bool,
        help="Define whether you'd like to run the model with OpenCV for testing.",
        default=False
    ),
    parser.add_argument(
        "--default_img_preprocess",
        type=str_bool,
        help="Define whether you'd like to preprocess the input image with defined"
             " PyTorch or TF functions for model test with OpenCV.",
        default=False
    ),
    parser.add_argument(
        "--evaluate",
        type=str_bool,
        help="Define whether you'd like to run evaluation of the models (ex.: TF vs OpenCV networks).",
        default=True
    )
    return parser


def create_extended_parser(model_list):
    parser = create_parser()
    parser.add_argument(
        "--model_name",
        type=model_str(model_list=model_list),
        help="\nDefine the model name to test.\n" +
             get_formatted_model_list(model_list),
        required=True
    )
    return parser
```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **get_full_model_path()**: A function/method defined in this file
- **set_pytorch_env()**: A function/method defined in this file
- **set_common_reproducibility()**: A function/method defined in this file
- **get_final_summary_info()**: A function/method defined in this file
- **set_tf_env()**: A function/method defined in this file
- **model_str()**: A function/method defined in this file
- **str_bool()**: A function/method defined in this file
- **type_model_list()**: A function/method defined in this file
- **get_test_module()**: A function/method defined in this file
- **create_parser()**: A function/method defined in this file
- **get_formatted_model_list()**: A function/method defined in this file
- **create_extended_parser()**: A function/method defined in this file
- **plot_acc()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `.test.configs.test_config`
- `the`
- `os`
- `random`
- `matplotlib.pyplot`
- `importlib.util`
- `numpy`
- `torch`
- `argparse`
- `CommonConfig`
- `tensorflow`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.


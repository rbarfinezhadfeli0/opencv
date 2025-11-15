# Documentation for `modules/dnn/src/layers/attention_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/attention_layer.cpp`
- **File Name**: `attention_layer.cpp`
- **File Size**: 14,076 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/attention_layer.cpp](../../../../modules/dnn/src/layers/attention_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "cpu_kernels/fast_gemm.hpp"
#include "cpu_kernels/softmax.hpp"

#include <opencv2/dnn/shape_utils.hpp>

namespace cv { namespace dnn {

static void packWeight(size_t num_heads, size_t head_size, size_t input_hidden_size,
                       const float *weight_data, size_t hidden_size, std::vector<float> &packed_weight, const FastGemmOpt &opt) {
    // num_heads * pack(head_size, input_hidden_size)
    size_t pack_size = fastGemmPackBSize(head_size, input_hidden_size, opt);
    size_t packed_weight_size = num_heads * pack_size;
    packed_weight.resize(packed_weight_size, 0.f);
    auto *packed_weight_data = packed_weight.data();
    for (size_t i = 0; i < num_heads; i++) {
        fastGemmPackB(false, head_size, input_hidden_size, weight_data, hidden_size, packed_weight_data, opt);
        packed_weight_data += pack_size;
        weight_data += head_size;
    }
}

// Operator spec: https://github.com/microsoft/onnxruntime/blob/v1.16.1/docs/ContribOperators.md#com.microsoft.Attention
class AttentionLayerImpl CV_FINAL : public AttentionLayer {
 public:
    AttentionLayerImpl(const LayerParams &params) {
        setParamsFrom(params);

        CV_CheckTrue(params.has("num_heads"), "DNN/Attention: num_heads is required but missing");
        num_heads = params.get<int>("num_heads"); // required, no default value

        CV_CheckTrue(params.has("qkv_hidden_sizes"), "DNN/Attention: qkv_hidden_sizes is required but missing");
        auto param_qkv_hidden_sizes = params.get("qkv_hidden_sizes");
        CV_CheckEQ(param_qkv_hidden_sizes.size(), 3, "DNN/Attention: qkv_hidden_sizes must and only have three elements");

        qkv_hidden_sizes.clear();
        qkv_hidden_sizes.resize(3);
        qkv_hidden_sizes[0] = static_cast<size_t>(param_qkv_hidden_sizes.get<int>(0));
        qkv_hidden_sizes[1] = static_cast<size_t>(param_qkv_hidden_sizes.get<int>(1));
        /* v_hidden_size needs to be initialized in finalize in case v_slice_end=INT_MAX */

        qkv_head_sizes.clear();
        qkv_head_sizes.resize(3);
        qkv_head_sizes[0] = static_cast<size_t>(qkv_hidden_sizes[0] / num_heads);
        qkv_head_sizes[1] = static_cast<size_t>(qkv_hidden_sizes[1] / num_heads);

        scale = 1.f / params.get<float>("scale", sqrt(qkv_head_sizes[0]));

        output_ndims = params.get<int>("output_ndims", 3);

        is_prepacked = false;
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE {
        return backendId == DNN_BACKEND_OPENCV;
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE {
        int num_inputs = inputs.size() + blobs.size();
        CV_CheckEQ(num_inputs, 3, "DNN/Attention: three inputs are required");
        const auto &input_shape = inputs[0];
        const auto &weight_shape = blobs.empty() ? inputs[1] : shape(blobs.front());
        const auto &bias_shape = blobs.empty() ? inputs[2] : shape(blobs.back());

        CV_CheckEQ(input_shape.size(), static_cast<size_t>(3), "DNN/Attention: invalid input dimension");
        CV_CheckEQ(weight_shape.size(), static_cast<size_t>(2), "DNN/Attention: invalid weight dimension");

        CV_CheckEQ(input_shape[2], weight_shape[0], "DNN/Attention: invalid input shape");
        CV_CheckEQ(weight_shape[1], bias_shape[0], "DNN/Attention: invalid weight or bias shape");

        if (output_ndims == 3) {
            outputs.assign(1, inputs[0]);
        } else if (output_ndims == 2) {
            int batch = input_shape[0], seq_len = input_shape[1], input_hidden_size = input_shape[2];
            MatShape output_shape{batch * seq_len, input_hidden_size};
            outputs.assign(1, output_shape);
        } else {
            CV_Error(Error::StsBadArg, format("DNN/Attention: invalid output dimension %zu, valid value is 2 or 3", output_ndims));
        }

        const int batch_size_ = input_shape[0], seq_len_ = input_shape[1],
                  hidden_size_ = weight_shape.back(),
                  num_heads_ = static_cast<int>(num_heads),
                  v_head_size_ = static_cast<int>((hidden_size_ - qkv_hidden_sizes[0] - qkv_hidden_sizes[1]) / num_heads);

        MatShape gemm_buffer_shape{batch_size_, seq_len_, hidden_size_},
                 attention_prob_shape{batch_size_ * num_heads_, seq_len_, seq_len_},
                 output_buffer_shape{batch_size_ * num_heads_, seq_len_, v_head_size_};
        internals.assign(1, gemm_buffer_shape);
        internals.push_back(attention_prob_shape);
        internals.push_back(output_buffer_shape);

        return false;
    }

    virtual void finalize(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr) CV_OVERRIDE {
        opt.init();

        std::vector<Mat> inputs;
        inputs_arr.getMatVector(inputs);
        const auto input_shape = shape(inputs[0]);
        batch_size = static_cast<size_t>(input_shape[0]);
        seq_len = static_cast<size_t>(input_shape[1]);
        input_hidden_size = static_cast<size_t>(input_shape[2]);

        const auto &weight = blobs.empty() ? inputs[1] : blobs.front();
        const auto weight_shape = shape(weight);
        hidden_size = weight_shape[1];
        qkv_hidden_sizes[2] = hidden_size - qkv_hidden_sizes[0] - qkv_hidden_sizes[1];
        qkv_head_sizes[2] = static_cast<size_t>(qkv_hidden_sizes[2] / num_heads);

        if (!blobs.empty()) {
            const auto *weight_data = weight.ptr<const float>();
            packWeight(num_heads, qkv_head_sizes[0], input_hidden_size, weight_data,                                             hidden_size, packed_weight_q, opt);
            packWeight(num_heads, qkv_head_sizes[1], input_hidden_size, weight_data + qkv_hidden_sizes[0],                       hidden_size, packed_weight_k, opt);
            packWeight(num_heads, qkv_head_sizes[2], input_hidden_size, weight_data + qkv_hidden_sizes[0] + qkv_hidden_sizes[1], hidden_size, packed_weight_v, opt);

            is_prepacked = true;
        }
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        if (inputs_arr.depth() == CV_16F)
        {
            forward_fallback(inputs_arr, outputs_arr, internals_arr);
            return;
        }

        std::vector<Mat> inputs, outputs, internals;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);
        internals_arr.getMatVector(internals);

        // prepack weights
        if (!is_prepacked) {
            const auto &weight = blobs.empty() ? inputs[1] : blobs.front();
            const auto *weight_data = weight.ptr<const float>();
            packWeight(num_heads, qkv_head_sizes[0], input_hidden_size, weight_data,                                             hidden_size, packed_weight_q, opt);
            packWeight(num_heads, qkv_head_sizes[1], input_hidden_size, weight_data + qkv_hidden_sizes[0],                       hidden_size, packed_weight_k, opt);
            packWeight(num_heads, qkv_head_sizes[2], input_hidden_size, weight_data + qkv_hidden_sizes[0] + qkv_hidden_sizes[1], hidden_size, packed_weight_v, opt);

            is_prepacked = true;
        }

        float *packed_weights[3] = {packed_weight_q.data(), packed_weight_k.data(), packed_weight_v.data()};
        size_t packed_weights_size[3] = {packed_weight_q.size() / num_heads, packed_weight_k.size() / num_heads, packed_weight_v.size() / num_heads};

        // Compute Q/K/V
        auto &gemm_buffer = internals[0];
        auto *Q = gemm_buffer.ptr<float>();
        auto *K = Q + batch_size * seq_len * qkv_hidden_sizes[0];
        auto *V = K + batch_size * seq_len * qkv_hidden_sizes[1];
        float *QKV[3] = {Q, K, V}; // Q, K, V: [B, N, S, H]
        {
            const auto &input = inputs[0];
            const auto &bias = blobs.empty() ? inputs[2] : blobs.back();
            const auto *input_data = input.ptr<const float>();
            const auto *bias_data = bias.ptr<const float>();

            opt.multi_thread = false;
            auto fn = [&](const Range &r) {
                for (int i = r.start; i < r.end; i++) {
                    const int batch_index = static_cast<int>((i / 3) / num_heads);
                    const int head_index = static_cast<int>((i / 3) % num_heads);
                    const int qkv_index = static_cast<int>(i % 3);

                    auto *dst = QKV[qkv_index];
                    size_t head_size = qkv_head_sizes[qkv_index];

                    int input_offset = batch_index * seq_len * input_hidden_size;
                    int bias_offset = qkv_index * qkv_hidden_sizes[0] + head_index * head_size;
                    int dst_offset = (batch_index * num_heads + head_index) * (seq_len * head_size);

                    // broadcast bias ([NH] -> [BN, SH]) and make copy to dst
                    const auto *bias_data_src = bias_data + bias_offset;
                    auto *dst_data = dst + dst_offset;
                    for (size_t seq_len_idx = 0; seq_len_idx < seq_len; seq_len_idx++) {
                        std::memcpy(dst_data, bias_data_src, head_size * sizeof(float));
                        dst_data += head_size;
                    }

                    auto *packed_weight = packed_weights[qkv_index] + packed_weights_size[qkv_index] * head_index;
                    // single-thread gemm kernel
                    fastGemm(false, seq_len, head_size, input_hidden_size,
                            1.f, input_data + input_offset, input_hidden_size,
                            packed_weight, 1.f, dst + dst_offset, head_size, opt);
                }
            };

            size_t loops = 3 * batch_size * num_heads;
            double nstripes = loops * seq_len * qkv_head_sizes[0] * input_hidden_size * (1 / 1024.0);
            parallel_for_(Range(0, loops), fn, nstripes);
        }

        // Compute Softmax(scale * MatMul(Q, K))
        auto &attention_prob = internals[1];
        {
            auto *output = attention_prob.ptr<float>();

            auto loops = batch_size * num_heads;
            auto seq_len_square = seq_len * seq_len;
            auto qk_head_size = qkv_head_sizes[0];
            auto qk_inner_size = seq_len * qk_head_size;

            // Compute scale * matmul(Q, K)
            opt.multi_thread = false;
            parallel_for_(Range(0, loops), [&] (const Range r) {
                for (int i = r.start; i < r.end; i++) {
                    const int output_offset = i * seq_len_square;

                    const auto *q = Q + qk_inner_size * i, *k = K + qk_inner_size * i;
                    fastGemm(false, true, seq_len, qk_head_size, seq_len, qk_head_size,
                             scale, q, qk_head_size, 1,
                             k, qk_head_size, 1, 0.f,
                             output + output_offset, seq_len, opt);
                }
            }, loops * seq_len * qk_head_size * seq_len * (1 / 1024.0));

            // Compute softmax on the last dimension
            softmax(attention_prob, attention_prob, shape(attention_prob).size() - 1);
        }

        // Compute MatMul(attention_prob, V)
        auto &output_buffer = internals[2];
        {
            auto *output = outputs[0].ptr<float>();
            auto *output_buff = output_buffer.ptr<float>();
            const auto *prob = attention_prob.ptr<const float>();

            auto loops = batch_size * num_heads;
            auto prob_inner_size = seq_len * seq_len;
            auto v_head_size = qkv_head_sizes[2];
            auto v_inner_size = seq_len * v_head_size;

            opt.multi_thread = false;
            parallel_for_(Range(0, loops), [&] (const Range &r) {
                for (int i = r.start; i < r.end; i++) {
                    const int output_offset = i * v_inner_size;

                    const auto *p = prob + i * prob_inner_size, *v = V + i * v_inner_size;
                    fastGemm(false, false, seq_len, seq_len, seq_len, v_head_size,
                             1.f, p, seq_len, 1,
                             v, v_head_size, 1, 0.f,
                             output_buff + output_offset, v_head_size, opt);

                    // tranpose on the fly
                    const int batch_index = static_cast<int>(i / num_heads);
                    const int head_index = static_cast<int>(i % num_heads);
                    auto *src = output_buff + output_offset;
                    auto *dst = output + (batch_index * seq_len * num_heads + head_index) * v_head_size;
                    for (int j = 0; j < seq_len; j++) {
                        std::memcpy(dst, src, v_head_size * sizeof(float));
                        src += v_head_size;
                        dst += qkv_hidden_sizes[2];
                    }
                }
            }, loops * seq_len * seq_len * v_head_size * (1 / 1024.0));
        }
    }

 private:
    size_t num_heads;
    std::vector<size_t> qkv_hidden_sizes; // order: {qk_hidden_size, qk_hidden_size, v_hidden_size}
    float scale;
    size_t output_ndims;

    std::vector<size_t> qkv_head_sizes; // order: {qk_head_size, qk_head_size, v_head_size}

    size_t batch_size;
    size_t seq_len;
    size_t input_hidden_size;
    size_t hidden_size;

    bool is_prepacked;
    std::vector<float> packed_weight_q;
    std::vector<float> packed_weight_k;
    std::vector<float> packed_weight_v;

    FastGemmOpt opt;
};

Ptr<AttentionLayer> AttentionLayer::create(const LayerParams &params) {
    return makePtr<AttentionLayerImpl>(params);
}

}} // cv::dnn
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

- **AttentionLayerImpl**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cpu_kernels/fast_gemm.hpp`
- `cpu_kernels/softmax.hpp`
- `../precomp.hpp`
- `opencv2/dnn/shape_utils.hpp`


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


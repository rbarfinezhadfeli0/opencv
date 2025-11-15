# Documentation for `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimpleVertexShader.hlsl`

## File Metadata

- **Full Path**: `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimpleVertexShader.hlsl`
- **File Name**: `SimpleVertexShader.hlsl`
- **File Size**: 1,023 bytes
- **File Type**: .hlsl
- **Link to Source**: [samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimpleVertexShader.hlsl](../../../../../samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimpleVertexShader.hlsl)

## Purpose and Role

This file is located in the `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
cbuffer ModelViewProjectionConstantBuffer : register(b0)
{
    matrix model;
    matrix view;
    matrix projection;
};

struct VertexInputType
{
    float4 position : POSITION;
    float2 tex : TEXCOORD0;
};

struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
};


////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType main(VertexInputType input)
{
    PixelInputType output;

    // Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

    // Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, model);
    output.position = mul(output.position, view);
    output.position = mul(output.position, projection);
    // Store the texture coordinates for the pixel shader.
    output.tex = input.tex;

    return output;
}

```

## General Information

This file is part of the OpenCV repository infrastructure.


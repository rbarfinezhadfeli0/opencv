# Documentation for `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl_docs.md`

## File Metadata

- **Full Path**: `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl_docs.md`
- **File Name**: `SimplePixelShader.hlsl_docs.md`
- **File Size**: 1,554 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl_docs.md](../../../../../../docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl_docs.md)

## Purpose and Role

This file is located in the `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl`

## File Metadata

- **Full Path**: `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl`
- **File Name**: `SimplePixelShader.hlsl`
- **File Size**: 634 bytes
- **File Type**: .hlsl
- **Link to Source**: [samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl](../../../../../samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp/SimplePixelShader.hlsl)

## Purpose and Role

This file is located in the `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1Comp` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Texture2D shaderTexture;
SamplerState SampleType;

//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
};

////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 main(PixelInputType input) : SV_TARGET
{
    float4 textureColor;


    // Sample the pixel color from the texture using the sampler at this texture coordinate location.
    textureColor = shaderTexture.Sample(SampleType, input.tex);

    return textureColor;
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.


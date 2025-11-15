# Documentation for `modules/python/src2/typing_stubs_generation/api_refinement.py`

## File Metadata

- **Full Path**: `modules/python/src2/typing_stubs_generation/api_refinement.py`
- **File Name**: `api_refinement.py`
- **File Size**: 15,846 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/src2/typing_stubs_generation/api_refinement.py](../../../../modules/python/src2/typing_stubs_generation/api_refinement.py)

## Purpose and Role

This file is located in the `modules/python/src2/typing_stubs_generation` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
__all__ = [
    "apply_manual_api_refinement"
]

from typing import cast, Sequence, Callable, Iterable, Optional

from .nodes import (NamespaceNode, FunctionNode, OptionalTypeNode, TypeNode,
                    ClassProperty, PrimitiveTypeNode, ASTNodeTypeNode,
                    AggregatedTypeNode, CallableTypeNode, AnyTypeNode,
                    TupleTypeNode, UnionTypeNode, ProtocolClassNode,
                    DictTypeNode, ClassTypeNode)
from .ast_utils import (find_function_node, SymbolName,
                        for_each_function_overload)
from .types_conversion import create_type_node


def apply_manual_api_refinement(root: NamespaceNode) -> None:
    refine_highgui_module(root)
    refine_cuda_module(root)
    export_matrix_type_constants(root)
    refine_dnn_module(root)
    # Export OpenCV exception class
    builtin_exception = root.add_class("Exception")
    builtin_exception.is_exported = False
    root.add_class("error", (builtin_exception, ), ERROR_CLASS_PROPERTIES)
    for symbol_name, refine_symbol in NODES_TO_REFINE.items():
        refine_symbol(root, symbol_name)
    version_constant = root.add_constant("__version__", "<unused>")
    version_constant._value_type = "str"

    convert_returned_scalar_to_tuple(root)

    """
    def redirectError(
        onError: Callable[[int, str, str, str, int], None] | None
    ) -> None: ...
    """
    root.add_function("redirectError", [
        FunctionNode.Arg(
            "onError",
            OptionalTypeNode(
                CallableTypeNode(
                    "ErrorCallback",
                    [
                        PrimitiveTypeNode.int_(),
                        PrimitiveTypeNode.str_(),
                        PrimitiveTypeNode.str_(),
                        PrimitiveTypeNode.str_(),
                        PrimitiveTypeNode.int_()
                    ]
                )
            )
        )
    ])


def make_optional_none_return(root_node: NamespaceNode,
                              function_symbol_name: SymbolName) -> None:
    """
    Make return type Optional[MatLike],
    for the functions that may return None.
    """
    function = find_function_node(root_node, function_symbol_name)
    for overload in function.overloads:
        if overload.return_type is not None:
            if not isinstance(overload.return_type.type_node, OptionalTypeNode):
                overload.return_type.type_node = OptionalTypeNode(
                    overload.return_type.type_node
                )

def export_matrix_type_constants(root: NamespaceNode) -> None:
    MAX_PREDEFINED_CHANNELS = 4

    depth_names = ("CV_8U", "CV_8S", "CV_16U", "CV_16S", "CV_32S",
                   "CV_32F", "CV_64F", "CV_16F")
    for depth_value, depth_name in enumerate(depth_names):
        # Export depth constants
        root.add_constant(depth_name, str(depth_value))
        # Export predefined types
        for c in range(MAX_PREDEFINED_CHANNELS):
            root.add_constant(f"{depth_name}C{c + 1}",
                              f"{depth_value + 8 * c}")
        # Export type creation function
        root.add_function(
            f"{depth_name}C",
            (FunctionNode.Arg("channels", PrimitiveTypeNode.int_()), ),
            FunctionNode.RetType(PrimitiveTypeNode.int_())
        )
    # Export CV_MAKETYPE
    root.add_function(
        "CV_MAKETYPE",
        (FunctionNode.Arg("depth", PrimitiveTypeNode.int_()),
         FunctionNode.Arg("channels", PrimitiveTypeNode.int_())),
        FunctionNode.RetType(PrimitiveTypeNode.int_())
    )


def make_optional_arg(*arg_names: str) -> Callable[[NamespaceNode, SymbolName], None]:
    def _make_optional_arg(root_node: NamespaceNode,
                           function_symbol_name: SymbolName) -> None:
        function = find_function_node(root_node, function_symbol_name)
        for arg_name in arg_names:
            found_overload_with_arg = False

            for overload in function.overloads:
                arg_idx = _find_argument_index(overload.arguments, arg_name)

                # skip overloads without this argument
                if arg_idx is None:
                    continue

                # Avoid multiplying optional qualification
                if isinstance(overload.arguments[arg_idx].type_node, OptionalTypeNode):
                    continue

                overload.arguments[arg_idx].type_node = OptionalTypeNode(
                    cast(TypeNode, overload.arguments[arg_idx].type_node)
                )

                found_overload_with_arg = True

            if not found_overload_with_arg:
                raise RuntimeError(
                    f"Failed to find argument with name: '{arg_name}'"
                    f" in '{function_symbol_name.name}' overloads"
                )

    return _make_optional_arg


def convert_returned_scalar_to_tuple(root: NamespaceNode) -> None:
    """Force `tuple[float, float, float, float]` usage instead of Scalar alias
    for return types due to `pyopencv_from` specialization for Scalar type.
    """

    float_4_tuple_node = TupleTypeNode(
        "ScalarOutput",
        items=(PrimitiveTypeNode.float_(),) * 4
    )

    def fix_scalar_return_type(fn: FunctionNode.Overload):
        if fn.return_type is None:
            return
        if fn.return_type.type_node.typename == "Scalar":
            fn.return_type.type_node = float_4_tuple_node

    for overload in for_each_function_overload(root):
        fix_scalar_return_type(overload)

    for ns in root.namespaces.values():
        for overload in for_each_function_overload(ns):
            fix_scalar_return_type(overload)


def refine_cuda_module(root: NamespaceNode) -> None:
    def fix_cudaoptflow_enums_names() -> None:
        for class_name in ("NvidiaOpticalFlow_1_0", "NvidiaOpticalFlow_2_0"):
            if class_name not in cuda_root.classes:
                continue
            opt_flow_class = cuda_root.classes[class_name]
            _trim_class_name_from_argument_types(
                for_each_function_overload(opt_flow_class), class_name
            )

    def fix_namespace_usage_scope(cuda_ns: NamespaceNode) -> None:
        USED_TYPES = ("GpuMat", "Stream")

        def fix_type_usage(type_node: TypeNode) -> None:
            if isinstance(type_node, AggregatedTypeNode):
                for item in type_node.items:
                    fix_type_usage(item)
            if isinstance(type_node, ASTNodeTypeNode):
                if type_node._typename in USED_TYPES:
                    type_node._typename = f"cuda_{type_node._typename}"

        for overload in for_each_function_overload(cuda_ns):
            if overload.return_type is not None:
                fix_type_usage(overload.return_type.type_node)
            for type_node in [arg.type_node for arg in overload.arguments
                              if arg.type_node is not None]:
                fix_type_usage(type_node)

    if "cuda" not in root.namespaces:
        return
    cuda_root = root.namespaces["cuda"]
    fix_cudaoptflow_enums_names()
    for ns in [ns for ns_name, ns in root.namespaces.items()
               if ns_name.startswith("cuda")]:
        fix_namespace_usage_scope(ns)


def refine_highgui_module(root: NamespaceNode) -> None:
    # Check if library is built with enabled highgui module
    if "destroyAllWindows" not in root.functions:
        return
    """
    def createTrackbar(trackbarName: str,
                       windowName: str,
                       value: int,
                       count: int,
                       onChange: Callable[[int], None]) -> None: ...
    """
    root.add_function(
        "createTrackbar",
        [
            FunctionNode.Arg("trackbarName", PrimitiveTypeNode.str_()),
            FunctionNode.Arg("windowName", PrimitiveTypeNode.str_()),
            FunctionNode.Arg("value", PrimitiveTypeNode.int_()),
            FunctionNode.Arg("count", PrimitiveTypeNode.int_()),
            FunctionNode.Arg("onChange",
                             CallableTypeNode("TrackbarCallback",
                                              PrimitiveTypeNode.int_("int"))),
        ]
    )
    """
    def createButton(buttonName: str,
                     onChange: Callable[[tuple[int] | tuple[int, Any]], None],
                     userData: Any | None = ...,
                     buttonType: int = ...,
                     initialButtonState: int = ...) -> None: ...
    """
    root.add_function(
        "createButton",
        [
            FunctionNode.Arg("buttonName", PrimitiveTypeNode.str_()),
            FunctionNode.Arg(
                "onChange",
                CallableTypeNode(
                    "ButtonCallback",
                    UnionTypeNode(
                        "onButtonChangeCallbackData",
                        [
                            TupleTypeNode("onButtonChangeCallbackData",
                                          [PrimitiveTypeNode.int_(), ]),
                            TupleTypeNode("onButtonChangeCallbackData",
                                          [PrimitiveTypeNode.int_(),
                                           AnyTypeNode("void*")])
                        ]
                    )
                )),
            FunctionNode.Arg("userData",
                             OptionalTypeNode(AnyTypeNode("void*")),
                             default_value="None"),
            FunctionNode.Arg("buttonType", PrimitiveTypeNode.int_(),
                             default_value="0"),
            FunctionNode.Arg("initialButtonState", PrimitiveTypeNode.int_(),
                             default_value="0")
        ]
    )
    """
    def setMouseCallback(
        windowName: str,
        onMouse: Callback[[int, int, int, int, Any | None], None],
        param: Any | None = ...
    ) -> None: ...
    """
    root.add_function(
        "setMouseCallback",
        [
            FunctionNode.Arg("windowName", PrimitiveTypeNode.str_()),
            FunctionNode.Arg(
                "onMouse",
                CallableTypeNode("MouseCallback", [
                    PrimitiveTypeNode.int_(),
                    PrimitiveTypeNode.int_(),
                    PrimitiveTypeNode.int_(),
                    PrimitiveTypeNode.int_(),
                    OptionalTypeNode(AnyTypeNode("void*"))
                ])
            ),
            FunctionNode.Arg("param", OptionalTypeNode(AnyTypeNode("void*")),
                             default_value="None")
        ]
    )


def refine_dnn_module(root: NamespaceNode) -> None:
    if "dnn" not in root.namespaces:
        return
    dnn_module = root.namespaces["dnn"]

    """
    class LayerProtocol(Protocol):
        def __init__(
            self, params: dict[str, DictValue],
            blobs: typing.Sequence[cv2.typing.MatLike]
        ) -> None: ...

        def getMemoryShapes(
            self, inputs: typing.Sequence[typing.Sequence[int]]
        ) -> typing.Sequence[typing.Sequence[int]]: ...

        def forward(
            self, inputs: typing.Sequence[cv2.typing.MatLike]
        ) -> typing.Sequence[cv2.typing.MatLike]: ...
    """
    layer_proto = ProtocolClassNode("LayerProtocol", dnn_module)
    layer_proto.add_function(
        "__init__",
        arguments=[
            FunctionNode.Arg(
                "params",
                DictTypeNode(
                    "LayerParams", PrimitiveTypeNode.str_(),
                    create_type_node("cv::dnn::DictValue")
                )
            ),
            FunctionNode.Arg("blobs", create_type_node("vector<cv::Mat>"))
        ]
    )
    layer_proto.add_function(
        "getMemoryShapes",
        arguments=[
            FunctionNode.Arg("inputs",
                             create_type_node("vector<vector<int>>"))
        ],
        return_type=FunctionNode.RetType(
            create_type_node("vector<vector<int>>")
        )
    )
    layer_proto.add_function(
        "forward",
        arguments=[
            FunctionNode.Arg("inputs", create_type_node("vector<cv::Mat>"))
        ],
        return_type=FunctionNode.RetType(create_type_node("vector<cv::Mat>"))
    )

    """
    def dnn_registerLayer(layerTypeName: str,
                          layerClass: typing.Type[LayerProtocol]) -> None: ...
    """
    root.add_function(
        "dnn_registerLayer",
        arguments=[
            FunctionNode.Arg("layerTypeName", PrimitiveTypeNode.str_()),
            FunctionNode.Arg(
                "layerClass",
                ClassTypeNode(ASTNodeTypeNode(
                    layer_proto.export_name, f"dnn.{layer_proto.export_name}"
                ))
            )
        ]
    )

    """
    def dnn_unregisterLayer(layerTypeName: str) -> None: ...
    """
    root.add_function(
        "dnn_unregisterLayer",
        arguments=[
            FunctionNode.Arg("layerTypeName", PrimitiveTypeNode.str_())
        ]
    )


def _trim_class_name_from_argument_types(
    overloads: Iterable[FunctionNode.Overload],
    class_name: str
) -> None:
    separator = f"{class_name}_"
    for overload in overloads:
        for arg in [arg for arg in overload.arguments
                    if arg.type_node is not None]:
            ast_node = cast(ASTNodeTypeNode, arg.type_node)
            if class_name in ast_node.ctype_name:
                fixed_name = ast_node._typename.split(separator)[-1]
                ast_node._typename = fixed_name


def _find_argument_index(arguments: Sequence[FunctionNode.Arg],
                         name: str) -> Optional[int]:
    for i, arg in enumerate(arguments):
        if arg.name == name:
            return i
    return None


NODES_TO_REFINE = {
    SymbolName(("cv", ), (), "resize"): make_optional_arg("dsize"),
    SymbolName(("cv", ), (), "calcHist"): make_optional_arg("mask"),
    SymbolName(("cv", ), (), "floodFill"): make_optional_arg("mask"),
    SymbolName(("cv", ), ("Feature2D", ), "detectAndCompute"): make_optional_arg("mask"),
    SymbolName(("cv", ), (), "findEssentialMat"): make_optional_arg(
        "distCoeffs1", "distCoeffs2", "dist_coeff1", "dist_coeff2"
    ),
    SymbolName(("cv", ), (), "drawFrameAxes"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "getOptimalNewCameraMatrix"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "initInverseRectificationMap"): make_optional_arg("distCoeffs", "R"),
    SymbolName(("cv", ), (), "initUndistortRectifyMap"): make_optional_arg("distCoeffs", "R"),
    SymbolName(("cv", ), (), "projectPoints"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solveP3P"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solvePnP"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solvePnPGeneric"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solvePnPRansac"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solvePnPRefineLM"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "solvePnPRefineVVS"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "undistort"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", ), (), "undistortPoints"): make_optional_arg("distCoeffs"),
    SymbolName(("cv", "fisheye"), (), "initUndistortRectifyMap"): make_optional_arg("D"),
    SymbolName(("cv", ), (), "imread"): make_optional_none_return,
    SymbolName(("cv", ), (), "imdecode"): make_optional_none_return,
}

ERROR_CLASS_PROPERTIES = (
    ClassProperty("code", PrimitiveTypeNode.int_(), False),
    ClassProperty("err", PrimitiveTypeNode.str_(), False),
    ClassProperty("file", PrimitiveTypeNode.str_(), False),
    ClassProperty("func", PrimitiveTypeNode.str_(), False),
    ClassProperty("line", PrimitiveTypeNode.int_(), False),
    ClassProperty("msg", PrimitiveTypeNode.str_(), False),
)
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

### Classes and Structures

- **builtin_exception**: A class/struct defined in this file
- **LayerProtocol**: A class/struct defined in this file

### Functions and Methods

- **setMouseCallback()**: A function/method defined in this file
- **forward()**: A function/method defined in this file
- **dnn_unregisterLayer()**: A function/method defined in this file
- **fix_namespace_usage_scope()**: A function/method defined in this file
- **root()**: A function/method defined in this file
- **createTrackbar()**: A function/method defined in this file
- **_find_argument_index()**: A function/method defined in this file
- **createButton()**: A function/method defined in this file
- **fix_scalar_return_type()**: A function/method defined in this file
- **convert_returned_scalar_to_tuple()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file
- **export_matrix_type_constants()**: A function/method defined in this file
- **make_optional_none_return()**: A function/method defined in this file
- **refine_cuda_module()**: A function/method defined in this file
- **fix_type_usage()**: A function/method defined in this file
- **_make_optional_arg()**: A function/method defined in this file
- **_trim_class_name_from_argument_types()**: A function/method defined in this file
- **refine_dnn_module()**: A function/method defined in this file
- **make_optional_arg()**: A function/method defined in this file
- **apply_manual_api_refinement()**: A function/method defined in this file
- **getMemoryShapes()**: A function/method defined in this file
- **dnn_registerLayer()**: A function/method defined in this file
- **redirectError()**: A function/method defined in this file
- **fix_cudaoptflow_enums_names()**: A function/method defined in this file
- **refine_highgui_module()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `.nodes`
- `.types_conversion`
- `typing`
- `cast`
- `.ast_utils`
- `create_type_node`


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


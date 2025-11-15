# Documentation for `modules/python/src2/typing_stubs_generation/types_conversion.py`

## File Metadata

- **Full Path**: `modules/python/src2/typing_stubs_generation/types_conversion.py`
- **File Name**: `types_conversion.py`
- **File Size**: 12,454 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/src2/typing_stubs_generation/types_conversion.py](../../../../modules/python/src2/typing_stubs_generation/types_conversion.py)

## Purpose and Role

This file is located in the `modules/python/src2/typing_stubs_generation` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
from typing import Tuple, List, Optional

from .predefined_types import PREDEFINED_TYPES
from .nodes.type_node import (
    TypeNode, UnionTypeNode, SequenceTypeNode, ASTNodeTypeNode, TupleTypeNode
)


def replace_template_parameters_with_placeholders(string: str) \
        -> Tuple[str, Tuple[str, ...]]:
    """Replaces template parameters with `format` placeholders for all template
    instantiations in provided string.
    Only outermost template parameters are replaced.

    Args:
        string (str): input string containing C++ template instantiations

    Returns:
        tuple[str, tuple[str, ...]]: string with '{}' placeholders  template
            instead of instantiation types and a tuple of extracted types.

    >>> template_string, args = replace_template_parameters_with_placeholders(
    ...     "std::vector<cv::Point<int>>, test<int>"
    ... )
    >>> template_string.format(*args) == "std::vector<cv::Point<int>>, test<int>"
    True

    >>> replace_template_parameters_with_placeholders(
    ...     "cv::util::variant<cv::GRunArgs, cv::GOptRunArgs>"
    ... )
    ('cv::util::variant<{}>', ('cv::GRunArgs, cv::GOptRunArgs',))

    >>> replace_template_parameters_with_placeholders("vector<Point<int>>")
    ('vector<{}>', ('Point<int>',))

    >>> replace_template_parameters_with_placeholders(
    ...     "vector<Point<int>>, vector<float>"
    ... )
    ('vector<{}>, vector<{}>', ('Point<int>', 'float'))

    >>> replace_template_parameters_with_placeholders("string without templates")
    ('string without templates', ())
    """

    template_brackets_indices = []
    template_instantiations_count = 0
    template_start_index = 0
    for i, c in enumerate(string):
        if c == "<":
            template_instantiations_count += 1
            if template_instantiations_count == 1:
                # + 1 - because left bound is included in substring range
                template_start_index = i + 1
        elif c == ">":
            template_instantiations_count -= 1
            assert template_instantiations_count >= 0, \
                "Provided string is ill-formed. There are more '>' than '<'."
            if template_instantiations_count == 0:
                template_brackets_indices.append((template_start_index, i))
    assert template_instantiations_count == 0, \
        "Provided string is ill-formed. There are more '<' than '>'."
    template_args: List[str] = []
    # Reversed loop is required to preserve template start/end indices
    for i, j in reversed(template_brackets_indices):
        template_args.insert(0, string[i:j])
        string = string[:i] + "{}" + string[j:]
    return string, tuple(template_args)


def get_template_instantiation_type(typename: str) -> str:
    """Extracts outermost template instantiation type from provided string

    Args:
        typename (str): String containing C++ template instantiation.

    Returns:
        str: String containing template instantiation type

    >>> get_template_instantiation_type("std::vector<cv::Point<int>>")
    'cv::Point<int>'
    >>> get_template_instantiation_type("std::vector<uchar>")
    'uchar'
    >>> get_template_instantiation_type("std::map<int, float>")
    'int, float'
    >>> get_template_instantiation_type("uchar")
    Traceback (most recent call last):
    ...
    ValueError: typename ('uchar') doesn't contain template instantiations
    >>> get_template_instantiation_type("std::vector<int>, std::vector<float>")
    Traceback (most recent call last):
    ...
    ValueError: typename ('std::vector<int>, std::vector<float>') contains more than 1 template instantiation
    """

    _, args = replace_template_parameters_with_placeholders(typename)
    if len(args) == 0:
        raise ValueError(
            "typename ('{}') doesn't contain template instantiations".format(typename)
        )
    if len(args) > 1:
        raise ValueError(
            "typename ('{}') contains more than 1 template instantiation".format(typename)
        )
    return args[0]


def normalize_ctype_name(typename: str) -> str:
    """Normalizes C++ name by removing unnecessary namespace prefixes and possible
    pointer/reference qualification. '::' are replaced with '_'.

    NOTE: Pointer decay for 'void*' is not performed.

    Args:
        typename (str): Name of the C++ type for normalization

    Returns:
        str: Normalized C++ type name.

    >>> normalize_ctype_name('std::vector<cv::Point2f>&')
    'vector<cv_Point2f>'
    >>> normalize_ctype_name('AKAZE::DescriptorType')
    'AKAZE_DescriptorType'
    >>> normalize_ctype_name('std::vector<Mat>')
    'vector<Mat>'
    >>> normalize_ctype_name('std::string')
    'string'
    >>> normalize_ctype_name('void*')  # keep void* as is - special case
    'void*'
    >>> normalize_ctype_name('Ptr<AKAZE>')
    'AKAZE'
    >>> normalize_ctype_name('Algorithm_Ptr')
    'Algorithm'
    """
    for prefix_to_remove in ("cv", "std"):
        if typename.startswith(prefix_to_remove):
            typename = typename[len(prefix_to_remove):]
    typename = typename.replace("::", "_").lstrip("_")
    if typename.endswith('&'):
        typename = typename[:-1]
    typename = typename.strip()

    if typename == 'void*':
        return typename

    if is_pointer_type(typename):
        # Case for "type*", "type_Ptr", "typePtr"
        for suffix in ("*", "_Ptr", "Ptr"):
            if typename.endswith(suffix):
                return typename[:-len(suffix)]
        # Case Ptr<Type>
        if _is_template_instantiation(typename):
            return normalize_ctype_name(
                get_template_instantiation_type(typename)
            )
        # Case Ptr_Type
        return typename.split("_", maxsplit=1)[-1]

    # special normalization for several G-API Types
    if typename.startswith("GArray_") or typename.startswith("GArray<"):
        return "GArrayT"
    if typename.startswith("GOpaque_") or typename.startswith("GOpaque<"):
        return "GOpaqueT"
    if typename == "GStreamerPipeline" or typename.startswith("GStreamerSource"):
        return "gst_" + typename

    return typename


def is_tuple_type(typename: str) -> bool:
    return typename.startswith("tuple") or typename.startswith("pair")


def is_sequence_type(typename: str) -> bool:
    return typename.startswith("vector")


def is_pointer_type(typename: str) -> bool:
    return typename.endswith("Ptr") or typename.endswith("*") \
        or typename.startswith("Ptr")


def is_union_type(typename: str) -> bool:
    return typename.startswith('util_variant')


def _is_template_instantiation(typename: str) -> bool:
    """Fast, but unreliable check whenever provided typename is a template
    instantiation.

    Args:
        typename (str): typename to check against template instantiation.

    Returns:
        bool: True if provided `typename` contains template instantiation,
            False otherwise
    """

    if "<" in typename:
        assert ">" in typename, \
            "Wrong template class instantiation: {}. '>' is missing".format(typename)
        return True
    return False


def create_type_nodes_from_template_arguments(template_args_str: str) \
        -> List[TypeNode]:
    """Creates a list of type nodes corresponding to the argument types
    used for template instantiation.
    This method correctly addresses the situation when arguments of the input
    template are also templates.
    Example:
    if `create_type_node` is called with
    `std::tuple<std::variant<int, Point2i>, int, std::vector<int>>`
    this function will be called with
    `std::variant<int, Point<int>>, int, std::vector<int>`
    that produces the following order of types resolution
                                    `std::variant` ~ `Union`
    `std::variant<int, Point2i>` -> `int`          ~ `int` -> `Union[int, Point2i]`
                                    `Point2i`      ~ `Point2i`
    `int` -> `int`
    `std::vector<int>` -> `std::vector` ~ `Sequence` -> `Sequence[int]`
                                  `int` ~ `int`

    Returns:
        List[TypeNode]: set of type nodes used for template instantiation.
        List is empty if input string doesn't contain template instantiation.
    """

    type_nodes = []
    template_args_str, templated_args_types = replace_template_parameters_with_placeholders(
        template_args_str
    )
    template_index = 0
    # For each template argument
    for template_arg in template_args_str.split(","):
        template_arg = template_arg.strip()
        # Check if argument requires type substitution
        if _is_template_instantiation(template_arg):
            # Reconstruct the original type
            template_arg = template_arg.format(templated_args_types[template_index])
            template_index += 1
        # create corresponding type node
        type_nodes.append(create_type_node(template_arg))
    return type_nodes


def create_type_node(typename: str,
                     original_ctype_name: Optional[str] = None) -> TypeNode:
    """Converts C++ type name to appropriate type used in Python library API.

    Conversion procedure:
        1. Normalize typename: remove redundant prefixes, unify name
           components delimiters, remove reference qualifications.
        2. Check whenever typename has a known predefined conversion or exported
           as alias e.g.
            - C++ `double` -> Python `float`
            - C++ `cv::Rect` -> Python `Sequence[int]`
            - C++ `std::vector<char>` -> Python `np.ndarray`
           return TypeNode corresponding to the appropriate type.
        3. Check whenever typename is a container of types e.g. variant,
           sequence or tuple. If so, select appropriate Python container type
           and perform arguments conversion.
        4. Create a type node corresponding to the AST node passing normalized
           typename as its name.

    Args:
        typename (str): C++ type name to convert.
        original_ctype_name (Optional[str]): Original C++ name of the type.
            `original_ctype_name` == `typename` if provided argument is None.
            Default is None.

    Returns:
        TypeNode: type node that wraps C++ type exposed to Python

    >>> create_type_node('Ptr<AKAZE>').typename
    'AKAZE'
    >>> create_type_node('std::vector<Ptr<cv::Algorithm>>').typename
    'typing.Sequence[Algorithm]'
    """

    if original_ctype_name is None:
        original_ctype_name = typename

    typename = normalize_ctype_name(typename.strip())

    # if typename is a known alias or has explicitly defined substitution
    type_node = PREDEFINED_TYPES.get(typename)
    if type_node is not None:
        type_node.ctype_name = original_ctype_name
        return type_node

    # If typename is a known exported alias name (e.g. IndexParams or SearchParams)
    for alias in PREDEFINED_TYPES.values():
        if alias.typename == typename:
            return alias

    if is_union_type(typename):
        union_types = get_template_instantiation_type(typename)
        return UnionTypeNode(
            original_ctype_name,
            items=create_type_nodes_from_template_arguments(union_types)
        )

    # if typename refers to a sequence type e.g. vector<int>
    if is_sequence_type(typename):
        # Recursively convert sequence element type
        if _is_template_instantiation(typename):
            inner_sequence_type = create_type_node(
                get_template_instantiation_type(typename)
            )
        else:
            # Handle vector_Type cases
            # maxsplit=1 is required to handle sequence of sequence e.g:
            # vector_vector_Mat -> Sequence[Sequence[Mat]]
            inner_sequence_type = create_type_node(typename.split("_", 1)[-1])
        return SequenceTypeNode(original_ctype_name, inner_sequence_type)

    # If typename refers to a heterogeneous container
    # (can contain elements of different types)
    if is_tuple_type(typename):
        tuple_types = get_template_instantiation_type(typename)
        return TupleTypeNode(
            original_ctype_name,
            items=create_type_nodes_from_template_arguments(tuple_types)
        )
    # If everything else is False, it means that input typename refers to a
    # class or enum of the library.
    return ASTNodeTypeNode(original_ctype_name, typename)


if __name__ == "__main__":
    import doctest
    doctest.testmod()
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

- **the**: A class/struct defined in this file
- **or**: A class/struct defined in this file
- **instantiation**: A class/struct defined in this file

### Functions and Methods

- **replace_template_parameters_with_placeholders()**: A function/method defined in this file
- **is_pointer_type()**: A function/method defined in this file
- **normalize_ctype_name()**: A function/method defined in this file
- **_is_template_instantiation()**: A function/method defined in this file
- **get_template_instantiation_type()**: A function/method defined in this file
- **will()**: A function/method defined in this file
- **is_union_type()**: A function/method defined in this file
- **create_type_nodes_from_template_arguments()**: A function/method defined in this file
- **is_sequence_type()**: A function/method defined in this file
- **is_tuple_type()**: A function/method defined in this file
- **create_type_node()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `doctest`
- `typing`
- `provided`
- `.nodes.type_node`
- `PREDEFINED_TYPES`
- `Tuple`
- `.predefined_types`


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


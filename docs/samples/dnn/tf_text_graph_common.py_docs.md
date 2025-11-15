# Documentation for `samples/dnn/tf_text_graph_common.py`

## File Metadata

- **Full Path**: `samples/dnn/tf_text_graph_common.py`
- **File Name**: `tf_text_graph_common.py`
- **File Size**: 10,055 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/tf_text_graph_common.py](../../samples/dnn/tf_text_graph_common.py)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
def tokenize(s):
    tokens = []
    token = ""
    isString = False
    isComment = False
    for symbol in s:
        isComment = (isComment and symbol != '\n') or (not isString and symbol == '#')
        if isComment:
            continue

        if symbol == ' ' or symbol == '\t' or symbol == '\r' or symbol == '\'' or \
           symbol == '\n' or symbol == ':' or symbol == '\"' or symbol == ';' or \
           symbol == ',':

            if (symbol == '\"' or symbol == '\'') and isString:
                tokens.append(token)
                token = ""
            else:
                if isString:
                    token += symbol
                elif token:
                    tokens.append(token)
                    token = ""
            isString = (symbol == '\"' or symbol == '\'') ^ isString

        elif symbol == '{' or symbol == '}' or symbol == '[' or symbol == ']':
            if token:
                tokens.append(token)
                token = ""
            tokens.append(symbol)
        else:
            token += symbol
    if token:
        tokens.append(token)
    return tokens


def parseMessage(tokens, idx):
    msg = {}
    assert(tokens[idx] == '{')

    isArray = False
    while True:
        if not isArray:
            idx += 1
            if idx < len(tokens):
                fieldName = tokens[idx]
            else:
                return None
            if fieldName == '}':
                break

        idx += 1
        fieldValue = tokens[idx]

        if fieldValue == '{':
            embeddedMsg, idx = parseMessage(tokens, idx)
            if fieldName in msg:
                msg[fieldName].append(embeddedMsg)
            else:
                msg[fieldName] = [embeddedMsg]
        elif fieldValue == '[':
            isArray = True
        elif fieldValue == ']':
            isArray = False
        else:
            if fieldName in msg:
                msg[fieldName].append(fieldValue)
            else:
                msg[fieldName] = [fieldValue]
    return msg, idx


def readTextMessage(filePath):
    if not filePath:
        return {}
    with open(filePath, 'rt') as f:
        content = f.read()

    tokens = tokenize('{' + content + '}')
    msg = parseMessage(tokens, 0)
    return msg[0] if msg else {}


def listToTensor(values):
    if all([isinstance(v, float) for v in values]):
        dtype = 'DT_FLOAT'
        field = 'float_val'
    elif all([isinstance(v, int) for v in values]):
        dtype = 'DT_INT32'
        field = 'int_val'
    else:
        raise Exception('Wrong values types')

    msg = {
        'tensor': {
            'dtype': dtype,
            'tensor_shape': {
                'dim': {
                    'size': len(values)
                }
            }
        }
    }
    msg['tensor'][field] = values
    return msg


def addConstNode(name, values, graph_def):
    node = NodeDef()
    node.name = name
    node.op = 'Const'
    node.addAttr('value', values)
    graph_def.node.extend([node])


def addSlice(inp, out, begins, sizes, graph_def):
    beginsNode = NodeDef()
    beginsNode.name = out + '/begins'
    beginsNode.op = 'Const'
    beginsNode.addAttr('value', begins)
    graph_def.node.extend([beginsNode])

    sizesNode = NodeDef()
    sizesNode.name = out + '/sizes'
    sizesNode.op = 'Const'
    sizesNode.addAttr('value', sizes)
    graph_def.node.extend([sizesNode])

    sliced = NodeDef()
    sliced.name = out
    sliced.op = 'Slice'
    sliced.input.append(inp)
    sliced.input.append(beginsNode.name)
    sliced.input.append(sizesNode.name)
    graph_def.node.extend([sliced])


def addReshape(inp, out, shape, graph_def):
    shapeNode = NodeDef()
    shapeNode.name = out + '/shape'
    shapeNode.op = 'Const'
    shapeNode.addAttr('value', shape)
    graph_def.node.extend([shapeNode])

    reshape = NodeDef()
    reshape.name = out
    reshape.op = 'Reshape'
    reshape.input.append(inp)
    reshape.input.append(shapeNode.name)
    graph_def.node.extend([reshape])


def addSoftMax(inp, out, graph_def):
    softmax = NodeDef()
    softmax.name = out
    softmax.op = 'Softmax'
    softmax.addAttr('axis', -1)
    softmax.input.append(inp)
    graph_def.node.extend([softmax])


def addFlatten(inp, out, graph_def):
    flatten = NodeDef()
    flatten.name = out
    flatten.op = 'Flatten'
    flatten.input.append(inp)
    graph_def.node.extend([flatten])


class NodeDef:
    def __init__(self):
        self.input = []
        self.name = ""
        self.op = ""
        self.attr = {}

    def addAttr(self, key, value):
        assert(not key in self.attr)
        if isinstance(value, bool):
            self.attr[key] = {'b': value}
        elif isinstance(value, int):
            self.attr[key] = {'i': value}
        elif isinstance(value, float):
            self.attr[key] = {'f': value}
        elif isinstance(value, str):
            self.attr[key] = {'s': value}
        elif isinstance(value, list):
            self.attr[key] = listToTensor(value)
        else:
            raise Exception('Unknown type of attribute ' + key)

    def Clear(self):
        self.input = []
        self.name = ""
        self.op = ""
        self.attr = {}


class GraphDef:
    def __init__(self):
        self.node = []

    def save(self, filePath):
        with open(filePath, 'wt') as f:

            def printAttr(d, indent):
                indent = ' ' * indent
                for key, value in sorted(d.items(), key=lambda x:x[0].lower()):
                    value = value if isinstance(value, list) else [value]
                    for v in value:
                        if isinstance(v, dict):
                            f.write(indent + key + ' {\n')
                            printAttr(v, len(indent) + 2)
                            f.write(indent + '}\n')
                        else:
                            isString = False
                            if isinstance(v, str) and not v.startswith('DT_'):
                                try:
                                    float(v)
                                except:
                                    isString = True

                            if isinstance(v, bool):
                                printed = 'true' if v else 'false'
                            elif v == 'true' or v == 'false':
                                printed = 'true' if v == 'true' else 'false'
                            elif isString:
                                printed = '\"%s\"' % v
                            else:
                                printed = str(v)
                            f.write(indent + key + ': ' + printed + '\n')

            for node in self.node:
                f.write('node {\n')
                f.write('  name: \"%s\"\n' % node.name)
                f.write('  op: \"%s\"\n' % node.op)
                for inp in node.input:
                    f.write('  input: \"%s\"\n' % inp)
                for key, value in sorted(node.attr.items(), key=lambda x:x[0].lower()):
                    f.write('  attr {\n')
                    f.write('    key: \"%s\"\n' % key)
                    f.write('    value {\n')
                    printAttr(value, 6)
                    f.write('    }\n')
                    f.write('  }\n')
                f.write('}\n')


def parseTextGraph(filePath):
    msg = readTextMessage(filePath)

    graph = GraphDef()
    for node in msg['node']:
        graphNode = NodeDef()
        graphNode.name = node['name'][0]
        graphNode.op = node['op'][0]
        graphNode.input = node['input'] if 'input' in node else []

        if 'attr' in node:
            for attr in node['attr']:
                graphNode.attr[attr['key'][0]] = attr['value'][0]

        graph.node.append(graphNode)
    return graph


# Removes Identity nodes
def removeIdentity(graph_def):
    identities = {}
    for node in graph_def.node:
        if node.op == 'Identity' or node.op == 'IdentityN':
            inp = node.input[0]
            if inp in identities:
                identities[node.name] = identities[inp]
            else:
                identities[node.name] = inp
            graph_def.node.remove(node)

    for node in graph_def.node:
        for i in range(len(node.input)):
            if node.input[i] in identities:
                node.input[i] = identities[node.input[i]]


def removeUnusedNodesAndAttrs(to_remove, graph_def):
    unusedAttrs = ['T', 'Tshape', 'N', 'Tidx', 'Tdim', 'use_cudnn_on_gpu',
                   'Index', 'Tperm', 'is_training', 'Tpaddings']

    removedNodes = []

    for i in reversed(range(len(graph_def.node))):
        op = graph_def.node[i].op
        name = graph_def.node[i].name

        if to_remove(name, op):
            if op != 'Const':
                removedNodes.append(name)

            del graph_def.node[i]
        else:
            for attr in unusedAttrs:
                if attr in graph_def.node[i].attr:
                    del graph_def.node[i].attr[attr]

    # Remove references to removed nodes except Const nodes.
    for node in graph_def.node:
        for i in reversed(range(len(node.input))):
            if node.input[i] in removedNodes:
                del node.input[i]


def writeTextGraph(modelPath, outputPath, outNodes):
    try:
        import cv2 as cv

        cv.dnn.writeTextGraph(modelPath, outputPath)
    except:
        import tensorflow as tf
        from tensorflow.tools.graph_transforms import TransformGraph

        with tf.gfile.FastGFile(modelPath, 'rb') as f:
            graph_def = tf.GraphDef()
            graph_def.ParseFromString(f.read())

            graph_def = TransformGraph(graph_def, ['image_tensor'], outNodes, ['sort_by_execution_order'])

            for node in graph_def.node:
                if node.op == 'Const':
                    if 'value' in node.attr and node.attr['value'].tensor.tensor_content:
                        node.attr['value'].tensor.tensor_content = b''

        tf.train.write_graph(graph_def, "", outputPath, as_text=True)
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

- **GraphDef**: A class/struct defined in this file
- **NodeDef**: A class/struct defined in this file

### Functions and Methods

- **addReshape()**: A function/method defined in this file
- **printAttr()**: A function/method defined in this file
- **readTextMessage()**: A function/method defined in this file
- **addSoftMax()**: A function/method defined in this file
- **parseMessage()**: A function/method defined in this file
- **writeTextGraph()**: A function/method defined in this file
- **parseTextGraph()**: A function/method defined in this file
- **addAttr()**: A function/method defined in this file
- **removeUnusedNodesAndAttrs()**: A function/method defined in this file
- **save()**: A function/method defined in this file
- **Clear()**: A function/method defined in this file
- **listToTensor()**: A function/method defined in this file
- **addFlatten()**: A function/method defined in this file
- **addSlice()**: A function/method defined in this file
- **tokenize()**: A function/method defined in this file
- **removeIdentity()**: A function/method defined in this file
- **addConstNode()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tensorflow`
- `tensorflow.tools.graph_transforms`
- `cv2`
- `TransformGraph`


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


# Documentation for `docs/samples/python/demo.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/demo.py_docs.md`
- **File Name**: `demo.py_docs.md`
- **File Size**: 10,118 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/demo.py_docs.md](../../../docs/samples/python/demo.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/demo.py`

## File Metadata

- **Full Path**: `samples/python/demo.py`
- **File Name**: `demo.py`
- **File Size**: 6,062 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/demo.py](../../samples/python/demo.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Sample-launcher application.
'''

# Python 2/3 compatibility
from __future__ import print_function
import sys

# local modules
from common import splitfn

# built-in modules
import webbrowser
from glob import glob
from subprocess import Popen

try:
    import tkinter as tk  # Python 3
    from tkinter.scrolledtext import ScrolledText
except ImportError:
    import Tkinter as tk  # Python 2
    from ScrolledText import ScrolledText


#from IPython.Shell import IPShellEmbed
#ipshell = IPShellEmbed()

exclude_list = ['demo', 'common']

class LinkManager:
    def __init__(self, text, url_callback = None):
        self.text = text
        self.text.tag_config("link", foreground="blue", underline=1)
        self.text.tag_bind("link", "<Enter>", self._enter)
        self.text.tag_bind("link", "<Leave>", self._leave)
        self.text.tag_bind("link", "<Button-1>", self._click)

        self.url_callback = url_callback
        self.reset()

    def reset(self):
        self.links = {}
    def add(self, action):
        # add an action to the manager.  returns tags to use in
        # associated text widget
        tag = "link-%d" % len(self.links)
        self.links[tag] = action
        return "link", tag

    def _enter(self, event):
        self.text.config(cursor="hand2")
    def _leave(self, event):
        self.text.config(cursor="")
    def _click(self, event):
        for tag in self.text.tag_names(tk.CURRENT):
            if tag.startswith("link-"):
                proc = self.links[tag]
                if callable(proc):
                    proc()
                else:
                    if self.url_callback:
                        self.url_callback(proc)

class App:
    def __init__(self):
        root = tk.Tk()
        root.title('OpenCV Demo')

        self.win = win = tk.PanedWindow(root, orient=tk.HORIZONTAL, sashrelief=tk.RAISED, sashwidth=4)
        self.win.pack(fill=tk.BOTH, expand=1)

        left = tk.Frame(win)
        right = tk.Frame(win)
        win.add(left)
        win.add(right)

        scrollbar = tk.Scrollbar(left, orient=tk.VERTICAL)
        self.demos_lb = demos_lb = tk.Listbox(left, yscrollcommand=scrollbar.set)
        scrollbar.config(command=demos_lb.yview)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        demos_lb.pack(side=tk.LEFT, fill=tk.BOTH, expand=1)

        self.samples = {}
        for fn in glob('*.py'):
            name = splitfn(fn)[1]
            if fn[0] != '_' and name not in exclude_list:
                self.samples[name] = fn

        for name in sorted(self.samples):
            demos_lb.insert(tk.END, name)

        demos_lb.bind('<<ListboxSelect>>', self.on_demo_select)

        self.cmd_entry = cmd_entry = tk.Entry(right)
        cmd_entry.bind('<Return>', self.on_run)
        run_btn = tk.Button(right, command=self.on_run, text='Run', width=8)

        self.text = text = ScrolledText(right, font=('arial', 12, 'normal'), width = 30, wrap='word')
        self.linker = _linker = LinkManager(text, self.on_link)
        self.text.tag_config("header1", font=('arial', 14, 'bold'))
        self.text.tag_config("header2", font=('arial', 12, 'bold'))
        text.config(state='disabled')

        text.pack(fill='both', expand=1, side=tk.BOTTOM)
        cmd_entry.pack(fill='x', side='left' , expand=1)
        run_btn.pack()

    def on_link(self, url):
        print(url)
        webbrowser.open(url)

    def on_demo_select(self, evt):
        name = self.demos_lb.get( self.demos_lb.curselection()[0] )
        fn = self.samples[name]

        descr = ""
        try:
            if sys.version_info[0] > 2:
                # Python 3.x
                module_globals = {}
                module_locals = {}
                with open(fn, 'r') as f:
                    module_code = f.read()
                exec(compile(module_code, fn, 'exec'), module_globals, module_locals)
                descr = module_locals.get('__doc__', 'no-description')
            else:
                # Python 2
                module_globals = {}
                execfile(fn, module_globals)  # noqa: F821
                descr = module_globals.get('__doc__', 'no-description')
        except Exception as e:
            descr = str(e)

        self.linker.reset()
        self.text.config(state='normal')
        self.text.delete(1.0, tk.END)
        self.format_text(descr)
        self.text.config(state='disabled')

        self.cmd_entry.delete(0, tk.END)
        self.cmd_entry.insert(0, fn)

    def format_text(self, s):
        text = self.text
        lines = s.splitlines()
        for i, s in enumerate(lines):
            s = s.rstrip()
            if i == 0 and not s:
                continue
            if s and s == '='*len(s):
                text.tag_add('header1', 'end-2l', 'end-1l')
            elif s and s == '-'*len(s):
                text.tag_add('header2', 'end-2l', 'end-1l')
            else:
                text.insert('end', s+'\n')

        def add_link(start, end, url):
            for tag in self.linker.add(url):
                text.tag_add(tag, start, end)
        self.match_text(r'http://\S+', add_link)

    def match_text(self, pattern, tag_proc, regexp=True):
        text = self.text
        text.mark_set('matchPos', '1.0')
        count = tk.IntVar()
        while True:
            match_index = text.search(pattern, 'matchPos', count=count, regexp=regexp, stopindex='end')
            if not match_index:
                break
            end_index = text.index( "%s+%sc" % (match_index, count.get()) )
            text.mark_set('matchPos', end_index)
            if callable(tag_proc):
                tag_proc(match_index, end_index, text.get(match_index, end_index))
            else:
                text.tag_add(tag_proc, match_index, end_index)

    def on_run(self, *args):
        cmd = self.cmd_entry.get()
        print('running:', cmd)
        Popen(sys.executable + ' ' + cmd, shell=True)

    def run(self):
        tk.mainloop()


if __name__ == '__main__':
    App().run()
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

- **LinkManager**: A class/struct defined in this file
- **App**: A class/struct defined in this file

### Functions and Methods

- **add()**: A function/method defined in this file
- **on_link()**: A function/method defined in this file
- **in()**: A function/method defined in this file
- **_click()**: A function/method defined in this file
- **for()**: A function/method defined in this file
- **match_text()**: A function/method defined in this file
- **_enter()**: A function/method defined in this file
- **reset()**: A function/method defined in this file
- **on_demo_select()**: A function/method defined in this file
- **on_run()**: A function/method defined in this file
- **format_text()**: A function/method defined in this file
- **add_link()**: A function/method defined in this file
- **run()**: A function/method defined in this file
- **_leave()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `glob`
- `sys`
- `webbrowser`
- `IPShellEmbed`
- `IPython.Shell`
- `tkinter.scrolledtext`
- `splitfn`
- `Tkinter`
- `print_function`
- `ScrolledText`
- `common`
- `tkinter`
- `Popen`
- `__future__`
- `subprocess`


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


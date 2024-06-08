---
layout: post
title:  "Week 1: Option Block Modularization and Workflow Separation"
author: zaky
categories: [ gsoc ]
image: assets/images/DFD-workflow-separation.png
---

In the kickoff meeting, Håkon and Sebastian advised me to make some adjustments to how I approach the project. Therefore, the option block modularization and workflow separation will be done first, before the grc separation. The main focus in the first half was to modularize block options and separate workflows. The option block will be converted into Python so we can easily modify its behavior. This will enable the option block to read information about workflows from multiple workflow.yml files instead of having the workflow information hardcoded.

## Option Block Modularization
The first thing we need to do is to make multiple YAML file to hold information about each workflows. The structure of each YAML file will be
```yaml
id (str): unique name of workflow
descripton (str): detailed information of workflow
output_language (str): target language
output_language_label (str): Information for users about the target language of the workflow
generator_class (str): Name of the code generator class
generator_module (str): Module name of where the code generator class is located
generator_options (str): Used to select a workflow
generator_options_label (str): Information for users to select a workflow
```

`generator_module` parameter will be using the actual python module format, for example: `gnuradio.grc.workflows.python_qt_gui`. There are 9 workflow that will be defined for this project:
* cpp_hb_nogui
* cpp_hb_qt_gui
* cpp_nogui
* cpp_qt_gui
* python_bokeh_gui
* python_hb_nogui
* python_hb_qt_gui
* python_nogui
* python_qt_gui

After making definition of each workflows, the next step will be implementing option block from YAML into python. Actually, GNU Radio already have some blocks that is converted/implemented into python, ex: `embedded_python.py` and `virtual.py`.

To make python version of option block, first we need to define the initial parameters, we use information from `options.block.yml` to define initial parameters of option block, then we will modify some of these parameters in `__init__` method if necessary.

When GRC is starting, it will call `Platform.build_library` to load all necessary information, such as blocks YAML file. So I modify this method to also load the workflow files, this method will create `Workflow` object and call `Workflow.load_workflow` to load each workflow files into the workflow object. With this, grc will store all information about the workflows, to access all information about each workflows, we can just use `workflow_manager.workflows` attribute of the platform object, it will return array of workflow objects, with workflow objects store information of each workflows.

The workflow object defined by:
```python
class Workflow:
    def __init__(self, *args, **kwargs):
        if len(args) == 1:
            with open(args[0], 'r') as wf:
                self.workflow_params = yaml.safe_load(wf)
        else:
            self.workflow_params = kwargs

        self.id = self.workflow_params.pop('id')
        self.descripion = self.workflow_params.pop('description')
        self.output_language = self.workflow_params.pop('output_language')
        self.output_language_label = self.workflow_params.pop('output_language_label')
        self.generator_class = self.workflow_params.pop('generator_class')
        self.generator_module = self.workflow_params.pop('generator_module')
        self.generator_options = self.workflow_params.pop('generator_options')
        self.generator_options_label = self.workflow_params.pop('generator_options_label')
        self.context = self.workflow_params # additional workflow parameters
```

When the GRC is initiating a flow graph, it will call `Platform.make_flow_graph` to make a flowgraph from a grc file (if the grc file is not provided, it will generate default flowgraph with only option block and variable), the `make_flow_graph` method will call `Platform.parse_flow_graph` to parse flow graph from grc file into a dictionary called `data`, then it will call `FlowGraph.import_data` with `data` as the parameter. With this information, I will make a method `Options.insert_grc_parameters` to insert information about option block parameters, this method will be called in `FlowGraph.import_data`.

With this, all information about workflows already being consumed by option block. When user changing output language or generate_options, `rewrite` method will be called, the `rewrite` method in option block will change available options in `generate_options` parameter based on the current value of `output_language`. 

![python output language](/assets/images/python_output_language.png)

For this case, if the output value is `Python`, the generate options will be:
![[python] generate options](/assets/images/python_generate_options.png)

![cpp output language](/assets/images/cpp_output_language.png)

If output value is `C++`, the generate options will be:

![cpp generate options](/assets/images/cpp_generate_options.png)

With this, options block modularization is finished :D

## Workflow Separation
The workflow separation will enable user to define their own workflow with making their own code generation class, template, and workflow YAML file. Each workflows will be stored inside `grc/workflows` folder

The structure of the workflow folder will be:
```
├── CMakeLists.txt
├── __init__.py
├── cpp_hb_nogui
│   ├── CMakeLists.txt.mako
│   ├── __init__.py
│   ├── cpp_hb_nogui.workflow.yml
│   ├── cpp_hier_block.py
│   ├── cpp_top_block.py
│   ├── flow_graph_hb_nogui.cpp.mako
│   └── flow_graph_hb_nogui.hpp.mako
├── cpp_hb_qt_gui
│   ├── CMakeLists.txt.mako
│   ├── __init__.py
│   ├── cpp_hb_qt_gui.workflow.yml
│   ├── cpp_hier_block.py
│   ├── cpp_top_block.py
│   ├── flow_graph_hb_qt_gui.cpp.mako
│   └── flow_graph_hb_qt_gui.hpp.mako
├── cpp_nogui
│   ├── CMakeLists.txt.mako
│   ├── __init__.py
│   ├── cpp_nogui.workflow.yml
│   ├── cpp_top_block.py
│   ├── flow_graph_nogui.cpp.mako
│   └── flow_graph_nogui.hpp.mako
├── cpp_qt_gui
│   ├── CMakeLists.txt.mako
│   ├── __init__.py
│   ├── cpp_qt_gui.workflow.yml
│   ├── cpp_top_block.py
│   ├── flow_graph_qt_gui.cpp.mako
│   └── flow_graph_qt_gui.hpp.mako
├── python_bokeh_gui
│   ├── __init__.py
│   ├── flow_graph_bokeh_gui.py.mako
│   ├── python_bokeh_gui.workflow.yml
│   └── top_block.py
├── python_hb_nogui
│   ├── __init__.py
│   ├── flow_graph_hb_nogui.py.mako
│   ├── hier_block.py
│   ├── python_hb_nogui.workflow.yml
│   └── top_block.py
├── python_hb_qt_gui
│   ├── __init__.py
│   ├── flow_graph_hb_qt_gui.py.mako
│   ├── hier_block.py
│   ├── python_hb_qt_gui.workflow.yml
│   └── top_block.py
├── python_nogui
│   ├── __init__.py
│   ├── flow_graph_nogui.py.mako
│   ├── python_nogui.workflow.yml
│   └── top_block.py
└── python_qt_gui
    ├── __init__.py
    ├── flow_graph_qt_gui.py.mako
    ├── python_qt_gui.workflow.yml
    └── top_block.py
```

each workflow will be a separate pythonn module, this will enable us to call the code generator class with python module path that is defined in `generator_module`.

When the user click "generate code", the Generator object will get the information about `generator_class_name` and `generator_module` from the option block, then it instantiate the generator class object. and save it as the Generator object attribute. Then, grc will call `write` method of the generator class object to actually generate the source code.

Below is the implementation of Gemerator class:
```python
class Generator(object):
    def __init__(self, flow_graph, output_dir):
        self.generator_class_name = flow_graph.get_option('generator_class_name')
        self.generator_module = flow_graph.get_option('generator_module')

        generator_module = importlib.import_module(self.generator_module)
        generator_cls = getattr(generator_module, self.generator_class_name)

        self._generator = generator_cls(flow_graph, output_dir)

    def __getattr__(self, item):
        """get all other attrib from actual generator object"""
        return getattr(self._generator, item)
```

The GRC successfully generate the code and executing it
![cpp code genereation](/assets/images/cpp_code_genereation.png)

## Writing Tests

To test this new workflow separation and option block modularization, I will create one grc file for each workflows. The test will be done on `test_generator.py::test_generator`. this function wil load every flow graph on `resources` folder, an check the validity of the flow graph, then generate the source code.

The source code is quite simple:
```python
def test_generator():
    """
    Verify flow graphs then generate source codes
    """
    grc_files = [
        path.join(path.dirname(__file__), 'resources', 'test_cpp.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_compiler.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_python_bokeh_gui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_python_hb_nogui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_python_hb_qt_gui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_python_nogui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_python_qt_gui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_cpp_hb_nogui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_cpp_hb_qt_gui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_cpp_nogui_workflow.grc'),
        path.join(path.dirname(__file__), 'resources', 'test_cpp_qt_gui_workflow.grc'),
    ]

    # read workflow files so it can be loaded by platform
    workflow_paths = [
        path.join(path.dirname(__file__), '../workflows/cpp_hb_nogui'),
        path.join(path.dirname(__file__), '../workflows/cpp_hb_qt_gui'),
        path.join(path.dirname(__file__), '../workflows/cpp_nogui'),
        path.join(path.dirname(__file__), '../workflows/cpp_qt_gui'),
        path.join(path.dirname(__file__), '../workflows/python_bokeh_gui'),
        path.join(path.dirname(__file__), '../workflows/python_hb_nogui'),
        path.join(path.dirname(__file__), '../workflows/python_hb_qt_gui'),
        path.join(path.dirname(__file__), '../workflows/python_nogui'),
        path.join(path.dirname(__file__), '../workflows/python_qt_gui'),
    ]

    platform = Platform(
        name='GNU Radio Companion Compiler',
        prefs=None,
        version='0.0.0',
    )
    platform.build_library(workflow_paths)

    for grc_file in grc_files:
        flow_graph = platform.make_flow_graph(grc_file)
        flow_graph.rewrite()
        flow_graph.validate()

        assert flow_graph.is_valid()

        generator = platform.Generator(
            flow_graph, path.join(path.dirname(__file__), 'resources'))
        generator.write()
```

The workflow_paths array will be used to tell the platform object to also read the workflows files.
You can call the test with: `pytest test_generator.py`
![test generator](/assets/images/test_generator.png)

## Conclusions
* The option block has been modularized by implementing it in Python instead of using a YAML file.
* The workflows are now more modular and pluggable. People can easily insert their own workflows by creating a workflow folder inside the `grc/workflows` directory and following the existing workflow format.
* The tests have already been completed.

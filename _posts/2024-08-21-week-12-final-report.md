---
layout: post
title:  "Week 12: Final Report"
author: zaky
categories: [ gsoc ]
image: /assets/images/end-image.png
featured: true
---
Hi,
GSoC'24 is almost over and it is time for me to bring everything that I did in the past 12 Weeks to a single place, including my work and the things that I learnt and the relationship I have built with my mentors and the GNU Radio Community.

# Project
Title: GRC: Standalone application and pluggable workflows

Mentors: Håkon Vågsether, Sebastian Koslowski

Goals:
* Separate GRC from gnuradio codebase, and make it as a python package
* Make workflows pluggable
* Support Jinja template
* Make High Level API

I have done all of the goals above and I also add A pull request about git pre commit, as a way for developer to automatically pass python code style check in CI. Another side note is that, we decided to keep the dependence of GRC on gnuradio, so GRC can still have access to some useful API such as `gr.prefix()` or `gr.prefs()`

# GRC Separation
This process is being done by removing GRC from gnuradio codebase, then moving freedesktop folder as grc icon for grc-gtk into root of gnuradio directory, then edit CMakeLists.txt to process the freedesktop folder
```
add_subdirectory(freedesktop)
```
another important thing I did is to handle *.conf.in file for grc.

for this, I move every conf.in file for GRC into the root of gnuradio folder, moving code from GRC CMakeLists.txt into the CMakeLists.txt in the project root, so the input files will get processed while gnuradio is being installed by CMakeLists.txt.
You can see the code in the [CMakeLists.txt](https://github.com/ZakyHermawan/gnuradio/blob/grc-removal/CMakeLists.txt)
```
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/grc.conf.in
    ${CMAKE_CURRENT_BINARY_DIR}/grc.conf
@ONLY)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/00-grc-docs.conf.in
    ${CMAKE_CURRENT_BINARY_DIR}/00-grc-docs.conf
@ONLY)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/grc.conf
    DESTINATION ${GR_PREFSDIR}
)
install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/00-grc-docs.conf
    DESTINATION ${GR_PREFSDIR}
)
```

On the GRC side, I move GRC into the new project by using git filter to keep the GRC commit histories, then make a new repo for it.
After that, I make the GRC as a python package by making [pyproject.toml](https://github.com/ZakyHermawan/grc/blob/gsoc/pyproject.toml) and [setup.cfg](https://github.com/ZakyHermawan/grc/blob/gsoc/setup.cfg). I also removing every cmake thing from GRC source code, since we did not use any cmake in the GRC build system anymore.

Another thing I did is to rename the package from grc into gnuradio_companion so it did not confuse with [another python package](https://pypi.org/project/grc/). We also decided to make GRC using src folder structure, so the project structure more or less be like this, where all the grc code will reside in src/gnuradio_companion
```
gnuradio_companion/
│
├── src/
│   │
│   └── gnuradio_companion/
│       └── __init__.py
│
├── tests/
│   └── __init__.py
│
├── README.md
└── pyproject.toml
```

If you want to use GRC as python package that is separated from gnuradio, you can install gnuradio with [grc-removal branch](https://github.com/ZakyHermawan/gnuradio/tree/grc-removal), then clone grc from [gsoc branch](https://github.com/ZakyHermawan/grc/tree/gsoc).

in the root of grc folder, do `pip install .`, you can uninstall it later with `pip uninstall gnuradio_companion`
You can verify the grc installation by doing `import gnuradio_companion` 
![import gnuradio_companion](/assets/images/import-gnuradio_companion.png)

Source Code: 
* [Separated GRC Repository](https://github.com/ZakyHermawan/grc/tree/gsoc)
* [GNU Radio codebase after GRC being removed](https://github.com/ZakyHermawan/gnuradio/tree/grc-removal)

# Pluggable workflow
Another important milestone is to make workflows in GRC is pluggable, so people can make their own workflow.
In order to generate their own workflow, people need to make a folder inside [workflows folder](https://github.com/ZakyHermawan/grc/tree/gsoc/src/gnuradio_companion/workflows) with unique folder name for their own workflow.
This folder consist of `__init__.py`, workflow YAML file, then their own python file for code generation class. This code generation can use mako or jinja template. You can see some examples in workflows folder. There are 9 workflows for current GRC, with 8 is the default workflow that supported without any OOT modules, then one bokeh GUI workflow

The `__init__.py` file is only used to make your code generator class can be accessed by grc.
The workflow YAML file takes 8 argument that must be filled,
```
id (str): Unique name of workflow
descripton (str): Detailed information of workflow
output_language (str): Target language
output_language_label (str): Information for users about the target language of the workflow
generator_class (str): Name of the code generator class
generator_module (str): Module name of where the code generator class is located
generator_options (str): Used to select a workflow
generator_options_label (str): Information for users to select a workflow
```

And five optional fields
```
parameters (list of dict): Parameters for options block
cpp_templates (dict): Templates arguments for C++ Code Generation
templates (dict): Templates arguments for Python Code Generation 
asserts (list): list of assert statements
context (dict): additional workflow parameters
```

[example](https://github.com/ZakyHermawan/grc/tree/gsoc/src/gnuradio_companion/workflows/python_qt_gui):
```yml
id: python_qt_gui_workflow
description: "python qt gui workflow"
output_language: python
output_language_label: Python
generator_module: gnuradio_companion.workflows.python_qt_gui
generator_class: TopBlockGenerator
generator_options: qt_gui
generator_options_label: QT GUI
parameters:
-   id: run
    label: Run
    dtype: bool
    default: 'True'
    options: ['True', 'False']
    option_labels: [Autostart, 'Off']
    hide: ${ 'part' if run else 'none' }
-   id: max_nouts
    label: Max Number of Output
    dtype: int
    default: '0'
    hide: ${ 'none' if max_nouts else 'part' }
-   id: realtime_scheduling
    label: Realtime Scheduling
    dtype: enum
    options: ['', '1']
    option_labels: ['Off', 'On']
    hide: $ { 'none' if realtime_scheduling else 'part' }
-   id: qt_qss_theme
    label: QSS Theme
    dtype: file_open
    hide: ${ 'none' if qt_qss_theme else 'part' }
-   id: run_command
    label: Run Command
    category: Advanced
    dtype: string
    default: '{python} -u {filename}'
    hide: 'part'
templates:
    imports: |-
        from gnuradio import gr
        from gnuradio.filter import firdes
        from gnuradio.fft import window
        import sys
        import signal
        % if generate_options == 'qt_gui':
        from PyQt5 import Qt
        % endif
        % if not generate_options.startswith('hb'):
        from argparse import ArgumentParser
        from gnuradio.eng_arg import eng_float, intx
        from gnuradio import eng_notation
        % endif
```

By the way, the most important thing to make pluggable workflow possible is that I need to convert options block from YAML into python.
You can see how i done it in [options.py](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/blocks/options.py). Then I edit the [platform.py](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/platform.py) file to read my .workflow.yml files
```py
with Cache(cache_file, version=self.config.version) as cache:
    for file_path in self._iter_files_in_block_path(path):
        if file_path.endswith('.block.yml'):
            loader = self.load_block_description
            scheme = schema_checker.BLOCK_SCHEME
        ...
        elif file_path.endswith('.workflow.yml'):
            loader = self.workflow_manager.load_workflow
            scheme = None
```

Source Code:
* [Option Block Modularization](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/blocks/options.py)
* [Pluggable Workflow](https://github.com/ZakyHermawan/grc/tree/gsoc/src/gnuradio_companion/workflows)

# Jinja Support
The next milestone is to add make GRC support jinja templating as a template engine option for code generation. First, I edit options block to add template engine options to choose between using jinja or mako template engine.
Code below is on [options.py](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/blocks/options.py)
```py3
dict(id='template_engine',
    label='Template Engine',
    dtype='enum',
    default='mako',
    options=['mako', 'jinja'],
    option_labels=['Mako', 'Jinja'],
),
```
![Template Engine Options](/assets/images/template-engine-options.png)

This is done by modifying code generator class on every workflow to generate jinja template engine if the user select jinja in `template_engine` parameter.
For example, in python qt gui workflow [code generator class](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/workflows/python_qt_gui/top_block.py):
```py3
    def __init__(self, flow_graph, output_dir):
        """
        Initialize the top block generator object.

        Args:
            flow_graph: the flow graph object
            output_dir: the path for written files
        """

        self._flow_graph = FlowGraphProxy(flow_graph)
        self.generate_options = self._flow_graph.get_option(
            'generate_options')
        self._template_engine = self._flow_graph.get_option('template_engine')

        self._mode = TOP_BLOCK_FILE_MODE
        # Handle the case where the directory is read-only
        # In this case, use the system's temp directory
        if not os.access(output_dir, os.W_OK):
            output_dir = tempfile.gettempdir()
        filename = self._flow_graph.get_option('id') + '.py'
        self.file_path = os.path.join(output_dir, filename)
        self.output_dir = output_dir
        self.env = Environment(
            loader=FileSystemLoader(searchpath=DATA_DIR),
            trim_blocks=True,
            lstrip_blocks=True
        )
        self.env.filters['repr'] = self.repr_filter
        self.template_jinja = self.env.get_template('flow_graph_qt_gui.py.jinja')

...

    def _build_python_code_from_template(self):
        """
        Convert the flow graph to python code.

        Returns:
            a string of python code
        """
        output = []

        fg = self._flow_graph
        platform = fg.parent
        title = fg.get_option('title') or fg.get_option(
            'id').replace('_', ' ').title()
        variables = fg.get_variables()
        parameters = fg.get_parameters()
        monitors = fg.get_monitors()

        for block in fg.iter_enabled_blocks():
            if block.key == 'epy_block':
                src = block.params['_source_code'].get_value()
            elif block.key == 'epy_module':
                src = block.params['source_code'].get_value()
            else:
                continue

            file_path = os.path.join(
                self.output_dir, block.module_name + ".py")
            output.append((file_path, src))

        self.namespace = {
            'flow_graph': fg,
            'variables': variables,
            'parameters': parameters,
            'monitors': monitors,
            'generate_options': self.generate_options,
            'version': platform.config.version,
            'catch_exceptions': fg.get_option('catch_exceptions')
        }

        if self._template_engine == 'jinja':
            flow_graph_code = self.template_jinja.render(
                title=title,
                imports=self._imports(),
                blocks=self._blocks(),
                callbacks=self._callbacks(),
                connections=self._connections(),
                **self.namespace
            )
        else:
            flow_graph_code = python_template.render(
                title=title,
                imports=self._imports(),
                blocks=self._blocks(),
                callbacks=self._callbacks(),
                connections=self._connections(),
                **self.namespace
            )

        # strip trailing white-space
        flow_graph_code = "\n".join(line.rstrip()
                                    for line in flow_graph_code.split("\n"))
        output.append((self.file_path, flow_graph_code))

        return output

```
Just check the value of self._template_engine, either its `'jinja'` or `'mako'`, then just render it.
Then, I make jinja template equivalent for every mako tempalte in each workflow.

Why do we need Jinja as a template engine options ? Since we can let people to define their own workflow, might as well to make them easier to add workflow by using template engine that might be people have more familiarity with, we choose Jinja because it is used in many places, for example on popular web framework (flask), django also use jinja like syntax, Jinja is also used in devops (for example: to write config files)

Source Code:
* Every code generator on each workflows is updated, for example, on [python qt gui workflow](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/workflows/python_qt_gui/top_block.py)
* jinja template is added on each workflow, for example, on [python qt gui workflow](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/workflows/python_qt_gui/flow_graph_qt_gui.py.jinja)

# High Level API
The last milestone is to provide High Level API via proxies, so that the code generator did not use any low level GRC API, this will make the API stable, if some changes being made on low level API, the old code generator did not break because we are using interface that is provided by proxies instead of calling the low level API itself.
You can see the code on [BlockProxy.py](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/generator/BlockProxy.py) and [FlowGraphProxy.py](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/generator/FlowGraphProxy.py)

Source Code: 
* [BlockProxy](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/generator/BlockProxy.py)
* [FlowGraphProxy](https://github.com/ZakyHermawan/grc/blob/gsoc/src/gnuradio_companion/core/generator/FlowGraphProxy.py)


# Pre Commit Git Hooks
Another thing that I'm proposing in my GSoC project is to use pre-commit githook for style formatting as a way to make developer easier to pass the CI tests for code style check.

Pull Request: https://github.com/gnuradio/gnuradio/pull/7431

# Workflow installation tutorial (use bokeh gui for example)
On this tutorial, we will try every feature on my gsoc project, we will use bokeh gui workflow as an example
1. Clone GNU Radio using my [grc-removal branch](https://github.com/ZakyHermawan/gnuradio/tree/grc-removal)
2. On gnuradio folder,
```bash
$ git checkout grc-removal
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
$ sudo ldconfig
```
3. Clone GRC on [gsoc branch](https://github.com/ZakyHermawan/grc/tree/gsoc)
4. In grc folder,
```bash
$ git checkout gsoc
$ pip install .
```
5. to run grc, do: `$ gnuradio-companion`, you can also verify if grc is installed by running `python3 -c "import gnuradio_companion"`
6. install [gr-bokehgui](https://github.com/gnuradio/gr-bokehgui)
7. Run gnuradio-companion
![Show Signal](/assets/images/show_signal.png)

Next we will try to add a new parameter to option block to show the option block modularization.

8. Open `src/gnuradio_companion/workflows/python_bokeh_gui/python_bokeh_gui.workflow.yml`, and add additional_param by changing the YAML file to below:

```YML
id: python_bokeh_gui_workflow
description: "python bokeh gui workflow"
output_language: python
output_language_label: Python
generator_module: gnuradio_companion.workflows.python_bokeh_gui
generator_class: TopBlockGenerator
generator_options: bokeh_gui
generator_options_label: Bokeh GUI
parameters:
-   id: placement
    label: Widget Placement
    dtype: int_vector
    default: (0,0)
    hide: 'part'
-   id: window_size
    label: Window size
    dtype: int_vector
    default: (1000,1000)
    hide: 'part'
-   id: sizing_mode
    label: Sizing Mode
    dtype: enum
    default: fixed
    options: [fixed, stretch_both, scale_width, scale_height, scale_both]
    option_labels: [Fixed, Stretch Both, Scale Width, Scale Height, Scale Both]
    hide: 'part'
-   id: run
    label: Run
    dtype: bool
    default: 'True'
    options: ['True', 'False']
    option_labels: [Autostart, 'Off']
    hide: ${ 'part' if run else 'none' }
-   id: max_nouts
    label: Max Number of Output
    dtype: int
    default: '0'
    hide: ${ 'none' if max_nouts else 'part' }
-   id: realtime_scheduling
    label: Realtime Scheduling
    dtype: enum
    options: ['', '1']
    option_labels: ['Off', 'On']
    hide: ${ 'none' if realtime_scheduling else 'part' }
-   id: run_command
    label: Run Command
    category: Advanced
    dtype: string
    default: '{python} -u {filename}'
    hide: 'part'
-   id: additional_param
    label: Additional Param
    default: ""
    hide: 'none'
templates:
    imports: |-
        from gnuradio import gr
        from gnuradio.filter import firdes
        from gnuradio.fft import window
        import sys
        import signal
        % if generate_options == 'qt_gui':
        from PyQt5 import Qt
        % endif
        % if not generate_options.startswith('hb'):
        from argparse import ArgumentParser
        from gnuradio.eng_arg import eng_float, intx
        from gnuradio import eng_notation
        % endif
    callbacks:
    - 'if ${run}: self.start()

        else: self.stop(); self.wait()'
```
9. Run `pip install .` in root grc folder
10. You should see Additional Param in the options block dialog in If you select Bokeh GUI in Generate Options (this new param only available on bokeh gui generate options since we only edit for bokeh gui workflow on step 8)
![Bokeh GUI additional parameter](/assets/images/Bokeh-GUI-additional-parameter.png)

With this, we successfully run bokehgui workflow (which means we have shown the pluggable workflow feature, you can files for bokeh gui workflow in ), we also show that the options is already modularize, which means you can add new parameters for your own workflow.

Now we will show the jinja template support by editing the template jinja file with out new workflow
11. edit Jinja template file (`src/gnuradio_companion/workflows/python_bokeh_gui/flow_graph_bokeh_gui.py.jinja`), 
```py3
...
if __name__ == '__main__':
    additional_data = "{{ flow_graph.get_option('additional_param') }}"
    print(f"Additional data from parameter: {additional_data}")
    print("hello from jinja")
    main()
```
12. Run `pip install .`
13. Run gnuradio-companion, make a flow graph with Bokeh GUI workflow, fill the Additional Param field with any string (for example, we will fill this param it with string `"additional data"`), select Jinja on Template Engine Parameter, then execute it.

![Additional Param with additional data](/assets/images/additional-param.png)

![Bokeh GUI fLow graph](/assets/images/bokeh-gui-flowgraph.png)
14. You should see the the output on console showing additional data from parameter and hello from jinja.

![Hello Jinja](/assets/images/hello-jinja-console-output.png)

15. The link that is shown on console (for me its localhost:5006)

![Bokeh GUI Show Signal](/assets/images/bokeh-gui-show-signal.png)

We have shown that GRC now support Jinja template engine, and we already show an example on how to use new parameter that we created, and print it to console.

# Demonstration Video
I also publish the [demonstration video](https://youtu.be/z4InG4KTPCs) on youtube

## Closing Remarks
This has been such a ride. I'm lucky to have such wonderful mentors, Håkon Vågsether, Sebastian Koslowski. They have guided me throughout this project. They always give really good feedbacks so that I don't stuck for a very long time and I know what to do on each weeks, it helps me a lot. We did online meeting every weeks, to find any blocker on my progress and suggestme me what to do and tell me if I made mistakes along the way. 

I also thank the gnu radio community, people here are very helpful, sometimes I ask in the matrix channel if i have question about gnuradio and they always give me really helpful answers. I learn so much from this community, before GSoC, I did not know anything about Software Defined Radio.

I also like how GNU Radio organize the Google Summer of Code, for example I am required to post a progress report update to the mailing list, to make people know about my project and it give me some publication (I also get another oportunity to work on Software Defined Radio because someone my posts on GNU Radio Community). I will highly reccomended people to look into GNU Radio organization for next years Google Summer of Code.

I will continue to contribute to GNU Radio after GSoC my period ended!

'til next time,
Zaky

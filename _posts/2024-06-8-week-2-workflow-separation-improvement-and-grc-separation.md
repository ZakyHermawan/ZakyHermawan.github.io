---
layout: post
title:  "Week 2: Workflow Separation Imrpovement and GRC separation"
author: zaky
categories: [ gsoc ]
image: assets/images/new_grc_repository.png
---

Two things that have been done this week, improving the workflow separation by moving some workflow dependant parameter and move grc to a separate repository.

# Improvement on Workflow Separation
After meeting with the mentors and discussing what was done last week, we notice there are still some parameter that is still being hard coded, and some of that parameter is actually workflow-dependant.
For example, gen_cmake and cmake_opt is actually only if the output language is C++,
```py
dict(id='gen_cmake',
    label='Generate CMakeLists.txt',
    dtype='enum',
    default='On',
    options=['On', 'Off'],
    hide="${ ('part' if output_language == 'cpp' else 'all') }",
),
dict(id='cmake_opt',
    label='CMake options',
    dtype='string',
    default='',
    hide="${ ('part' if output_language == 'cpp' else 'all') }",
),
```
To make it more modular, we move these parameters into each cpp workflows. Also, with previous code, people cannot actually making their own parameter without modifying the options block. To fix this issue, we a function to update parameters and call it inside the rewrite method.
```py
def update_parameters_from_workflow(self) -> None:
    new_params_from_workflow = self.current_workflow.parameters
    if new_params_from_workflow == None: # no additional parameter
        return

    new_params_data = build_params(
        params_raw=new_params_from_workflow,
        have_inputs=False,
        have_outputs=False,
        flags=Block.flags,
        block_id=self.key
    )

    for data in new_params_data:
        data['workflow_origin'] = self.current_workflow.id

    new_params: typing.OrderedDict[str, Param] = (OrderedDict(
        (data['id'], param_factory(parent=self, **data)) for data in new_params_data))

    for key, val in new_params.items():
        if not self.params.get(key):
            self.params[key] = val
```
The new parameters will be appended to self.params, each diffrent parameters should have unique id accross every workflows, make sure that if the parameter is defined in multiple workflow, they have exact attributes. Also, note that because we just appending each new parameters into existing parameters, people need to write the hide attribute of each method so that it would be invisible from another unrelated workflow. For example in python hier block qt gui workflow, the parameter is:
```yaml
parameters:
-   id: category
    label: Category
    dtype: string
    default: '[GRC Hier Blocks]'
    hide: ${ ('none' if generate_options.startswith('hb') else 'all') }
```
this category parameter will only be visible if the workflow is a hier block (cpp_hb_nogui, cpp_hb_qt_gui, python_hb_nogui, python_hb_qt_gui). Also, this parameter is defined inside that 4 diffrent hier block workflows. For each hier block workflows, they have same category parameter with exactly same attributes (id, label, dtype, default, and hide).

With this, people can define their own parameters inside their `workflow.yml` file. Just make sure they have unique id vs other diffrent parameters. Also, if you want to add this new parameter, you need to write the hide attribute so that it would be invisible from another unrelated workflow

# GRC Separation
Another milestone of this GSoC project is to make grc as a standalone app with separate repository. In order to achieve that, first, a separate grc repository need to be made, then we make the grc directory in GNU Radio codebase into gitmodule that refer to its own repository. With this, nothing is changed in gnuradio codebase.

The new repository for GNU Radio is in: https://github.com/ZakyHermawan/grc/

How to run grc:
move to `gnuradio/grc` (the root folder of grc)
```
$ pip install .
$ grc
```

## Conclusion
* Workflow-dependant parameter have been moved into its own workflow YAML files
* People now can define their own parameter by modifying their own workflow YAML file
* Now GRC already have a separate repo
* Now GRC is a submodule of GNU Radio

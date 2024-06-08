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

To make it more modular, we move these parameters into each cpp workflows. Also, with previous code, people cannot actually making their own parameter without modifying the options block. To fix this issue, each workflow-dependant parameters will be read and get inserted into `self.workflow_params`.
```py
def parse_workflows(self) -> None:
    for workflow in self.workflows:
        # ...

        params = workflow.parameters
        for param in params:
            param['workflow'] = workflow.id
            self.workflow_params.append(param)
```

Then each time rewrite function getting called, call `update_params_hide` to show/hide parameters based on current workflow
```py
def update_params_hide(self) -> None:
    """
    update hide attributes of self.parameters
    """
    new_params_from_workflow = self.current_workflow.parameters
    if new_params_from_workflow == None: # no additional parameter
        return

    # if the parameters already been updated for current workflow, then do nothing
    if self.params['current_workflow'].get_value() == self.current_workflow.id:
        return

    additional_params = []
    for param in self.workflow_params:
        if param['workflow'] == self.current_workflow.id:
            additional_params.append(param)

    for param in additional_params:
        self.params[param['id']].hide = param.get('hide')

    self.params['current_workflow'].set_value(self.current_workflow.id)
```

Now people can just insert their own parameter inside the workflow YAML file
```yaml
parameters:
-   id: category
    label: Category
    dtype: string
    default: '[GRC Hier Blocks]'
    hide: 'none'
```

# GRC Separation
Another milestone of this GSoC project is to make grc as a standalone app with separate repository. In order to achieve that, first, a separate grc repository need to be made, then we make the grc directory in GNU Radio codebase into gitmodule that refer to its own repository. With this, nothing is changed in gnuradio codebase.

The new repository for GNU Radio is in: https://github.com/ZakyHermawan/grc/

How to run grc:
move to `gnuradio/grc` (the root folder of grc)
```
$ pip install .
$ grc
```

## Conclusions
* Workflow-dependant parameter have been moved into its own workflow YAML files
* People now can define their own parameter by modifying their own workflow YAML file
* Now GRC have a separate repository
* Now GRC is a submodule of GNU Radio

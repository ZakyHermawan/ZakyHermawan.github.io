---
layout: post
title:  "Week 11: Add requirements"
author: zaky
categories: [ gsoc ]
image: assets/images/6.jpg
---
Even tho the project is considered complete, I realized that I haven't add install requirements to prevent the package being built if some requirements is not satisfied.

# Setup.cfg
I add install_requires and python_requires in setup.cfg.
```
[options]
packages = find:
package_dir =
    = src
python_requires = >=3.5
install_requires =
    PyYAML >= 3.11
    Mako >= 1.1.0
    PyGObject >= 2.28.6
    pycairo >= 1.0
    numpy
    jsonschema
```
This will prevent package from being installed if there is at least one requirement that is not satisfied.

# Python requirement
I set the requirement for python is having version >= 3.5, since GRC is using type hints which introduced in python 3.5

## Conclusions
* Python requirements and install requirements have been added to gnuradio-companion

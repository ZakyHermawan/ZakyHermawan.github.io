---
layout: post
title:  "Week 3: Fix parameters modularizations and load YAML files into GRC"
author: zaky
categories: [ gsoc ]
image: assets/images/modularized_options_block.png
---

Main focus of the third week is to fix the parameters modularization and load YAML files to GRC

# Fix paramters modularization
There are some bugs that still happened on my last week's work, the some parameters did not show correctly based on the workflow on PropsDialog
The way I fix this issue is to make all parameters when options block is initialized, just swap them based on current workflow that user choose from PropsDialog.
Thanks to sebastian who suggest me this solution`

# Load YAML files into GRC
When GNU Radio is gettin installed, ALL YAML files will be copied to block's path managed by OS. Which files and where to copy is defined on CMakeLists.
The Problem is, after GRC have a separate repository, it will not running CMake, so the YAML files won't get copied.
To fix this, I modify block_paths from Config class to add all folders that contain YAML files to path_sources, so it will get loaded by the loader in platform.

## Conclusions
* The bugs regarding to parameters modularization is now fixed
* All YAML files are now can be accessed by GRC

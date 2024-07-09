---
layout: post
title:  "Week 7: GRC Removal and Jinja"
author: zaky
categories: [ gsoc ]
image: assets/images/4.jpg
---

This week, im removing grc completely from gnuradio codebase and doing research about jinja.

# GRC Removal
The main idea is to just remove grc folder from gnuradio codebase, this can be achieved by removing necessary cmake codes into root CMakeLists.txt of gnuradio, and also moving freedesktop folder into root of gnuradio, this freedesktop folder is used to install logo for gtk grc.

# Freedesktop
This process is quite simple, just move the folder into root of gnu radio, then modify CMakeLists.txt root:
```
...
add_subdirectory(freedesktop)
...
```

# Moving the input conf file
in grc, there is grc.conf.in and 00-grc-docs.conf.in, the logic of using input config file into the root CMakeLists.txt.
So, I modify the CMakeLists.txt into:
```
...
########################################################################
# Install configuration files for GRC
########################################################################
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
...
```

# Set ENABLE_GRC variable in CMakeLists.txt
the variable ENABLE_GRC is being set in CMakeLists.txt inside grc directory, but since the grc now is a python package, mean no CMakeLists.txt is allowed, I also move this logic into root CMakeLists.txt. The modified CMakeLists.txt is:
```
...
GR_REGISTER_COMPONENT("gnuradio-companion" ENABLE_GRC
    ENABLE_GNURADIO_RUNTIME
    ENABLE_PYTHON
)
...
```

# Jinja
???

# How to install GNU Radio + GRC
You can either install GNU Radio first or GRC first, it does not matter.
1. Install GNU Radio
2. Move to grc root directory (same level as pyproject.toml), then run `pip install .`
![GRC now installed](/assets/images/grc_installed.png)

## Conclusions
* Now GRC has been removed from gnuradio codebase
* You can install grc using pip install
* Now GRC GTK will have icon, no matter either you install GNU Radio or GRC first
* Jinja: ?

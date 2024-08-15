---
layout: post
title:  "Week 10: Merge Everything on GRC separated repo"
author: zaky
categories: [ gsoc ]
image: assets/images/complete_puzzle.png
---
Since the GSoC period is almost done, Im working toward final report,
from last meeting, I decided to turn all my focus on completing everything on GRC Separated repository, because at the end, I will be writing report and demo using code from GRC repo instead of gnuradio repo.


# GRC Separate repo
On GRC repository, There are 4 important thing, GRC is becoming a python package which can be installed using `pip install .` and completely separated from gnuradio codebase, despite still depend on gnuradio API.
We decided to do this way, because GRC currently only used with gnuradio, and dependent on gnuradio have some advantage, for example, we can use some gnuradio APi like `gr.prefix()` or `gr.prefs()`.
Anothing thing that we add is making high level interfaces/API for code generation, so people who make their own workflows and codegen class can use this interfaces instead of interacting with core APIs.
You can see the code in gsoc branch of grc [repository](https://github.com/ZakyHermawan/grc/tree/gsoc).

# High Level Codegen API
This high level codegen live inside gnuradio_companion.core.generator module. It consist of `FlowGraphProxy` and `BlockProxy` to prevent people to access core API directly from for code generation. This will make the user defined workflow have more stable.

## Conclusions
* The code for GSoC project is completed
* Add High Level Code Generation API

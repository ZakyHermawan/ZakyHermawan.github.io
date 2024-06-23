---
layout: post
title:  "Week 4: Restructure folders and further research on solving cmake problems"
author: zaky
categories: [ gsoc ]
image: assets/images/cpp_code_genereation.png
---

This week I got stuck on the problem where the GRC cannot generated code correctly for C++, this is actually the problem on my local, but I finally solved it.
Another thing Im doing is I re-structure the project into src layout.
```
grc/
│
├── src/
│   │
│   └── grc/
│       └── __init__.py
│
├── tests/
│   └── __init__.py
│
├── README.md
└── pyproject.toml
```

The last thing i did this week is doing research on how to solve the cmake problem, because after GRC being separated from GR codebase, we dont use any cmake, so I need to figure out how should GRC handle the CMakeLists.txt. Another challenge is that, there is `conf.in` file, where this file is being processed while while cmake command is executed via commandline. As I said before, since we dont use any cmake, I need to figure out how to fix this issue.

One thing that comes in mind is to write the scripts to kind of copy what cmake doing, I also see some package that might help, I already try another diffrent approaches, but I'm not feeling sure which approach should I do, well, cant say much about this, lets just see on the next week.

Even tho the progress on this week is not that much (since I got stucked on codegen problem). I still ahead of schedule, hopefully I can keep this up and not fall behind.

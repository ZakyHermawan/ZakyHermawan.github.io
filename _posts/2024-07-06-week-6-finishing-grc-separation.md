---
layout: post
title:  "Week 6: Finishing GRC separation"
author: zaky
categories: [ gsoc ]
image: assets/images/grc_separation_pr.png
---

Main focus on this week is week is to finish my work on GRC separation, fix tests, and make PRs

# Pull Request
I have made 2 PR about grc separation, first [PR](https://github.com/gnuradio/gnuradio/pull/7419) is to make grc as a submodule and fix the CI to checkout recursively. Second [PR](https://github.com/gnuradio/gnuradio-companion/pull/1) is to move grc code into official gnuradio/gnuradio-companion [repository](https://github.com/gnuradio/gnuradio-companion/).

# GRC Package Name and Script name
gnuradio-companion was choosen as a script name to avoid confusion with https://pypi.org/project/grc/, also, gnuradio_companion is choosen as the package name of grc because python package did not allow '-' as a package name, so if someone want to import grc, they can do something like: `import gnuradio_companion`.

# Fixing tests
in GNU Radio, we use relative import in our tests, this is not correct, while doing tests, we need to actually import the package and test its functionalities. So im chaning from: `import grc.something` to `import gnuradio_companion`, there are some problem when i try to fix the tests, i.e circular import error happened, but I have fixed it.

# Change entry point command
Before this week, im runnig main function from main.py and compiler.py, after i realized that, there is script called gnuradio-companion and grcc, which what gnuradio usually execute when we write gnuradio-companion or grcc in commandline. So im making our entry point to execute gnuradio-companion/grcc. instead of main.py/compiler.py.

# Testing
The test is consist of 3 parts. First is to generate code and execute code from flow 11 diffrent graphs, then run `make install` and `pytest`.

# Git Hooks ?
I promised that I will make git hooks for GNU Radio in my proposal, especially pre-commit, because the problem with our source code is that people need to run autoformatter script by themself to pass code style checks. We can automate this process by running autoformat script when people want to commit their work. But I will make this feature only after every PR on my grc separation is being merged.

![make test](assets/images/make_test.png)
![pytest](assets/images/pytest.png)

## Conclusions
* PRs have been made for to make grc into submodule
* work on GRC separation is finished
* Testing is done
* Git hooks will be added after every PR about GRC separation is being merged

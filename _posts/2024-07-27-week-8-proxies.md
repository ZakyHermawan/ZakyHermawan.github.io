---
layout: post
title:  "Week 8: GRC Removal and auto format pre-commit"
author: zaky
categories: [ gsoc ]
image: assets/images/proxy-diagram.png
---
This week, im making PRs and add proxy class to enable higher level API

# Jinja Support Pull Request
This [PR](https://github.com/gnuradio/gnuradio/pull/7449) is more of a proposing idea on jinja template support, if got accepted then i will continue to work on jinja template support

# Pluggable workflow
Because my work on pluggable workflow is finished, I made the [PR](https://github.com/gnuradio/gnuradio/pull/7445) to gnurardio codebase

# Proxy classes
Because we enable people to write their own workflow, we need to make higher level API for code generation so when the core API is changed (For example: method name is changed), it will not break anything
The code for proxies will be on [proxies branch](https://github.com/ZakyHermawan/gnuradio/tree/proxies)

## Conclusions
* 2 pull requests have been made
* Proxy classes have been made to provide higher level API for code generation

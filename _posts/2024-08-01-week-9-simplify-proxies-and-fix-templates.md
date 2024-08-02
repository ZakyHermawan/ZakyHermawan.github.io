---
layout: post
title:  "Week 8: GRC Removal and auto format pre-commit"
author: zaky
categories: [ gsoc ]
image: assets/images/partial_jinja_generated.png
---
This week, im spending my time to simplify the proxy classes and fix templates

# Proxy simplification
After receiving command by sebastian: The here is to create proxies for all objects that are currently used in generator templates. Mako templates should not have access to the implementation classes.
These proxies should only provide the minimal required attributes and methods. And in no way should a generator be able to alter a proxied object (e.g. rewrite() is not sth to be proxied).
I realize that I overcomplicate things by making abstract class and make abstract method of the implementation class, while the only thing that I need to do here is to just make a proxy class with as minimal method as possible to provide interface so code generator do not access implementation classes directly.

# Fix templates
I continue my work on jinja template support, there are so many limitation when using jinja template compared to mako template, for example,
in mako template, you can just write the embedded python code directly, you can't do this directly in jinja. Let's take a look on how to do list comperhension
in mako:
param_str = ', '.join(['self'] + ['%s=%s'%(param.name, param.templates.render('make')) for param in parameters])

to convert it to jinja, we need to do:
```
{% set parts = ['self'] %}

{% for param in parameters %}
    {% set _ = parts.append('{}={}'.format(param.name, param.templates.render('make'))) %}
{% endfor %}

{% set param_str = ', '.join(parts) %}
```
I for now, not every code from jinja template have been success fully generated, but im currently working on it!

## Conclusions
* The implementation of proxy need to be simplified
* The jinja template support is currenly work in progress to fix

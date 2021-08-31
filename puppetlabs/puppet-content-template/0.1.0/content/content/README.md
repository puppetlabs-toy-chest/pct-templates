# Puppet Content Template Author Tips

## Writing Templates

### Structure

A PCT is an archive containing a templated set of files and folders that represent a completed set of content. Files and folders stored in the template aren't limited to formal Puppet project types. Source files and folders may consist of any content that you wish to create when the template is used, even if the template engine produces just one file as its output.

### Composition

A PCT must contain a `pct-config.yml` in the root directory, alongside a `content` directory. The root directory must be named the same as the `id` value defined in  your `pct-config.yml`

The `content` directory contains the files and folders required to produce the `project` or `item`.

To mark a file as a template, use the `.tmpl` extension. Templated files can also use the global variable of {{"`{{.pct_name}}`"}} to access the input from the `--name` cli argument.

> :memo: Folders within the `content` directory can also use the {{"`{{.pct_name}}`"}} variable

Example template file names:

``` bash
myConfig.json.tmpl
{{`{{.pct_name}}`}}_spec.rb
```

> :memo: One, all or none of the files can be templated.

#### pct-config.yml

Format of pct-config.yml

``` yaml
---
template:
  id: <a unique name>
  type: <'item' or 'project'>
  display: <a human readable name>
  version: <semver>
  url: <url to project repo>

<template parameters>
```

> :memo: Template `id` must not contain spaces or special characters. We recommend using a hyphen to break up the identifier.

Example pct-config.yml:

``` yaml
---
template:
  id: example-template
  type: project
  display: Example
  version: 0.1.0
  url: https://github.com/puppetlabs/pct-example
```

Example structure for `example-template`:

``` bash
> tree ~/.pdk/pct/example-template-params
/Users/me/.pdk/pct/example-template-params
├── content
│   └── example.txt.tmpl
└── pct-config.yml
```

### Templating Language

PCT uses [Go's templating language](https://golang.org/pkg/text/template/#hdr-Actions).

Example pct-config.yml with parameters:

``` yaml
---
template:
  id: example-template-params
  type: project
  display: Example with Parameters
  version: 0.1.0
  url: https://github.com/puppetlabs/pct-example-params


example_template:
  foo: "bar"
  isPuppet: true
  colours:
  - "Red"
  - "Blue"
  - "Green"

```

In the above template `example-template-params` the parameters can be accessed in a `.tmpl` file like so:

``` go
{{.example_template.foo}}
{{.example_template.isPuppet}}
{{.example_template.colours}}
```

outputs:

``` text
bar
<no value>
[Red Blue Green]
```

As a template author you can chose your own parameters and parameter structure so long as it is [valid YAML](https://yaml.org/spec/1.2/spec.html). Then utilise the GO templating language to display or iterate over these.

For most templates, we believe that you can do most of the things you would want with these common template controls:

``` go
// Outputs the value of `foo` defined within pct.yml
{{.example_template.foo}}

// A conditional
{{if .example_template.isPuppet}}
 "boo :("
{{else}}
 "yay!"
{{end}}

// Loops over all "colours" and renders each using {{.}}
{{range .example_template.colours}} {{.}} {{end}}
```

For more examples look at the existing templates provided at `$HOME/.pdk/pct/`

### Dos and Don'ts

* `project` templates should provide all the code necessary to create a project from scratch and no more.
* Do not include configuration files that can be added via an `item` template later by an end user, for example, CI job configuration.
* Templates should be self documenting to help guide new users on how to use the file that has been created.

## Overriding Template Defaults

Perhaps you use a template often and find that you set the same values over and over?
As a template user, you can choose to override the default values specified by a template author.

> :memo: To view the default parameters for a template, look at it's `pct-config.yaml`. You can find all the current installed templates in the default template location: `$HOME/.pdk/pct/`

To override these defaults you need to create a `pct.yml` within `$HOME/.pdk/`.  This file can contain overrides for multiple templates.

Example:

``` yaml
example_template:
  foo: "wibble"
  isPuppet: false
  colours:
  - "Red"
  - "Blue"

another_template:
  key: "value"
```

> :memo: You don't need to override everything
>
> ``` yaml
> example_template:
>  isPuppet: false
> ```
>

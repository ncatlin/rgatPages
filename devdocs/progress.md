---
subtitle: Development - Design Overview
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## rgat

dev stuff

library choices 

modules etc 
[example]


### Features

- GPU accellerated graph layout
- Thread preview graphs
- Trace replay
- Heatmap generation
- API recording
- Signature scanning with YARA and Detect-It-Easy support
- Customisable instrumentation (choose what is traced and how much)
- Remote tracing - perform tracing and view the results on different machines
- Command line tracing - record a trace without a GPU for future replay

Check the Trello for the features under development or scheduled to be worked on.



## Requirements and Installation

The two main requirements are:
- Windows, with the ability to run .NET Core programs
- For GUI usage: A GPU with Vulkan support

To install:
- Download a release
- Unzip it somewhere
- Run rgat.exe
- Either start using it or configure it to your liking in the settings

## Documentation

- Usage Guide
- How it works
- Development

### Technologies

- [Intel Pin](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) for generating instruction traces
- [Veldrid](https://github.com/mellinoe/veldrid), a .NET graphics library
- [Dear ImGui](https://github.com/ocornut/imgui) providing the GUI, via [ImGui.NET](https://github.com/mellinoe/ImGui.NET)
- Force-directed graph layout based on Jared McQueens [WebGL algorithm](https://github.com/jaredmcqueen/analytics/tree/eed32e17922ef16288984e27f46717e8b7a2d602)
- [Yara](https://github.com/virustotal/yara), via the Airbus CERT [dnYara library](https://github.com/airbus-cert/dnYara)
- A woefully incomplete .NET [port](https://github.com/ncatlin/DiELibDotNet) of the [Detect-It-Easy engine](https://github.com/horsicq/DIE-engine)
- [PeNet](https://github.com/secana/PeNet) for static analysis of PE binaries

A full list and discussion of libraries can be found in the development documentation

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
code
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/ncatlin/rgat/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
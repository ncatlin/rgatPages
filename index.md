---
subtitle: Instruction Trace Visualisation
layout: page
show_sidebar: true
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## rgat

rgat is a software reverse engineering tool which collects and visualises instruction traces from target binaries. It is intended to bridge the gap between the high level API view of malware sandboxes and the low level function view of disassemblers/decompilers. 

It currently supports 32 and 64 bit Windows EXE's and DLL's, but has been recently rewritten for .NET Core so Linux support shouldn't be too far off. 

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

The two main requirements for 0.6.0 are:
- Windows, with the ability to run .NET 5 programs
- For the computer running the visualiser: A GPU with Vulkan driver support (ie: [_this_](https://github.com/skeeto/vulkan-test) works)

To install:
- Download a release
- Unzip it somewhere
- Run rgat.exe
- Configure it to your liking in the settings

## Documentation

- [Usage Guide](userdocs/overview.md)
- [How it works/Development](devdocs/overview.md)

### Technologies

- [Intel Pin](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) for generating instruction traces
- [Veldrid](https://github.com/mellinoe/veldrid), a .NET graphics library
- [Dear ImGui](https://github.com/ocornut/imgui) providing the GUI, via [ImGui.NET](https://github.com/mellinoe/ImGui.NET)
- Force-directed graph layout based on Jared McQueens [WebGL algorithm](https://github.com/jaredmcqueen/analytics/tree/eed32e17922ef16288984e27f46717e8b7a2d602)
- [Yara](https://github.com/virustotal/yara), via the Airbus CERT [dnYara library](https://github.com/airbus-cert/dnYara)
- A woefully incomplete .NET [port](https://github.com/ncatlin/DiELibDotNet) of the [Detect-It-Easy engine](https://github.com/horsicq/DIE-engine)
- [PeNet](https://github.com/secana/PeNet) for static analysis of PE binaries

A full list and discussion of libraries can be found in the development documentation

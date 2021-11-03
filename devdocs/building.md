---
subtitle: Development - Building
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
# Building

The [github actions script](https://github.com/ncatlin/rgat/blob/master/.github/workflows/BuildWindows.yml) builds rgat using some custom-compiled libraries. All compilation is tested in Visual Studio 2019 Community.

### rgat Core and UpdateFinalizer

As a .NET project built around nupkg libraries rgat should be very easy to build. The only gotcha (at the moment) is custom built libraries that have not been put on NuGet by their maintainers. These are available by adding https://nuget.pkg.github.com/ncatlin/index.json as a source, or you can fetch and build the libraries listed in the [libraries](/devdocs/libraries) section.

UpdateFinalizer is trivial .NET and should compile without issue.

### Tests, DLL-Loaders

The tests and DLL loaders are a mix of assembly and basic C/C++ with no unusual dependencies which should compile out of the box with Visual Studio.

### Pin Tool

This is the one item that has the potential to put people through days of compiler hell. Intel have a [compiling guide with a list of possible issues](https://software.intel.com/sites/landingpage/pintool/docs/98437/PinCRT/PinCRT.pdf). As long as you have Pin setup in the right place (see the CI script at the top) then the rgat pintool should compile fine.
---
subtitle: Development - Libraries/Tools
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
# Libraries

This section describes third party (or third party derived) code included in or with rgat and the rationale for using them


#### [Intel Pin](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) (Dynamic Binary Instrumentation)

Originally rgat used DynamoRIO and I'm fond of it for being open source, well documented and actively developed - the support for unusual instructions has been slow to appear and there were times when it [just wouldn't run](https://github.com/DynamoRIO/dynamorio/issues/2524) or stopped working when new Windows builds were released. Ideally, once the Pin tool is stable and passing a nice comprehensive suite of tests then a DynamoRIO tool can be re-implemented and the tools can either be both supported or the one which works best. By 'best' that will either be in terms of performance or ease of providing APi instrumentation.


#### [Capstone](https://www.capstone-engine.org/) (Disassembler)

Another migrant from before the re-write, Capstone has a C# bindings that made it trivial to write into the new code. 
It is a native binary though and I would like to cut back on these for when Linux combatibility is added. 

[Iced](https://github.com/icedland/iced) is the obvious alternative to move to, but I had a go and the API/data available just doesn't fit with how rgat is currently written, so this is a bigger bit of work than expected. I also have no idea how the speed compares, but disassembly with Capstone is only a bottleneck when saving or loading very large traces.


#### [ImGui](https://github.com/ocornut/imgui) via [ImGui.Net](https://github.com/mellinoe/ImGui.NET) (GUI)

The rgat re-write stalled for quite a while due to a failure to find a UI framework that met three requirements
* Cross platform (rules out windows forms)
* Must have support for direct GPU drawing ( which [ruled out the main .NET Qt implementation](https://github.com/qmlnet/qmlnet/issues/79) )
* Must be reasonably well supported or actively developed, which ruled out a few small projects that looked reasonable but probably wouldn't have supported some of the features needed

I looked into browsery/javascripty solutions and then found ImGui - beloved tool of video game (and cheat) developers everywhere. The documentation is done by example so it's a bit of a learning curve getting used to the core concepts (What's a frame? What's a child? What's a... child frame? What do '#','##' and '###' do in labels?) but it's quite nice to program in compared to the Signal/Slot/QML/LGPL set of issues that come with Qt.


#### [Veldrid](https://github.com/mellinoe/veldrid) (GPU access)

Veldrid is an essential library for both running ImGui.NET and providing a clean interface for rgat to perform GPU compute and graph rendering from .NET. There are a [couple](https://github.com/mellinoe/veldrid/pull/399) of critical [fixed](https://github.com/mellinoe/veldrid/pull/412) issues that haven't made their way to the NuGet repo, so until that happens rgat will be built with [a custom built version](https://github.com/ncatlin?tab=packages&repo_name=veldrid)


#### [Analytics](https://github.com/jaredmcqueen/analytics) (Force Directed Graph Layout)

This gorgeous JavaScript/WebGL demo of 3D force-directed (Fruchterman-Reingold) graph layout was what finally made using force-directed graph layout seem plausible. The shaders are reimplemented as Vulkan Compute shaders and the code has evolved a bit but the Velocity/Position/Attribute shader combination is the core of the GraphLayoutEngine.

Note that this code was integrated into the rgat codebase before the licence changed from MIT/Metastack to GPL.


#### [PeNet](https://github.com/secana/PeNet) (Portable Executable parsing)

This is used for the file info in the trace launching tab, deciding how to launch files and sees extensive use in DieLibDotNet.


#### [Yara](https://virustotal.github.io/yara/) (via [dnYara](https://github.com/airbus-cert/dnYara)) (Signature Scanning)

Ever the indispensible malware analysis tool, Yara is made accessible via the airbus-cert C# wrapper. As with Veldrid, this required a few changes to work nicely with rgat (external variable support and the use of precompiled rule files) that aren't on NuGet yet - so rgat comes with a [custom version](https://github.com/ncatlin?tab=packages&repo_name=dnYara)

Yara inclusion brings in the requirement to bundle an unmanged binary. rgat includes one with OpenSSL compiled in so hash based signaturing is supported, because what's another megabyte or two? 


#### [Detect It Easy's DIE engine](https://github.com/horsicq/DIE-engine) (via [DieLibDotNet](https://github.com/ncatlin/DiELibDotNet)) (Signature Scanning)

I really wanted the file-type detection capabilities of DIE, but the library has a hard requirement on Qt and all the unmanaged binaries that come with it. After having just migrated away from Qt it would have been silly to re-add it just for signature scanning, so rgat uses a custom built C# port of DIE's engine. It only supports the enough features to run the bulk of the PE scanning scripts (no ELF or MACH) and may not run them all very well - but there is such a rich range of [detection scripts](https://github.com/horsicq/Detect-It-Easy/tree/master/db/PE) that it's worth it.


#### [Jint](https://github.com/sebastienros/jint) (Javascript interpreter)

Jint is a .NET Javascript interpreter which is used by DiELibDotNet in place of Qt's Qt Script interpreter. There is also scope for rgat to support user scripting with it in the future.


#### [GraphShape](https://github.com/KeRNeLith/GraphShape) and [QuikGraph](https://github.com/KeRNeLith/QuikGraph) (Analysis Chart Graph Layout)

With all the effort of getting instruction trace graphs drawn, the hope was that drawing the simple process/thread/file graphs in the analysis tab could be done simply by a popular library. I tried [Microsoft Automatic Graph Layout](https://github.com/microsoft/automatic-graph-layout) but it just didn't produce a good result for this use case with zooming/live adding + replotting nodes. GraphShape gives a better result but has to be fully replot on every change and zooming implementation in rgat is not pleasent.

I'm probably just using these libraries wrong, we might find out next time there is an analysis chart sprint.

#### [Humanizer.Core](https://github.com/Humanizr/Humanizer) (Text processing)

Lovely library for text processing of unit quantities. rgat probably doesn't make the most of its functionality.

#### [NaCl.Core](https://github.com/daviddesmet/NaCl.Core) (Crypt)

Provides the ChaCha20Poly1305 implementation used for remote trace encryption.

#### [Json.Net](https://www.newtonsoft.com/json)

rgat used Newtonsoft as a C++ project and felt a better fit than the System.Text.Json API, which caused problems when trying to decode objects of unknown format. It would probably work but no need to change.

#### [CommandLineParser](https://github.com/commandlineparser/commandline) (Command Line... Parsing)

rgat includes a [custom version](https://github.com/ncatlin/commandline/packages/998130) of this because at the time of writing the version of GetOpt support was not available on NuGet. Hopefully this will change by the time command line tracing is implemented.

#### [FFMpeg](https://www.ffmpeg.org/) (via [FFMpegCore](https://github.com/rosenbjerg/FFMpegCore)) (Video Capture)

FFMpeg is not distributed with rgat because it's
 - Huge
 - Optional (for people who want builtin video capture rather than using something like [OBS Studio](https://obsproject.com/))
 - Has Licence issues

 It is mainly intended for future command-line tracing support where video is captured automatically.
---
subtitle: Development
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## Progress

In 0.6 rgat finally has a reasonably portable and maintainable codebase, but while it's functional it's probably not genuinely useful on real targets. 

#### Performance Bottlenecks

Running a 'big' target (eg: VLC) presents two major problems:

* The number of nodes overwhelms the layout algorithm. It gets up to 400k nodes within seconds and the FR-node pipeline grinds to a halt. This can be mitigated a little by changing layouts or filtering out some of the many, many different modules VLC executes but fine-grained filtering within modules is needed.

* Tracing - It's a testament to the amazing power of JIT'ing that - in release builds - the C# port of rgat can clear the trace backlog faster than the pin client can generate it. This may mean that the bottleneck is in tracing and I'm not sure how much more room there is for optimisation there.

#### Layout Quality

* Standard Fruchterman-Reingold is probably just not the best fit for control flow graphs in aesthetic terms. But:
* None of the layouts are going to work very well until the node clumping issue is dealt with. This is likely going to mean writing out high-degree nodes from the layout algorithm. I tried reducing the force of their edges but results were not that helpful.


#### Stability

* Every crash encountered so far has been reproduced and removed, but testing has not been rigorous due to how fast the code has been changing. The use of un-managed Vulkan and Imgui API usage will likely be a source of crashes as the UI and layout pipelines are refined.

#### Trace Accuracy

* The pintool tracing code is much similar and more robust now (at the expense of some performance) but it is almost certainly going to break or at least be inaccurate on more interesting bits of code.

#### API Tracing

* This is hamstrung by requiring implementation in both rgat.exe and the pintool. Ideally find a way to have the pintool load everything it needs from the same file rgat uses to interpret API parameters/effects.

## Roadmap

* 0.6 is a 'it works now, but as a research project' release
* 0.7 will come out when it feels actually useful on real targets
* 0.7 -> 0.8 will start getting more novel functionality - memory operations would be idea. Also think about a Linux port as all the core functionality should be settled by now.

### 0.7 goals/requirements

* Fixes for any showstopping bugs in other peoples environments (incl. NVIDIA GPU testing)
* Creation of a suite of test cases
* A command-line tracing mode to integrate test-cases into CI
* Reliable generation of useful layouts
* Resilience with large traces
* API instrumentation revamp
* Instrumentation of usefully large set of APIs
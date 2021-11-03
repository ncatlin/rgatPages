---
subtitle: Documentation - Usage Overview
layout: page
menubar: userdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## Usage

### Trace Setup

#### Preparation Step 1 - Initialise Remote Tracing  *(Optional)* 

To run the target on a different machine than the one being used for visualisation (eg: tracing malware on a VM or other analysis environment) you will need to establish a remote tracing connection
[This is documented here](remote-tracing)  

#### Preparation Step 2 - Choose a target or saved trace

Use the splash screen or main menu to load a target. This will take you to the trace launching tab. If loading a trace then you can processed straight to the visualiser/analysis tabs.

#### Preparation Step 3 - Customise tracing settings  *(Optional)* 

Customise what will be traced and how, as [documented here](ui-starttracetab), and click **Start Trace** to run the target.

### Trace Analysis

#### Runtime Visualisation

In the [Visualiser Tab](ui-visualisertab) you can view the threads being plotted as the trace is collected and start manipulating, navigating and highlighting the graphs immediately. Processes can be paused, resumed and terminated if you want to adjust the trace settings and re-launch.

#### Replay Visualisation

If you didn't select "Discard Replay Data" in the launching tab then when the trace terminates you will be able to replay the recorded trace.

#### Analysis Tab

The [analysis chart](ui-analysistab) is only lightly implemented in release 0.6.0, but it lists various events that were recorded during tracing, such as the launch of processes and threads, and API calls.

#### Saving/Loading

Traces can be saved to disk at any time through the menu bar 'Target' option

#### Media Capture

Image capture and, with FFMpeg.exe configured, Video recording is available through the relevant keybind.
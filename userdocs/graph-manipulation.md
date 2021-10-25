---
subtitle: Documentation - Usage Overview
layout: page
menubar: userdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## Graph Manipulation

- [Graph Manipulation](#graph-manipulation)
  - [Navigation](#navigation)
    - [Modifiers](#modifiers)
    - [Yaw](#yaw)
    - [Pitch](#pitch)
    - [Roll](#roll)
    - [Movement](#movement)
    - [Auto Centering](#auto-centering)
  - [Static Layouts](#static-layouts)
  - [Force Directed Layouts](#force-directed-layouts)
  - [Feature Colouration](#feature-colouration)
    - [Heatmap](#heatmap)
    - [Conditionals](#conditionals)
    - [Degree](#degree)

### Navigation

All graphs are rendered in a 3D space and can be rotated and moved. 

#### Modifiers 

Many keybinds can be used with keyboard modifiers to change their magnitude

| Modifiers  | Multiplier   | Note | 
|---|---|---|
|  None | No Multiplier | Small movements |
|  Shift  | Small Multiplier  | Moderate movements | 
|  Ctrl  | Large Multiplier  | Huge movements | 
|  Shift + Ctrl  | Proportional Multiplier  | Larger or smaller than ctrl, depending on the current value. Useful for excessive plot sizes. |



The following useful default keybinds are configurable

#### Yaw

| Action  | Keybind   | Mouse |
|---|---|---|
|  Yaw + | End  | Alt+ Drag Left |
|  Yaw - | Delete  | Alt+ Drag Right|

![yaw](https://user-images.githubusercontent.com/5470374/138618757-94512557-d833-4d51-9609-7a7eda38ba82.mp4)

#### Pitch

| Action  | Keybind    | Mouse |
|---|---|---|
|  Pitch Forward | Page Up  | Alt+ Drag Up |
|  Pitch Back | Page Down | Alt+ Drag Down|

![Pitch](https://user-images.githubusercontent.com/5470374/138619203-28f0a8d8-4116-40e1-95f8-dc7316fc8a36.mp4)

#### Roll

| Action  | Keybind   | Mouse |
|---|---|---|
|  Roll Clockwise | Home  | |
|  Roll AntiClockwise | Insert  | |

![Roll](https://user-images.githubusercontent.com/5470374/138619321-140e8342-042d-4eee-a4dd-0d5de0e79bae.mp4)

#### Movement

Standard WASD and keyboard arrows can be used for moving on the X and Y axis, but mouse dragging is usually more comfortable.
You can also drag the sliders in the control pane or double click them to set specific values.

#### Auto Centering


| Action  | Keybind   |
|---|---|---|
|  Center View | Q  | 
|  Keep View Centered | Shift-Q  | 


### Static Layouts


Static layouts place instruction nodes according to rules, so they run very quickly and result in predictable plots with minimal configuration needed. 

The Cylinder respects call/return relationships so it gives a traditional control flow graph layout

![Cylinder Video](https://user-images.githubusercontent.com/5470374/138753109-07f8bd52-dc6e-4143-b3ed-eaeb13076922.mp4)

The circle layout may also be useful for getting an idea of how the graphs connections

![Circle Video](https://user-images.githubusercontent.com/5470374/138760236-d34a4bda-992d-4ca3-bf43-3b51023b9bc9.mp4)


### Force Directed Layouts

The force-directed layouts offered by rgat in 0.6.0 are a basic Fruchterman-Reingold algorithm that repels each node away from each other and then attracts connected nodes towards each other. 

The force applied is controlled by a temperature, which falls over time. After a number of steps (depending on the settings you apply) the nodes fall into an equilibrium which may or may not give you some insight into the structure of the program.

The two layouts offered are:

* Force-directed nodes: Each node is laid out independently
* Force-directed blocks: As above, but nodes are grouped together as basic blocks.

The blocks algorithm is significantly less taxing on the GPU and will probably give a more useful result. Depending on your GPU, the nodes algorithm may be able to handle a few hundred thousand unique instructions on a graph - at which point other layouts should be used.

It's important to limit the amount of instrumented code to make this practical - though at the moment rgat can only do this at a module-level granularity.

Using the [Quick Menu](ui-visualisertab.md#quick-menu) may be essential to getting a useful layout depending on the structure of the target you are working with.

![Node Repulsion](https://user-images.githubusercontent.com/5470374/138764236-8be52728-7e3a-4805-b50d-5f4aab3214ec.mp4)

*Adjust node repulsion to control the size of the layout*

![Speed Limits](
https://user-images.githubusercontent.com/5470374/138764751-d1946f2b-3797-4b74-b597-d3059fd9bfc1.mp4)

*High temperatures and speed limits can make graph layout fast, but nodes will shake violently when they approach equilibrium*

![Replotting](https://user-images.githubusercontent.com/5470374/138766266-8bed8d95-c9aa-42a4-931e-4250c0c96e2c.mp4)

*Experiment with different replotting methods*

### Feature Colouration

| Action  | Keybind   |
|---|---|---|
|  Heatmap | X  | 
|  Conditionals | C  | 


As well as the standard control flow colours, which depict the type of control flow (call, jump, ret, external, none-new, none-old or exception), other renderings can be chosen:

#### Heatmap

The heatmap can show areas of high activity in a large plot

![Heatmap](../img/heatmap.png)


#### Conditionals

The conditionals render offers a quick way to see if a conditional jump as always been taken, never been taken or fully explored.

![Conditional](../img/conditionals.png)


#### Degree

The degree plot shows how connected nodes are using heatmap colouration. Nodes above the clump limit have their own colour, which shows the nodes affected by the clumping multiplier in the force directed layout settings.

![Degree](../img/degree.png)

---
subtitle: Development - Testing
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
# Testing

Having a big suite of tests is be essential to verify that the traces gathered are (and remain) correct enough. rgat is in the awkward stage of its development where a lot of time and effort spent on creating a built-in test harness but not actually having many tests to run yet. This is because the command-line-tracing mode isn't implemented yet, which could allow tests to be integrated into the CI process.

### The test harness

### Tests

### Creating Tests

Build dependencies -> masm
Right cick the .asm file, set Content = yes, Item Type = Microsoft Macro Assembler
turn off safeseh (Configuration Properties -> Linker -> Advanced -> Image Has Safe Exception Handlers)
General->Output Directory -> $(SolutionDir)\tests\Output\{OS}}\{Category}\{Test name}\
General->Output Directory -> $(SolutionDir)\tests\Output\win\basic\ConsolePrint\
Set json test specs to Content: Yes, and Item Type: Copy File
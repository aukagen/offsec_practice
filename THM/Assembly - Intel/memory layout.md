Segments
===
**stack** - non-static local variables
**heap** - dynamically allocated data
(can be uninitialised initially)
**.data** - global + static data initialised to _non-zero_ values
**.bss** - global + static data initialised to _zero_/_uninitialised_
**.text** - contains code of programme

![[Screenshot 2022-06-15 at 11.03.48.png]]
_Stack_ = area in memory used quickly for data allocation

_Heap_ = similar to stack but dynamic + slower
_Program Image_ = program/executable loaded into memory (windows = Portable Executable(PE))
_TEB_ - Thread Environment Block = info about currently running threads
_PEB_ - Process E B = info about process + loaded modules

![[Screenshot 2022-06-15 at 11.14.27.png]]
![[Screenshot 2022-06-15 at 11.19.19.png]]


DO THIS:
[One of the best ways to learn is to write your own software and reverse it to see what it looks like. I often do this when I want to learn or get familiar with a new topic. It's especially useful when learning new methods of attack or trying exploits. It also makes you a better programmer which is nice for exploit development and more advanced reverse engineering techniques.]
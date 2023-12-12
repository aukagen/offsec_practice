_callee_ = function being called
_caller_ = function making the call
Often written with `__` in front i.e. `__fastcall`

Most Common
===
`syscall` --> I think Linux?
`cdecl`
`stdcall`
`fastcall` --> x64 for Windows
Fastcall
===
- when a function is called RBP is saved so it can be restored
- RAX = integer, bool, char
- MM0X = float, double
- Member functions + 'this' pointer --> RCX
- Caller must _always_ allocate space --> 4 parameters+
- _volatile_ = RAX, RCX, RDX, R8-11, XMM0-5
- _nonvolatile_ = RBX, RBP, RDI, RSI, RSP, R12-15, XMM6-15

Stack Access
===
- parameters are accessed using RSP (or RBP)
- 4 initial parameters always have space reserved on the stack:
	- 32 bytes --> __0x20__ offset
1-4 params = pushed via registers _left to right (-->)_
4+ params = pushed onto stack at offset __RSP+0x20__ _right to left (<--)_

cdecl (c Declaration)
===
- parameters pushed onto stack left to right (backwards)
- RBP saved (for restoring)
- ret val passed via EAX
- _caller cleans the stack_ 
	--> allows for variable # of params
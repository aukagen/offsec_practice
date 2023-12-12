![[Screenshot 2022-06-05 at 09.15.42.png]]

by reading the script I found the directory which contained the wasm file. From this I got the encoded string +�n�Ȳ�A����3�ŕ�7ß��?��2����e��6ȕ��A� which I believe is the flag - from its position and my experience with the previous two challenges.

However it appears to be utf encoded as well as encrypted in some form which I am not certain of. Step one is therefore to figure out how to remove the unicde encoding.

decoding it with ISO8859-1, which was the indicated encoding format when trying to open it with mousepad, I got the following string for what I believe might be the flag: .A...+.n.È²¹A..Æß3ÀÅ.Þ7Ã..ß?ÉÃÂ.2..Á.e..Â.6È.À..A«...ñ§ð.í

Reading the wasm code in firefox debugger, there are three distinct functions: strcmp and check_flag are as expected to check if the flag is correct, but there is also a copy_char function. From the name I assume it copies some characters (duh) but not sure what it does
I got a copy of the wabt from git (can also be installed with apt though I think) and then I used this to './wasm-decompile ~/Documents/Pico/web/someAssemblyRequired3/qCCYI0ajpD
from this, I could understand the check_flag function better, at least from a first look


 I have learnt that for wasm challenges, I need to find the wasm file/code from the website, then I must decompile it -- in order to better understand what it does. Then I must simply reverse engineer it and create a script to do what it has done in order to get the flag.
for this challenge #3 I almost got the functionality right, just forgot that it was about indexing the given variables from hex. See test.py for the solution:)


# tool of the trade = wabt
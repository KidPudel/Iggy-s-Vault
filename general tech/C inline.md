In C inline function is one for which compiler **copies the code from the function definition directly into the code** of the calling function **instead of creating a separate set of instructions in memory**.
**Instead of transferring control to and from the function code segments**, function body can be directly substituted for the function call.
![[Pasted image 20240909144047.png]]This makes the call stack smaller, but binary bigger
Clean and not too brut-force way to architect easy to understand and scale project, that could be any type, distributed or cli tools.
Obviously, think before you write. 
Organize different parts of applications into its own components.
If it is a logical components into its own services (i mean type XServer, or RedisWrapperService, etc)
Utilize go's great strength of [[goroutines]] in a smart and clean way. Think of the optimizations, and centralizations, so here you can use errgroup, [[event bus]], echo framework.
Lay at the foundation the extensibility opportunity with [[interface]].
If project grows and it is business logic heavy, with different versions of components, L from [[SOLID]] is the way to go. and there you already can implement light-weight version of [[clean architecture]].
Write tests for your features and components, this will make it more stable in a long run.

Keep pushing and practicing, this makes it more natural.


Additionally
look at [[must-have packages aka pack of productive engineer]]
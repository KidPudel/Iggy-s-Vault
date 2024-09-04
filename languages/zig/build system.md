The Zig build system allows to build a project by writing logic in a build.zig file, using the Zig Build System API to declare and configure [[build artifacts]] and other tasks, which allows for cross-platform dependency-free way to declare the logic required to build a project.
***Zig build system, like most build systems is based on modeling the project as a directed acyclic graph ([[DAG]]) of steps, which are independently and concurrently run.***

Build system can help with:
- Making building process in parallel and caching the results
- Depending on other projects
- Providing package for other projects to depend on
- Creating a [[build artifacts]] by executing the Zig compiler (including building from Zig source code and from C, C++ source code).
- Capturing and using user-configured options for the build
- Caching artifacts to avoid repeating steps
- Executing [[build artifacts]] or system-installed tools
- Running tests and verifying the output of executing a [[build artifacts]] matches the expected values
- Running `zig fmt` on a codebase or a subset of it
- Custom tasks


> To use a build system, run `zig build --help`, this will include project-specific options that were declared in the build.zig script.


---

> **NOTE:** the build script may seem [[imperative]], but it is [[declarative]], on a way of constructing a build graph that will be executed by an external runner. Meaning even though it feels like imperative, we just declare the end result, that we want.

## Simple executable

src/main.zig
```rust
const std = @import("std");

pub fn main() !void {
	std.debug.print("Hello World!\n", .{});
}
```

This build script creates an executable from Zig file containing public main function definition
```rust
const std = @import("std");

pub fn build(b: *std.Build) void {
	const exe = b.addExecutable(.{
		.name = "hello",
		.root_source_file = b.path("src/main.zig"),
		.target = b.host,
	});
	// adds artifact to the dependecies to be processed by default (install) step
	b.installArtifacts(exe);
}
```

## Build flow

By default, main step in the graph is the **install** step, whose purpose is to **copy build artifacts into their final resting place**.
Default install step starts with no dependencies, and therefore nothing would happen if run `zig build`, since there is no artifacts to operate on.
This is the job of project's build script to add to the set of things to install, which is what the `installArtifacts` function does.


## Build result
After build, there are two directories generated:
- `zig-cache`: to make subsequent builds faster
- `zig-out`: is a [[installation prefix]], so it contains artifacts like `bin` which is an [[executable binaries]], or lib that contains libraries 



## Customizations
We can add Run step to provide a way to run one's main application directly from the build command

```rust
...
b.installArtifact(exe);
const run_exe = b.addRunArtifact(exe);
const run_step = b.step("run", "Rub the application");
run_step.dependOn(&exe.step);
```




# The basics

## Custom options

Define new options for the user to use with `b.option()`
```rust
pub fn build(b: *std.Build) void {
	const windows = b.option(bool, "windows", "Target Microsoft Windows") orelse false;
	const exe = b.addExecutable(.{
		.name = "hello",
		.root_source_file = b.path("src/main.zig"),
		.target = b.resolveTargetQuery(.{
			.os_tag = if (windows) .windows else null,
		}),
	});
	b.installArtifact(exe);
}
```

and in option list when calling `zig build --help` we can find:
```
Project-Specific Options:
-Dwindows=[bool] Target Microsoft Windows
```


But since most of the projects wants to provide the ability to change the target and optimization framework, zig conveniently provides the helper functions, `standardTargetOptions` and `stadardOptimizeOptions`, including them in build script, will allow users to choose those options

```rust
const target = b.standardTargetOptions(.{})
const optimize = b.standardOptimizeOptions(.{})
const exe = b.addExecutable(.{
	.name = "hello",
	.root_source_file = "src/main.zig",
	.target = target,
	.optimize = optimize,
})
```


## We can expose options inside our code

```rust
const version = b.option([]const u8, "version", "application version string") orelse "0.0.0"; const enable_foo = detectWhetherToEnableLibFoo(); const options = b.addOptions(); options.addOption([]const u8, "version", version);
options.addOption(bool, "have_libfoo", enable_foo);

exe.root_module.addOptions("config", options);

b.installArtifact(exe);
```

```rust
const config = @import("config");

pub fn main() void {
	std.debug.print("version: {s}\n", .{config.version});
	if (config.have_libfoo) {
		foo_bar();
	}
}
```


## [[static library]]

expose the function to be a part of an api by [[exporting]] it
math.zig
```rust
export fn add(a: i32, b: i32) i32 {
	return a + b;
}
```

utilize external library
```rust
extern fn add(a: i32, b: i32) i32;

pub fn main() !void {
	return add(2, 3);
}
```

create/link a [[static library]]
```rust
const std = @import("std");

pub fn build(b: *std.Build) void {
	// define target and optimize...

	const libmath = b.addStaticLibrary(.{
		.name = "math",
		.root_source_file = "path/to/lib/math.zig",
		.target = target,
		.optimize = optimize,
	});
	// this is to add library as well, so that it would be modular
	b.installArtifact(libmath);

	const exe = b.addExecutable(.{
		.name = "hello",
		.root_source_file = "main.zig",
		.target = target,
		.optimize = optimize,
	})

	exe.linkLibrary(libmath);

	b.installArtifact(exe);
}
```

here is what we get as the result
```
zig-out/
├── bin
│  └── demo
└── lib 
	└── libfizzbuzz.a // .a is for static
```

**NOTE:** Installing static library gives as the modular result. 
Meaning we have separate library packaged, so that we can reuse it.
Thats why adding the code base itself as a static library, could be beneficial.
- Modularity: If your library is used by multiple executable, it makes sense to build it as a static or dynamic library.
- Reusability: If you want to distribute the library as a standalone component
- 

> **NOTE:** if we don't install executable artifact, build system is not going to waste any time building the executable, unless specified, because build system is based on [[DAG]] with dependency edges


## [[dynamic library]]
build.zig
```rust
const libmath = b.addSharedLibrary(.{
	.name = "math",
	.root_source_file = "path/to/lib/math.zig",
	.target = target,
	.optimize = optimize,
});
b.installArtifact(libmath);

// to embed information about library
exe.linkLibrary(libmath);
```

```
zig-out 
└── lib 
	├── libfizzbuzz.so -> libfizzbuzz.so.1 
	├── libfizzbuzz.so.1 -> libfizzbuzz.so.1.2.3 
	└── libfizzbuzz.so.1.2.3
```

calling `linkLibrary` in build script in case of dynamic library ensures that Zig compiler and [[linker]] can **check that the executable correctly references the shared library**, 
and so that dynamic linker **could use the embedded information in the executable to find and load the shared library at runtime**.
This information includes library's name and version, so the correct library is loaded.



## Getting and adding external dependencies

Previously what we've talked is great, but it is for organizing project's **local code** into separate libraries, and managing them as libraries.
But if we want to manage **external or third-party** dependencies, we need to tell it to the Zig build system.

But first of all, how we even get external dependencies?
```zsh
zig fetch --save <some-url-for-download>
// like https://github.com/Not-Nik/raylib-zig/archive/devel.tar.gz

```

> This field is optional.
> Each dependency must either provide a `url` and `hash`, or a `path`.
> `zig build --fetch` can be used to fetch all dependencies of a package, recursively.
> Once all dependencies are fetched, `zig build` no longer requires
> internet connectivity.


And now we need to **tell the build system that we depend on it**.
This done with `b.dependency(.{})`, this tells that our project depends on external package or library, and you want Zig to handle the details of locating, configuring, and integrating that dependency into you build process!

```rust
pub fn build(b: *std.Build) void {
	const raylib_dep = b.dependency("raylib-zig", .{
		.target = target,
        .optimize = optimize,
	});
	
	// raylib C library
	const raylib_artifact = raylib_dep.artifact("raylib");
	exe.linkLibrary(raylib_artifact);
	
	// get main module from raylib
	const raylib = raylib_dep.module("raylib");
	exe.root_module.addImport("raylib-zig", raylib);

}
```
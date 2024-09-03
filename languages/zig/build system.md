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
- `zig-out`: is a [[installation prefix]]



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
```rust
export fn add(a: i32, b: i32) i32 {
	return a + b;
}
```


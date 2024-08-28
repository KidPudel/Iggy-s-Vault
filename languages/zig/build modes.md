- `Debug`: Optimizations off and safety on (default)
- `ReleaseFast`: Optimizations on and safety off
- `ReleaseSafe`: Optimizations on and safety on
- `ReleaseSmall`: Size optimizations on and safety off

### [Debug](https://ziglang.org/documentation/master/#toc-Debug) [](https://ziglang.org/documentation/master/#Debug)

`$ zig build-exe example.zig`
- Fast compilation speed
- Safety checks enabled
- Slow runtime performance
- Large binary size
- No reproducible build requirement

### [ReleaseFast](https://ziglang.org/documentation/master/#toc-ReleaseFast) [](https://ziglang.org/documentation/master/#ReleaseFast)

`$ zig build-exe example.zig -O ReleaseFast`
- Fast runtime performance
- Safety checks disabled
- Slow compilation speed
- Large binary size
- Reproducible build

### [ReleaseSafe](https://ziglang.org/documentation/master/#toc-ReleaseSafe) [](https://ziglang.org/documentation/master/#ReleaseSafe)

`$ zig build-exe example.zig -O ReleaseSafe`
- Medium runtime performance
- Safety checks enabled
- Slow compilation speed
- Large binary size
- Reproducible build

### [ReleaseSmall](https://ziglang.org/documentation/master/#toc-ReleaseSmall) [](https://ziglang.org/documentation/master/#ReleaseSmall)

`$ zig build-exe example.zig -O ReleaseSmall`
- Medium runtime performance
- Safety checks disabled
- Slow compilation speed
- Small binary size
- Reproducible build
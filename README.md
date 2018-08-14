# raylib-rs

raylib-rs is a simple, minimal Rust binding for [raylib](http://www.raylib.com/) 2.0. It currently targets the *stable* Rust toolchain, version 1.26 or higher.

Do note that this binding is not fully idiomatic:
- Much like in the C version of raylib, you must handle the loading and unloading of resources manually. Higher-level facilities such as RAII or other resource management methods are left as an exercise to the reader/coder.
- The library structs all derive `Copy`, which means they all have copy semantics like in C. Also like in C raylib, this does *not* mean that pointed-to resources are copied.
- It is more or less a 1:1 mapping of raylib functions to C functions, so there are functions that seem like they can be `impl`ed on a struct, but aren't. This may change in the future, but currently this is intentionally kept simple to keep familiarity with the C API.

**Disclaimer: I created this binding as a way to learn Rust. There may be some things I can do better, or make more ergonomic for users. Feel free to make suggestions!**

# Installation

So far, I have only tested on Windows. Tips on making things work smoothly on all platforms is appreciated.

(the git path below is temporary until raylib-rs is published to crates.io)

1. Add the dependency to your `Cargo.toml`:
```toml
[dependencies]
raylib-rs = { git = "https://github.com/deltaphc/raylib-rs" }
```

2. Download raylib 2.0 from https://github.com/raysan5/raylib/releases/tag/2.0.0, and pick the one that matches your Rust toolchain. MSVC with MSVC, MinGW with GNU, 32-bit or 64-bit.

3. Copy `libraylib.a` (for MinGW) or `raylib.lib` (for MSVC) to the appropriate path in your Rust toolchain.
   - For rustup/MSVC: `.rustup\toolchains\stable-x86_64-pc-windows-msvc\lib\rustlib\x86_64-pc-windows-msvc\lib`
   - For rustup/GNU: `.rustup\toolchains\stable-x86_64-pc-windows-gnu\lib\rustlib\x86_64-pc-windows-gnu\lib`

4. Start coding!
```rust
extern crate raylib_rs;
use raylib_rs as ray; // or raylib_rs::* if you prefer

fn main() {
    ray::init_window(640, 480, "raylib-rs");

    while !ray::window_should_close() {
        ray::begin_drawing();

        ray::clear_background(ray::WHITE);
        ray::draw_text("Hello, world!", 12, 12, 20, ray::BLACK);

        ray::end_drawing();
    }

    ray::close_window();
}
```

# Tech Notes and Differences from C raylib

- Covers nearly the entire raylib 2.0 API. The only omissions are `SubText` and `FormatText`, which are covered by Rust's string slicing and Rust's `format!` macro, respectively.
- Functions dealing with string data take in `&str` and/or return an owned `String`, for the sake of safety.
- In C, `LoadFontData` returns a pointer to a heap-allocated array of `CharInfo` structs. In this Rust binding, said array is copied into an owned `Vec<CharInfo>`, the original data is freed, and the owned Vec is returned.
- A `Font::from_data` method was added to create a `Font` from loaded `CharInfo` data.
- In C, `GetDroppedFiles` returns a pointer to an array of strings owned by raylib. Again, for safety and also ease of use, this binding copies said array into a `Vec<String>` which is returned to the caller.
- `TraceLog` functionality is actually reimplemented in Rust since the language does not have variadic functions in safe code, nor a way to unpack sequences into arguments (to my knowledge). However, since `format!` is variadic, one could use that macro in combination with `trace_log` to achieve the same effect.
- The raw FFI binding was initially generated by `bindgen` and then hand-tweaked a bit.
- I've tried to make linking automatic, though I've only tested on Windows. Other platforms may have other considerations.

# Extras

- In addition to the base library, there is also a convenient `ease` module which contains various interpolation/easing functions ported from raylib's `easings.h`, as well as a `Tween` struct to assist in using these functions.
- Equivalent math and vector operations, ported from `raymath.h`, are `impl`ed on the various Vector and Matrix types. Operator overloading is used for more intuitive design.

# Future Goals

- Port raylib examples over to Rust.
- Perhaps use Rust enums instead of consts.
- More tests.
- More platform testing.
- Even more testing.
- Physac port?

# crystal-wasm-tools
Tools for compiling Crystal dependencies, bindings, etc. for use with WASM

I'm currently working on targeting wasm32-wasi for Crystal. This keeps the focus
rather narrow on things that are supported w/ roughly mainline LLVM. Initially,
I focused on emscripten, but decided I didn't want to invest the effort getting
super into the Emscripten internals, tools, etc.

## Setting up prerequisites

0. Make sure you've installed autoconf, automake, build-essential, etc.
1. `mkdir -p ~/toolchains/crystal-wasm-libs/targets/wasm32-wasi`
2. Download [latest release of wasi-sdk](https://github.com/WebAssembly/wasi-sdk/releases)
3. Extract to `~/toolchains`. The resulting directory shoud be something like `~/toolchains/wasi-sdk-10.0`
4. Run (and/or add to your shell): `export WASI_SDK_PATH="$HOME/toolchains/wasi-sdk-10.0`
5. Install [WAVM](https://github.com/WAVM/WAVM)

### Compiling libpcre for wasm

I have a copy of libpcre w/ some minor changes to support building for WASI. The
PR is here: https://github.com/maxfierke/libpcre/pull/1

For building Crystal programs for wasm32-wasi, we'll need a compiled version of
libpcre for the platform. Luckily, libpcre is super easy to compile for wasm32-wasi.

1. Clone my fork of libpcre:
   ```sh
   git clone git@github.com:maxfierke/libpcre.git -b mf-wasm32-wasi-cross-compile
   ```
2. `cd libpcre`
3. Run `./build_for_crystal.sh`. This should compile successfully and place the
   compiled `.a` static library in `targets/wasm32-wasi`
4. `cp targets/wasm32-wasi/*.a ~/toolchains/crystal-wasm-libs/targets/wasm32-wasi`

## Compiling Crystal for wasm

I have a fork of crystal with a small-level of support for wasm32-wasi. It can
compile for it, and simple binaries can execute in WAVM.

0. Follow usual instructions for [installing Crystal on your platform](https://crystal-lang.org/install/). You may need
   to also [install some additional packages](https://github.com/crystal-lang/crystal/wiki/All-required-libraries)
   for your platform.
1. Run `git clone git@github.com:maxfierke/crystal.git -b mf-spike_wasm_take_two crystal-wasm`
2. `cd crystal-wasm`
3. `make clean && make` to compile the compiler
4. Run `spec/run_wasm_spec.sh` to run the full spec suite

You can use `spec/run_wasm_spec.sh` as a basis for how to cross-compile programs
for wasm.

## Caveats

* This is _very_, _very_ rough. Many APIs are not supported at all (e.g.
  most IO, signals, sockets, Process, GC, concurrency, OpenSSL, Zlib, Big, etc.)
* There is **no support for exceptions**. This is mostly due to a limitation
  WebAssembly itself. The exception handling proposal is still in development.
  Some runtimes have varying levels of support for it, but the codegen from LLVM
  requires LLVM master, isn't supported in wasi-sdk, etc. As a result, any code
  that throws exceptions will crash (if it throws). This alone _severely_ limits
  use of Crystal on wasm, as a lot of code relies on exception handling, including
  parts of the stdlib.
* There are [strange require order dependencies I don't understand](https://github.com/maxfierke/crystal/commit/56a6a148853ef26f52c0ebfd7cfe08887ec9e88f)

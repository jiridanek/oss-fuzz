# Fuzz testing for qpid-proton-c

## Dockerfile

Easiest way to build and run the fuzzing is using attached Dockerfile. Run the following command from the top project directory

    docker build -t -f .

## Building

### with libFuzzer

1. Download and compile libFuzzer. Use http://llvm.org/docs/LibFuzzer.html for detailed instructions.
2. Rename libFuzzer.a (from previous step) to `libFuzzingEngine.a`
3. Build qpid-proton with the following configuration
  * set `CC` and `CXX` variables to the same compiler you used to build libFuzzer (some recent clang)
  * set `CFLAGS` with the coverage and sanitizer(s) you want to use, see libFuzzer documentation for details
  * set `LDFLAGS` to add the directory with `libFuzzingEngine.a` to your link path if necessary
  * set `FUZZ_TEST=ON`

For example:

    CC=~/Work/third_party/llvm-build/Release+Asserts/bin/clang \
    CXX=~/Work/third_party/llvm-build/Release+Asserts/bin/clang++ \
    CFLAGS="-fsanitize-coverage=trace-pc-guard -fsanitize=address -fno-omit-frame-pointer -fno-optimize-sibling-calls" \
    LDFLAGS="-L/path/to/LibFuzzingEngine/directory" \
    cmake -DCMAKE_BUILD_TYPE=Debug -DFUZZ_TEST=ON ..

### with afl-fuzz

TODO


## Running

Execute one of the `fuzz-*` binaries. When given a folder as first argument, it will load corpus from the folder and also store newly discovered inputs (that extend coverage) there. See http://llvm.org/docs/LibFuzzer.html for details.

# Justifications

The reason for renaming the fuzzing library to `libFuzzingEngine` is to be compatible with https://github.com/google/oss-fuzz.

In fuzzing data, corpus and crashes are two different directories because fuzzers can perform corpus minimization. Crashes should not be subjected to corpus minimization, so they need to be kept separately.

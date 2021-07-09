# How to build LLVM for OSS-Fuzz

> Disclaimer: these commands are ocumented here so I don't forget them. Not intented to be used by anyone.

Build dependencies in the Dockerfile:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends cmake ninja-build
```

Build and install everything:

```bash
(
cd $SRC/llvm-project && mkdir build_x64_opt && cd build_x64_opt

if [ "$SANITIZER" = "address" ]
then
  LLVM_USE_SANITIZER=Address
else
  LLVM_USE_SANITIZER=""
fi

cmake \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_USE_SANITIZER=$LLVM_USE_SANITIZER \
    -DLLVM_ENABLE_PROJECTS="compiler-rt;clang;lld;lldb" \
    -DLLVM_USE_LINKER=lld \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DLLVM_BUILD_TOOLS=OFF \
    -DLLVM_BUILD_UTILS=OFF \
    -DLLVM_BUILD_TESTS=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
    -DLLDB_ENABLE_PYTHON=0 \
    -DLLDB_INCLUDE_TESTS=OFF \
    -DCLANG_ENABLE_ARCMT=OFF \
    -DCLANG_ENABLE_STATIC_ANALYZER=OFF \
    -DCMAKE_INSTALL_PREFIX="$SRC/llvm" \
    -GNinja \
    ../llvm

ninja \
    install-clang \
    install-clang-headers \
    install-clang-libraries \
    install-liblldb \
    install-lld \
    install-lldb-headers \
    install-lldb-server \
    install-llvm-headers \
    install-llvm-libraries
)

cp -r $SRC/llvm-project $OUT/llvm-out
```

Copy all the generated files, we'll need them later for `coverage`:

```bash
find oss-fuzz/build/out/lldb-eval/llvm-out/llvm-project/build_x64_opt -type f -regex '.*\.\(c\|cc\|cpp\|h\|inc\|def\)' | cpio -updm llvm-build-genfiles
```

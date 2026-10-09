# hrx-windows-build
Git diff files to patch and build llama.cpp AMD HRX on Windows

# Steps
1. Clone each repo
2. Apply changes to each repo (rocm-systems gives you hsa-runtime64.dll, hrx-system gives you hrx.dll and loomc.dll, llama.cpp gives you ggml-hrx.dll, llama-server.exe, etc.)
3. Set build time environment variables for CMake/Ninja
4. Configure using CMake
5. Build using Ninja
6. Install using CMake if you want to save it outside of build directory
7. Set the runtime environment variables such as HSA_USE_SVM=1 to fully load hsa-runtime64.dll, and HSA_OVERRIDE_GFX_VERSION="11.5.1" to enable the path to the RDNA3.5 ML Math kernel corpus.
8. Make sure all the relevant binaries and dlls are in System or User PATH (i.e. llama-server.exe, llama-bench.exe, hsa-runtime64.dll, hrx.dll, loomc.dll, libomp.dll, etc.)

File1: rocm-systems.diff
Command: `git clone https://github.com/ROCm/rocm-systems --branch=develop`
Command: `cd rocm-systems`
Command: `git apply ../hrx-windows-build/rocm-systems.diff`

File2: hrx-system.diff
Command: `git clone https://github.com/ROCm/hrx-system --branch=main`
Command: `cd hrx-system`
Command: `git apply ../hrx-windows-build/hrx-system.diff`

File3: llamacpp.diff
Command: `git clone https://github.com/AMD-Ecosystem/llama.cpp --branch=hrx-graph-develop-v2`
Command: `cd llama.cpp`
Command: `git apply ../hrx-windows-build/llamacpp.diff`

# PowerShell Build Time Environment Variables
$env:GPU_TARGETS="gfx1152"
$env:GPU_FAMILIES="gfx1152"
$env:HAL_GPU_TARGETS=$env:GPU_TARGETS
$env:LOOM_GPU_TARGETS=$env:GPU_TARGETS
$env:HRX_ROCM_ROOT=$env:HIP_PATH
$env:HIP_DEVICE_LIB_PATH="$env:HIP_PATH/lib/llvm/amdgcn/bitcode"
$env:ROCM_DEVICE_LIB_PATH="$env:HIP_DEVICE_LIB_PATH"
$env:AMD_ICD_LIBRARY_DIR="$env:HIP_PATH/bin"

# Example ROCM-Systems Cmake Command:
cmake -B build -G Ninja `
    -D CMAKE_PREFIX_PATH="$TheRock/base/rocm-cmake;$EXTRA_PREFIX_PATHS" `
    -D CMAKE_C_COMPILER="clang-cl.exe" `
    -D CMAKE_CXX_COMPILER="clang-cl.exe" `
    -D CMAKE_LINKER="lld-link.exe" `
    -D THEROCK_ENABLE_BASE=ON `
    -D THEROCK_ENABLE_CORE=ON `
    -D THEROCK_FLAG_HSA_WINDOWS_SHARED_RUNTIME=ON `
    -D GPU_TARGETS=$env:GPU_TARGETS `
    -D AMDGPU_TARGETS=$env:GPU_TARGETS `
    -D THEROCK_AMDGPU_TARGETS=$env:GPU_TARGETS `
    -D THEROCK_TEST_AMDGPU_TARGETS=$env:GPU_TARGETS `
    -D GPU_ARCHS=$env:GPU_TARGETS `
    -D THEROCK_DIST_AMDGPU_FAMILIES=$env:GPU_FAMILIES `
    -D THEROCK_TEST_AMDGPU_FAMILIES=$env:GPU_FAMILIES `
    -D THEROCK_AMDGPU_DIST_BUNDLE_NAME=$env:GPU_FAMILIES `
    -D CMAKE_HIP_ARCHITECTURES=$env:GPU_TARGETS

# Ninja Command
ninja -C build -j8

# CMake Install Command
cmake --install build --prefix /path/to/destination/rocm-systems

# Example HRX-System CMake Command
cmake -B build -G Ninja `
    -D CMAKE_PREFIX_PATH="$TheRock/base/rocm-cmake;$env:HIP_PATH/lib/cmake/hip;$EXTRA_PREFIX_PATHS" `
    -D CMAKE_C_COMPILER="clang-cl.exe" `
    -D CMAKE_CXX_COMPILER="clang-cl.exe" `
    -D CMAKE_LINKER="lld-link.exe" `
    -D IREE_HAL_AMDGPU_TARGETS=$env:GPU_TARGETS `
    -D IREE_HAL_AMDGPU_DEVICE_TOOLCHAIN=rocm `
    -D IREE_HAL_AMDGPU_DEVICE_TOOLCHAIN_ROCM_PATH=$env:HIP_PATH `
    -D IREE_HAL_AMDGPU_DEVICE_BINARY_BUILD_MODE=source `
    -D AMDF_BUILD=ON `
    -D LIBHRX_BUILD_HIP_BINDING=ON `
    -D LIBHRX_BUILD_CTS=ON `
    -D LOOM_TARGET_ARCH_AMDGPU=ON `
    -D LOOM_TARGET_ARCH_XDNA=ON `
    -D LOOM_TARGET_ARCH_SPIRV=ON `
    -D LOOM_TARGET_AMDGPU_TARGETS=$env:LOOM_GPU_TARGETS `
    -D LOOM_EMIT_AMDGPU=ON `
    -D LOOM_EMIT_XDNA=ON `
    -D LOOM_EMIT_SPIRV=ON `
    -D IREE_HAL_DRIVER_AMDGPU=ON `
    -D IREE_HAL_DRIVER_VULKAN=ON `
    -D IREE_ENABLE_WERROR_FLAG=OFF `
    -D IREE_BUILD_TESTS=ON `
    -D IREE_BUILD_BENCHMARKS=ON `
    -D BUILD_SHARED_LIBS=ON

# Ninja Command
ninja -C build -j8

# CMake Install Command
cmake --install build --prefix /path/to/destination/hrx-system

# Example llama.cpp CMake Command
cmake -B build -G Ninja `
    -D CMAKE_C_COMPILER="clang-cl.exe" `
    -D CMAKE_CXX_COMPILER="clang-cl.exe" `
    -D CMAKE_LINKER="lld-link.exe" `
    -D GGML_NATIVE=ON `
    -D GGML_HRX=ON `
    -D GGML_VULKAN=ON `
    -D BUILD_SHARED_LIBS=ON `
    -D BUILD_TESTING=ON `
    -D BUILD_TESTS=ON `
    -D LLAMA_BUILD_TESTS=OFF `
    -D GGML_BUILD_EXAMPLES=OFF `
    -D ZLIB_BUILD_EXAMPLES=OFF `
    -D GGML_ALL_WARNINGS=OFF `
    -D GPU_TARGETS=$env:GPU_TARGETS `
    -D GPU_ARCHS=$env:GPU_TARGETS `
    -D CMAKE_HIP_ARCHITECTURES=$env:GPU_TARGETS

# Ninja Command
ninja -C build -j8

# CMake Install Command
cmake --install build --prefix /path/to/destination/llamacpp

# PowerShell Run Time Environment Variables
$env:HSA_USE_SVM=1
$env:HSA_OVERRIDE_GFX_VERSION="11.5.1"

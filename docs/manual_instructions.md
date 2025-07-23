# Manual Build Instructions

> **⚠️ Important Notice**
> 
> These manual build instructions are provided for reference purposes only and may not reflect the most current build process. We cannot guarantee that these instructions are up-to-date or will work in all environments. 
> 
> For the most reliable and current build process, please refer to our [automated GitHub Actions workflow](../.github/workflows/build-llamacpp-rocm.yml). The workflow represents our recommended approach for building Llama.cpp with ROCm support.

---

If you prefer to build locally, follow these steps:

## Part 1: Download required software

I used chocolatey, but you can also install those manually.
  ```
  choco install visualstudio2022buildtools -y --params "--add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.VC.CMake.Project --add Microsoft.VisualStudio.Component.VC.ATL --add Microsoft.VisualStudio.Component.Windows11SDK.22621"
  choco install cmake --version=3.31.0 -y
  choco install ninja -y
  choco install ccache -y
  choco install python -y
  choco install strawberryperl -y
  ```
> Note: cmake is not strictly needed, as we 

## Part 2: Organizing artifacts
* Step 1: Get the latest run id from main [here](https://github.com/ROCm/TheRock/actions/workflows/release_windows_packages.yml).
  * Example: [TheRock/actions/runs/16218534118/job/45793425858](https://github.com/ROCm/TheRock/actions/runs/16218534118/job/45793425858)
* Step 2:
Look at the upload logs for `gfx1151`. This is what I see:
  ```
  ://therock-nightly-tarball/therock-dist-windows-gfx1151-7.0.0rc20250711.tar.gz
  ://therock-nightly-python/v2/gfx1151/rocm-7.0.0rc20250711.tar.gz
  ://therock-nightly-python/v2/gfx1151/rocm_sdk_libraries_gfx1151-7.0.0rc20250711-py3-none-win_amd64.whl
  ://therock-nightly-python/v2/gfx1151/rocm_sdk_devel-7.0.0rc20250711-py3-none-win_amd64.whl
  ://therock-nightly-python/v2/gfx1151/rocm_sdk_core-7.0.0rc20250711-py3-none-win_amd64.whl
  ```
* Step 4: Download the nightly tarball 
  * Example: [therock-nightly-tarball.s3.amazonaws.com/YOUR_FILE](https://therock-nightly-tarball.s3.amazonaws.com/therock-dist-windows-gfx1151-7.0.0rc20250711.tar.gz)
* Step 5: Extract the contents of this tar.gz file to `C:\opt\rocm`
* Setp 6: Add `C:\opt\rocm\lib\llvm\bin` to path
* Step 7: clone llamacpp

## Part 3: Updating llama.cpp

Open `C:\<YOUR_LLAMACPP_PATH>\ggml\src\ggml-cuda\vendors\hip.h` and replace `HIP_VERSION >= 70000000` with `HIP_VERSION >= 50600000`

## Part 4: Building Llama.cpp + ROCm

Open `x64 Native Tools Command Prompt` and run the following commands:

```
set HIP_PATH=C:/opt/rocm
set PATH=%HIP_PATH%/bin;%PATH%
set HIP_PLATFORM=amd
cd "C:\<YOUR_LLAMACPP_PATH>\llama.cpp"
mkdir build
cd build
cmake .. -G Ninja -DCMAKE_C_COMPILER="C:\opt\rocm\lib\llvm\bin\clang.exe" -DCMAKE_CXX_COMPILER="C:\opt\rocm\lib\llvm\bin\clang++.exe" -DCMAKE_CROSSCOMPILING=ON -DCMAKE_BUILD_TYPE=Release -DAMDGPU_TARGETS="gfx1151" -DBUILD_SHARED_LIBS=ON -DLLAMA_BUILD_TESTS=OFF -DGGML_HIP=ON -DGGML_OPENMP=OFF -DGGML_CUDA_FORCE_CUBLAS=OFF -DGGML_HIP_ROCWMMA_FATTN=OFF -DGGML_HIP_FORCE_ROCWMMA_FATTN_GFX12=OFF -DLLAMA_CURL=OFF -DGGML_NATIVE=OFF -DGGML_STATIC=OFF -DCMAKE_SYSTEM_NAME=Windows
cmake --build . -j 24 2>&1 | findstr /i "error"
```

If you see no errors, that means that llama.cpp has correctly been built and files are available inside your `build\bin` folder. 
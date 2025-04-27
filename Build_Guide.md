# Guide: Building llama-cpp-python (v0.3.8) Wheel with CUDA 12.8 for Windows (x64) that supports GPU offloading for Google's SOTA Gemma 3 Models

## This guide details a specific process used to build a working llama-cpp-python (v0.3.8) Python wheel with Gemma 3 support with NVIDIA CUDA 12.8 acceleration on Windows 10/11 (x64). This build was verified to offload model layers to the GPU using llama-cpp-python running Gemma 3 models. 

**Note:** This guide documents a process that proved successful after troubleshooting. It involves obtaining the Python binding source from PyPI, relies on a separately pre-built llama.cpp CUDA DLL, and utilizes specific environment variable settings. Your paths and exact versions may vary.

## Prerequisites
**Before starting, ensure you have the following installed:**

**Visual Studio 2022 (Community Edition or higher):** Make sure to include the "Desktop development with C++" workload during installation.

**NVIDIA CUDA Toolkit 12.8:** Install the CUDA Toolkit version 12.8. Ensure the installation successfully adds CUDA to your system's PATH.

**NVIDIA Drivers:** Ensure you have NVIDIA drivers installed that are compatible with CUDA 12.8.

**Visual C++ Redistributable for Visual Studio 2015-2022:** Download and install the latest version from Microsoft.

**Python 3.8 or higher (x64):** Recommended to use Python 3.11 as the wheel is tagged for cp311. Ensure you have pip installed.

**Python Virtual Environment:** Highly recommended for managing dependencies.

**Git (Optional but Recommended):** Useful for managing source code.

**An up to date Pre-built llama.cpp CUDA 12.8 DLL:** This guide relies on having a working llama.cpp shared library file (llama.dll) compiled with CUDA 12.8 support obtained from the official [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp/releases/) repository. This specific build process used a DLL located at **[Path to your prebuilt llama.cpp CUDA DLL]**. You will need to adapt this path to your specific file location.

## Step 1: Get the llama-cpp-python Source Code

Obtain the source distribution (.tar.gz) for llama-cpp-python version 0.3.8 from PyPI.

Create a directory to work in, e.g., [Path to your source directory].

Extract the contents of the downloaded .tar.gz file into this directory.

## Step 2: The Importance of Environment Variables

Building complex software with native code dependencies and external SDKs like NVIDIA CUDA heavily relies on environment variables. These variables act as crucial signposts and configuration switches for build tools (like compilers, linkers, and CMake).

**Discovering Dependencies:** Environment variables (such as PATH, INCLUDE, LIB, and specific SDK variables like CUDA_PATH, CUDA_PATH_V12_8, CUDA_NVPCC_PATH, CUDA_NVVM_LIB_PATH) tell the build system where to find necessary header files (.h), library files (.lib, .dll), and executable tools (cl.exe, link.exe, nvcc.exe) required to compile and link the software.

**Resolving Ambiguity:** On systems with multiple versions of Visual Studio or CUDA, explicit environment variables ensure the build system uses the correct and compatible versions of tools and libraries, preventing conflicts and errors.

**Controlling Build Behavior:** Variables like FORCE_CMAKE, CMAKE_ARGS, LLAMA_CPP_LIB, and SCB_LOCAL_VERSION provide direct instructions to the build backend (scikit-build-core) and the underlying build system (CMake), allowing you to:

**1.** Force a specific build system to run (FORCE_CMAKE).

**2.** Pass specific configuration options to CMake (e.g., enabling CUDA via CMAKE_ARGS).

**3.** Specify the path to a pre-built native library to be used instead of building from source (LLAMA_CPP_LIB).

**4.** Inject dynamic information like a local version identifier (SCB_LOCAL_VERSION).

**5.** Reproducibility and Debugging: Carefully documenting the environment variables used in a successful build is vital for others to reproduce it. When builds fail, checking and correcting environment variables is often the first and most critical debugging step.

**6.** Using the "x64 Native Tools Command Prompt" helps set many of the standard Visual Studio and Windows SDK environment variables correctly, providing a solid base before adding project-specific variables.

## Step 3: Set Up the Build Environment

This is a critical step for CUDA builds on Windows.

**1.** Close any standard Command Prompt or PowerShell windows.

**2.** Open the "x64 Native Tools Command Prompt for VS 2022" from your Windows Start Menu. This prompt sets up the necessary environment variables and paths for the Visual Studio C++ build tools and should also help locate CUDA.

**3.** In this Native Tools Command Prompt, navigate to your llama-cpp-python source directory ([Path to your source directory]).

**4.** Activate your Python virtual environment. Use the correct path to your venv:
```
[Path to your venv]\scripts\activate
```
**5.** In this same Native Tools Command Prompt session, set the following environment variables. These guide the scikit-build-core and CMake build process:
```
set FORCE_CMAKE=1
set CMAKE_ARGS=-DGGML_CUDA=on
set LLAMA_CPP_LIB=[Path to your prebuilt llama.cpp CUDA DLL]
```

Note: Ensure **[Path to your prebuilt llama.cpp CUDA DLL]** is the exact path to your CUDA 12.8 llama.dll. 

**6.** (Optional but Recommended) Purge the pip cache to ensure a clean download/build process:
```
pip cache purge
```

## Step 4: Build the Wheel

Ensure you are in the llama-cpp-python source directory ([Path to your source directory]) in the Native Tools Command Prompt with your venv activated and environment variables set.

Run the pip wheel command to build the wheel from the current source directory (.) and save it to a specified output directory:


```
pip wheel --no-cache-dir . -w [Path to your output directory]
```

**Note:** Adjust [Path to your output directory] to your desired folder for the .whl file.

Let the build process complete. This will take several minutes as it compiles native code.

## Step 5: Verify the Created Wheel

Once the pip wheel command finishes successfully, navigate to your output folder ([Path to your output directory]).

You should find the built wheel file named something like llama_cpp_python-0.3.8+[Your Version Tag]-cp311-cp311-win_amd64.whl.

## Step 6: Verify CUDA Functionality

To ensure the built wheel correctly enables CUDA acceleration:

**1.** Ensure you are in a terminal with your Python virtual environment activated.

**2.** Uninstall any existing llama-cpp-python installations: pip uninstall llama-cpp-python.

**3.** Install the wheel file you just built using pip:

```
pip install [Path to your output directory]\llama_cpp_python-0.3.8+[Your Version Tag]-cp311-cp311-win_amd64.whl
```
Note: Adjust the path and filename to your .whl file, including your specific version tag.

**4.** Run the CUDA verification script (ensure you replace the model path):

```
python -c "from llama_cpp import Llama; print('Attempting to initialize Llama with GPU offload...'); try: model = Llama(model_path='[Path to a small .gguf model file]', n_gpu_layers=-1, verbose=True); print('Initialization attempted.
Check output above for GPU layers being offloaded.'); except FileNotFoundError: print('Error: Model file not found at the specified path.'); print('Please ensure the model file exists.'); except Exception as e: print(f'An error occurred during initialization: {e}');"
```
**5.** Look for output messages indicating layers being offloaded to the GPU.

## Troubleshooting Notes
**CMake Error: LLAMA_CUBLAS is deprecated... Use GGML_CUDA instead:**

This means the CMAKE_ARGS environment variable was set with the old flag. 

Correct it to set **CMAKE_ARGS=-DGGML_CUDA=on.** Ensure this is set in the Native Tools Prompt before running pip wheel.

**Build Fails/CPU Fallback:** Ensure you are using the **"x64 Native Tools Command Prompt for VS 2022" command window** Verify all required environment variables **(FORCE_CMAKE, CMAKE_ARGS, LLAMA_CPP_LIB, SCB_LOCAL_VERSION)** are set correctly in that specific session before running pip wheel. 

Ensure your **CUDA Toolkit 12.8** is installed and its paths are correctly in your system's environment (though the Native Tools Prompt helps). 

Using **--no-cache-dir** can also help force a clean build.

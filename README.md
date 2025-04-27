# llama-cpp-python Prebuilt Wheel (Windows x64, CUDA 12.8, Gemma 3 Support)

This repository provides a prebuilt Python wheel (`.whl`) file for **llama-cpp-python**, specifically compiled for Windows 10/11 (x64) with NVIDIA CUDA 12.8 acceleration enabled.

Building `llama-cpp-python` with CUDA support on Windows can be a complex process involving specific Visual Studio configurations, CUDA Toolkit setup, and environment variables. This prebuilt wheel aims to simplify installation for users with compatible systems.

This build is based on **llama-cpp-python** version `0.3.8` of the Python bindings, and the underlying **llama.cpp** source code as of **April 26, 2025**. It has been verified to work with **Gemma 3 models**, correctly offloading layers to the GPU.

---

## Features

- **Prebuilt for Windows x64**: Ready to install using `pip` on 64-bit Windows systems.
- **CUDA 12.8 Accelerated**: Leverages your NVIDIA GPU for faster inference.
- **Gemma 3 Support**: Verified compatibility with Gemma 3 models.
- **Based on llama-cpp-python version `0.3.8` bindings.**
- **Uses [llama.cpp release b5192](https://github.com/ggml-org/llama.cpp/releases/tag/b5192) from April 26, 2025.**

---

## Compatibility & Prerequisites

To use this wheel, you must have:

- An **NVIDIA GPU**.
- NVIDIA drivers compatible with **CUDA 12.8** installed.
- **Windows 10 or Windows 11 (x64)**.
- **Python 3.8 or higher** (the wheel is built specifically for **Python 3.11** (`cp311`)).
- The **Visual C++ Redistributable for Visual Studio 2015-2022** installed.

---

## Installation

It is highly recommended to install this wheel within a Python virtual environment.

1. Ensure you have met all the prerequisites listed above.
2. Create and activate a Python virtual environment:

    ```bash
    python -m venv venv_llama
    .\venv_llama\Scripts\activate
    ```

3. Download the `.whl` file from this repository's **Releases** section.
4. Open your Command Prompt or PowerShell.
5. Navigate to the directory where you downloaded the `.whl` file.
6. Install the wheel using `pip`:

    ```bash
    pip install llama_cpp_python-0.3.8+cu128.gemma3-cp311-cp311-win_amd64.whl
    ```

---

## Verification (Check CUDA Usage)

To verify that `llama-cpp-python` is using your GPU via CUDA after installation:

```bash
python -c "from llama_cpp import Llama; print('Attempting to initialize Llama with GPU offload...'); try: model = Llama(model_path='path/to/a/small/model.gguf', n_gpu_layers=-1, verbose=True); print('Initialization attempted. Check output above for GPU layers.'); except FileNotFoundError: print('Model file not found, but library initialization output above might still indicate CUDA usage.'); except Exception as e: print(f'An error occurred during initialization: {e}');"
```

Note: Replace path/to/a/small/model.gguf with the actual path to a small .gguf model file.

Look for output messages indicating layers being offloaded to the GPU, such as assigned to device CUDA0 or memory buffer reports.

## Alternative Verification: Python Script

If you prefer, you can verify that llama-cpp-python is correctly using CUDA by running a small Python script inside your virtual environment.

Replace the placeholder paths below with your actual .dll and .gguf file locations:

```bash
import os
from llama_cpp import Llama

# Set the environment variable to point to your custom-built llama.dll
os.environ['LLAMA_CPP_LIB'] = r'PATH_TO_YOUR_CUSTOM_LLAMA_DLL'

try:
    print('Attempting to initialize Llama with GPU offload (-1 layers)...')
    
    # Initialize the Llama model with full GPU offloading
    model = Llama(
        model_path=r'PATH_TO_YOUR_MODEL_FILE.gguf',
        n_gpu_layers=-1,
        verbose=True
    )
    
    print('Initialization attempted. Check the output above for CUDA device assignments (e.g., CUDA0, CUDA1).')

except FileNotFoundError:
    print('Error: Model file not found. Please double-check your model_path.')
except Exception as e:
    print(f'An error occurred during initialization: {e}')
```
**What to look for in the output:**

Lines like assigned to device CUDA0, assigned to device CUDA1.

VRAM buffer allocations such as CUDA0 model buffer size = ... MiB.

Confirmation that your GPU(s) are being used for model layer offloading.

## Usage
Once installed and verified, you can use llama-cpp-python in your projects as you normally would. Refer to the official llama-cpp-python documentation for detailed usage instructions.

## Acknowledgments
This prebuilt wheel is based on the excellent llama-cpp-python project by Andrei Betlen (@abetlen). All credit for the core library and Python bindings goes to the original maintainers and to llama.cpp by Georgi Gerganov (@ggerganov).

This specific wheel was built by Bernard Peter Fitzgerald (@boneylizard) using the source code from abetlen/llama-cpp-python, compiled with CUDA 12.8 support for Windows x64 systems, and verified for Gemma 3 model compatibility.

## License
This prebuilt wheel is distributed under the MIT License, the same license as the original llama-cpp-python project.

## Reporting Issues
If you encounter issues specifically with installing this prebuilt wheel or getting CUDA offloading to work using this wheel, please report them on this repository's Issue Tracker.

For general issues with llama-cpp-python itself, please report them upstream at the [official llama-cpp-python GitHub Issues page](https://github.com/ggml-org/llama.cpp/issues).

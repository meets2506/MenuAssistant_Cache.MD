# LLama Android SDK 1.2

## Overview

LLama Android SDK is a powerful implementation of the LLaMA large language model for Android platforms. This SDK enables Android developers to integrate on-device LLM capabilities into their applications, providing text generation, completion, and conversational AI features without requiring internet connectivity or sending sensitive data to external services.

Built on the high-performance llama.cpp library, this SDK delivers native performance with a clean, developer-friendly Kotlin API that makes advanced AI text generation accessible to mobile application developers.

## Features

- **On-device inference**: Run LLaMA language models entirely on Android devices
- **Privacy-focused**: Process text locally without sending data to external servers
- **Optimized for mobile**: Efficient implementation designed for resource-constrained environments
- **Simple API**: Clean Kotlin interface with coroutines and Flow support
- **Highly configurable**: Control context size, thread count, batch size, and memory usage
- **Streaming generation**: Token-by-token text generation using Kotlin Flow
- **Memory management options**: Support for memory-mapped models for improved performance
- **JNI performance**: Native C++ implementation with Java Native Interface for optimal speed
- **Multiple sampling methods**: Temperature and top-p (nucleus) sampling for varied outputs

## Architecture Diagram

```
Android Application
       │
       ▼
┌─────────────────┐
│ LLamaSDK.kt     │◄───┐
│ (High-level API)│    │
└────────┬────────┘    │
         │             │
         ▼             │
┌─────────────────┐    │
│ LLamaAndroid.kt │    │ Kotlin/Java Layer
│ (Native Bridge) │    │
└────────┬────────┘    │
         │             │
         ▼             │
┌─────────────────┐    │
│   JNI Layer     │    │
└────────┬────────┘    │
         │             │
         ▼             │
┌─────────────────┐    │
│ llama-android.cpp│   │
│ (Native Code)   │    │ C++ Layer
└────────┬────────┘    │
         │             │
         ▼             │
┌─────────────────┐    │
│    llama.cpp    │    │
│    (Library)    │────┘
└─────────────────┘
```

## Directory and File Structure

```
llama.android-sdk-1.2/
├── llama/                       # Main SDK module
│   ├── build.gradle.kts         # Gradle build configuration
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── cpp/
│           │   ├── CMakeLists.txt        # CMake build configuration for native code
│           │   ├── llama-android.cpp     # JNI implementation for LLaMA
│           │   └── simple_llama_jni.cpp  # Alternative JNI implementation
│           └── java/
│               └── android/
│                   └── llama/
│                       └── cpp/
│                           ├── LLamaAndroid.kt  # Core native bridge implementation
│                           └── LLamaSDK.kt      # High-level API for applications
├── app/                         # Sample application
├── DISTRIBUTION_GUIDE.md        # Guide for publishing and distributing the SDK
└── README.md                    # This file
```

### Key Components

- **llama-android.cpp**: Core native implementation providing JNI functions for model loading, text generation, and memory management
- **LLamaAndroid.kt**: Kotlin wrapper that interfaces with the native code, handling thread management and resource lifecycle
- **LLamaSDK.kt**: High-level API that simplifies integration for application developers

## How to Build

### Prerequisites

- Android Studio Arctic Fox (2020.3.1) or newer
- CMake 3.18.1+
- NDK 21.4.7075529+
- Kotlin 1.5.0+
- Java 11+
- Minimum SDK version: 33 (Android 13)

### Building the SDK

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/llama.android-sdk.git
   cd llama.android-sdk-1.2
   ```

2. Open the project in Android Studio and build, or use Gradle directly:
   ```bash
   ./gradlew llama:assembleRelease
   ```

3. Find the AAR file in `llama/build/outputs/aar/llama-release.aar`

## How to Use

### Integration

Add the SDK to your project using one of these methods:

#### Method 1: Gradle dependency (recommended)

```kotlin
// In settings.gradle.kts
repositories {
    mavenCentral()
    // Or your private repository
}

// In app/build.gradle.kts
dependencies {
    implementation("com.yourdomain:llama-android-sdk:1.2")
}
```

#### Method 2: Using the AAR directly

1. Place the AAR file in your app's `libs` folder
2. Add to your app's `build.gradle.kts`:
   ```kotlin
   dependencies {
       implementation(files("libs/llama-release.aar"))
   }
   ```

### Basic Usage

```kotlin
import android.llama.cpp.LLamaSDK
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.launch
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers

class MyApplication {
    private val llama = LLamaSDK()
    private val scope = CoroutineScope(Dispatchers.Main)
    
    fun initialize() {
        scope.launch {
            // 1. Load the model
            llama.load(
                pathToModel = "/path/to/model.gguf",
                nCtx = 512,          // Context size
                nThreads = 4,         // Number of threads
                batchSize = 128,      // Batch size
                mlock = false         // Use memory locking
            )
            
            // 2. Generate text
            llama.send("Hello, I am a language model").collect { token ->
                // Process each token as it's generated
                print(token)
            }
            
            // 3. Unload model when done
            llama.unload()
        }
    }
}
```

### Advanced Configuration

The SDK offers several configuration options:

- **Context size** (`nCtx`): Controls the size of the context window (token buffer). Larger values allow more context but require more memory.
- **Thread count** (`nThreads`): Number of CPU threads to use for processing.
- **Batch size**: Controls how many tokens are processed in parallel.
- **Memory locking** (`mlock`): When enabled, locks model weights in memory to prevent paging.

```kotlin
// Advanced model loading with custom parameters
llama.load(
    pathToModel = "/sdcard/models/llama-7b.gguf",
    nCtx = 2048,        // Larger context for more history
    nThreads = 6,       // More threads on powerful devices
    batchSize = 256,    // Larger batch size for throughput
    mlock = true        // Lock in memory for performance
)
```

## Current Challenges and Solutions

### Challenges

1. **Token-to-Text Conversion Limitations**:
   - The current implementation in `llama-android.cpp` and `simple_llama_jni.cpp` has a simplistic approach to token-to-text conversion, particularly for non-ASCII tokens.
   - The text output may include spacing issues and artifacts, especially with non-English languages.

2. **Memory Management on Mobile Devices**:
   - Large language models require significant memory resources which can exceed what's available on many Android devices.
   - Current implementation may crash on devices with limited RAM when using larger models.

3. **Performance on Lower-End Devices**:
   - Inference speed is slow on entry-level and mid-range Android devices.
   - Balancing quality and speed remains challenging.

4. **API Compatibility Issues**:
   - The SDK needs to maintain compatibility with evolving llama.cpp APIs.
   - Currently using workarounds for some functionality where the APIs have changed.

5. **Limited Sampling Options**:
   - While the SDK implements temperature and top-p sampling, it lacks other advanced techniques like top-k, repetition penalties, etc.

### Solutions

1. **Improved Token-to-Text Conversion**:
   - Implement a more robust token-to-text conversion using llama.cpp's latest tokenization API.
   - Add proper handling for special tokens, multi-byte characters, and BPE-based detokenization.
   - Replace the current simple character-based output with proper token decoding.

2. **Memory Optimization**:
   - Implement model quantization support (4-bit, 5-bit) for reduced memory footprint.
   - Add automatic context pruning to maintain history within memory constraints.
   - Implement streaming KV cache to reduce peak memory usage.

3. **Performance Improvements**:
   - Leverage hardware acceleration via Vulkan compute or ARM compute libraries.
   - Implement more aggressive batching strategies for inference.
   - Add support for smaller, distilled models optimized for mobile.

4. **API Modernization**:
   - Refactor JNI code to use consistent, future-proof API patterns.
   - Create an abstraction layer to handle llama.cpp API changes without requiring client code changes.
   - Update `generate` method implementation to align with the latest llama.cpp approach.

5. **Enhanced Text Generation Controls**:
   - Add support for top-k sampling, frequency penalty, and presence penalty.
   - Implement proper prompt templating with system/user message separation.
   - Add support for guided generation with logit biasing.

## Memory Requirements

- Model size depends on the specific GGUF file used (typically 4-15GB)
- RAM usage scales with model size, context length, and batch size
- Consider device constraints when setting parameters

## Performance Considerations

- Smaller models (7B parameters) work best on mobile devices
- Reducing context size (`nCtx`) significantly lowers memory usage
- Use appropriate thread count based on device CPU (typically 2-6)
- Format chat correctly for better responses
- Consider adding a loading UI during model initialization

## Contribution Guide

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Built on the [llama.cpp](https://github.com/ggerganov/llama.cpp) project
- Inspired by advancements in on-device AI capabilities
- Thanks to all contributors who have helped improve this SDK

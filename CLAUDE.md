# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AudioSR is a versatile audio super-resolution tool that enhances audio quality by upsampling to 48kHz. It works with all types of audio (music, speech, environmental sounds) and supports different input sampling rates. The project uses deep learning models based on latent diffusion for audio enhancement.

## Development Commands

### Installation and Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Install package in development mode
pip install -e .

# Alternative: Install specific version
pip install audiosr==0.0.7
```

### Running the Application
```bash
# Run Gradio web interface
python app.py

# Command-line single file processing
audiosr -i example/music.wav

# Batch processing from file list
audiosr -il batch.lst

# Full command with options
audiosr -i input.wav -s ./output --model_name basic --ddim_steps 50 --guidance_scale 3.5
```

### Development and Testing
```bash
# Run inference script directly
python inference.py

# Build package for distribution
python setup.py sdist bdist_wheel

# Upload to PyPI (maintainer only)
python setup.py upload
```

## Architecture Overview

### Core Components

**Main Entry Points:**
- `app.py` - Gradio web interface with chunking support for long audio
- `bin/audiosr` - Command-line interface script
- `inference.py` - Direct inference script
- `predict.py` - Cog/Replicate prediction interface

**AudioSR Package Structure:**
- `audiosr/pipeline.py` - Main processing pipeline and model loading
- `audiosr/utils.py` - Utility functions (file I/O, audio processing, seeding)
- `audiosr/lowpass.py` - Low-pass filtering utilities
- `audiosr/latent_diffusion/` - Diffusion model implementation
- `audiosr/latent_encoder/` - Audio encoder/decoder components
- `audiosr/hifigan/` - HiFi-GAN vocoder models
- `audiosr/clap/` - CLAP (audio-text) model components
- `audiosr/utilities/` - Additional utility modules

### Key Models

**Available Models:**
- `basic` - General-purpose audio super-resolution model
- `speech` - Optimized for speech enhancement

**Model Configuration:**
- Uses DDIM sampling with default 50 steps
- Guidance scale default: 3.5
- Output sample rate: 48kHz
- Latent time resolution: 12.8 samples per second

### Processing Pipeline

1. **Input Processing**: Audio files are loaded and preprocessed
2. **Chunking**: Long audio files are automatically chunked for processing
3. **Model Inference**: Latent diffusion model processes audio chunks
4. **Post-processing**: Amplitude normalization and artifact removal
5. **Output**: Enhanced 48kHz audio files

## Important Implementation Details

### Cutoff Pattern Sensitivity
AudioSR is trained on low-pass filtered data and may struggle with other degradation patterns (e.g., MP3 compression artifacts). For best results with compressed audio, apply low-pass filtering before processing.

### Memory Management
- Uses `torch.set_float32_matmul_precision("high")` for optimization
- Implements chunking in `app.py` for processing long audio files
- Includes garbage collection strategies for GPU memory management

### Audio Processing Features
- Automatic silence detection and trimming
- Amplitude matching between original and processed audio
- Cross-fade blending for seamless chunk joining
- Support for various audio formats via librosa/soundfile

## Configuration Files

- `requirements.txt` - Python dependencies with CUDA support
- `setup.py` - Package configuration and PyPI metadata
- `cog.yaml` - Replicate/Cog deployment configuration
- `MANIFEST.in` - Package data inclusion rules

## Dependencies

### Core ML Libraries
- PyTorch 2.0.1+ (with CUDA 11.8 support)
- transformers==4.30.2
- diffusers (from git)
- huggingface_hub

### Audio Processing
- librosa==0.9.2
- soundfile
- torchaudio==0.20.2+

### Web Interface
- gradio (for web UI)
- progressbar (for CLI progress)

## Development Notes

### Platform Support
- CUDA builds for Linux/Windows
- CPU-only builds for macOS
- Automatic device selection with `device="auto"`

### File Naming Convention
- Output files use suffix: `_AudioSR_Processed_48K`
- Timestamps are added to output directory names
- Supports custom suffixes via CLI `--suffix` parameter

### Batch Processing
- Use `batch.lst` file format for processing multiple files
- One file path per line in the batch list
- Output preserves original file names with suffix
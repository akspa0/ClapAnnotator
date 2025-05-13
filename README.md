# CLAP Audio Annotator

## Overview

This tool provides a user interface (built with Gradio) to annotate audio files using the CLAP (Contrastive Language-Audio Pretraining) model. It first separates the input audio into 'Vocals' and 'Instrumental' stems using a model from the `python-audio-separator` library, then processes each stem with CLAP based on user-provided text prompts.

This version focuses on processing single audio files.

## Features (Phase 1)

*   **Audio Separation**: Choose from available models (e.g., UVR-MDX-NET, Demucs) to separate vocals and instrumentals.
*   **CLAP Annotation**: Annotate separated audio stems based on text prompts.
*   **Chunking**: Processes long audio files efficiently by analyzing them in chunks.
*   **Prompt Presets**: Load predefined text prompts or save your own frequently used prompts to text files in the `_presets/clap_prompts/` directory.
*   **Configurable Threshold**: Adjust the confidence threshold for CLAP detections via a slider.
*   **Gradio UI**: Easy-to-use web interface for uploading audio, managing presets, selecting models, and viewing results.
*   **CLI Interface**: Command-line interface for processing single files or batch processing multiple files.
*   **Standardized Output**: Saves results in a structured JSON format within a unique, timestamped directory for each input file under `ClapAnnotator_Output/`.

## Prerequisites

*   **Python**: 3.10+
*   **ffmpeg**: You **must** have `ffmpeg` installed on your system and accessible in your system's PATH. This is required for audio resampling. You can download it from [https://ffmpeg.org/](https://ffmpeg.org/).
*   **(Optional) CUDA**: For GPU acceleration (recommended for performance), ensure you have a compatible NVIDIA GPU, CUDA Toolkit, and cuDNN installed. PyTorch should be installed with CUDA support (this is handled by the `transformers[torch]` dependency if CUDA is detected).

## Installation

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd ClapAnnotator
    ```

2.  **Create a Python virtual environment (recommended):**
    ```bash
    python -m venv venv
    # Activate the environment
    # Windows:
    # venv\Scripts\activate
    # macOS/Linux:
    # source venv/bin/activate
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Hugging Face Token:**
    
    You have two options for setting up your Hugging Face token:
    
    **Option 1: Using the setup script (Recommended)**
    
    Run the setup script with your token:
    ```bash
    python setup_hf_token.py --token "YOUR_HUGGING_FACE_TOKEN_HERE"
    ```
    
    Or use interactive mode:
    ```bash
    python setup_hf_token.py --interactive
    ```
    
    **Option 2: Manual setup**
    *   Create a file named `.env` in the project root directory (where `requirements.txt` is located).
    *   Add your Hugging Face API token to this file. You need a token (even a read-only one) to download the CLAP model. Get one from [https://huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).
    *   The content of the `.env` file should be:
        ```
        HF_TOKEN="YOUR_HUGGING_FACE_TOKEN_HERE"
        ```
        Replace `YOUR_HUGGING_FACE_TOKEN_HERE` with your actual token.

## Configuration

*   **Environment Variables**: `HF_TOKEN` is loaded from the `.env` file.
*   **Application Settings**: Most other settings (model names, default thresholds, paths, chunk sizes) are defined in `config/settings.py`. You can modify this file directly if needed.

## Usage

### Gradio Web Interface

The Gradio web interface provides an intuitive, interactive way to annotate single audio files using CLAP. Below is a guide to the interface and workflow:

### Launching the App

1. Activate your Python virtual environment (if using one).
2. Run:
   ```bash
   python gradio_app/app.py
   ```
3. Open the provided local URL (e.g., `http://127.0.0.1:7860`) in your browser.

### Interface Overview

- **Audio File Upload**: Upload a single audio file (supported formats: mp3, wav, flac, etc.).
- **Audio Separation Model Selection**: Choose from available models (e.g., Mel Band RoFormer Vocals, UVR-MDX-NET, Demucs) for separating vocals and instrumentals.
- **CLAP Prompt Input**:
  - **Preset Dropdown**: Select a preset of CLAP prompts from the dropdown (populated from `_presets/clap_prompts/`).
  - **Manual Entry**: Enter custom prompts (one per line) in the text box.
  - **Save Preset**: Enter a name and click "Save Preset" to save the current prompts for future use.
- **Parameter Controls**:
  - **CLAP Confidence Threshold**: Adjust the slider to set the minimum confidence for detections (default: 0.55).
  - **Chunk Duration**: Set the duration (in seconds) for each CLAP analysis chunk (default: 3 seconds).
  - **Audio Preservation**: Toggle whether to save separated audio files alongside the results (enabled by default).
- **Analyze Button**: Click to start the analysis. The UI will show progress updates.
- **Results Display**:
  - **JSON Output**: View the structured results directly in the UI.
  - **Download Link**: Download the results JSON file and any preserved audio files from the output directory.
- **Status/Feedback**: The interface provides real-time feedback on progress, errors, and completion.

### Typical Workflow

1. Upload your audio file.
2. Select the desired audio separation model.
3. Choose a CLAP prompt preset or enter your own prompts.
4. Adjust the confidence threshold and chunk duration if needed.
5. (Optional) Save your custom prompts as a new preset.
6. Click "Analyze Audio".
7. Wait for processing to complete. Progress and status will be shown.
8. Review the results in the UI and download the output files.

### Output Location
- Results are saved in a unique, timestamped subdirectory under `ClapAnnotator_Output/`.
- The output includes a `results.json` file and, if enabled, separated audio files.

### Troubleshooting
- **ffmpeg not found**: Ensure `ffmpeg` is installed and in your system PATH.
- **Model errors**: Check your internet connection and Hugging Face token setup.
- **Unsupported file**: Use a supported audio format and check file integrity.

### Command Line Interface (CLI)

The CLI allows you to process audio files directly from the command line, with support for both single files and batch processing.

#### Basic Usage

Process a single audio file:

```bash
python cli.py path/to/audio.mp3 --prompts "voice,speech,singing"
```

Use a preset for prompts:

```bash
python cli.py path/to/audio.mp3 --preset my_preset_name
```

Use prompts from a text file (one prompt per line):

```bash
python cli.py path/to/audio.mp3 --prompt-file path/to/prompts.txt
```

#### Additional Options

Specify a separator model:

```bash
python cli.py path/to/audio.mp3 --prompts "voice,speech" --separator-model "UVR-MDX-NET Main"
```

Set a custom confidence threshold:

```bash
python cli.py path/to/audio.mp3 --prompts "voice,speech" --confidence 0.6
```

Specify a custom output directory:

```bash
python cli.py path/to/audio.mp3 --prompts "voice,speech" --output-dir path/to/output
```

Keep temporary files after processing:

```bash
python cli.py path/to/audio.mp3 --prompts "voice,speech" --keep-temp
```

#### Batch Processing

Process all audio files in a directory:

```bash
python cli.py path/to/audio/folder --batch --prompts "voice,speech"
```

#### Help

For a complete list of options:

```bash
python cli.py --help
```

## Presets

*   You can add your own CLAP prompt presets by creating `.txt` files in the `_presets/clap_prompts/` directory.
*   Each line in the `.txt` file should contain a single text prompt.
*   The filename (without the `.txt` extension) will be used as the preset name in the Gradio dropdown.
*   You can also save the prompts currently entered in the UI as a new preset using the "Save Current Prompts as Preset Name" input and the "Save Preset" button.

## Output

*   Results for each analysis run are saved in a unique subdirectory within the `ClapAnnotator_Output/` folder.
*   The subdirectory name is based on the sanitized input filename and a timestamp (e.g., `MyAudioFile_20231027_103000`).
*   Inside this directory, you will find the main `results.json` file containing the analysis.
*   Temporary files used during processing are created in a `temp/` subdirectory within the run's output folder and are deleted automatically upon completion or error.

## Future Work (Phase 2)

*   Implement batch processing by allowing folder inputs.
*   Develop a job queue system to manage multiple analyses.
*   Add UI controls to view, pause, and cancel queued jobs.

## License

(Placeholder - Consider adding an appropriate open-source license like MIT)

## Acknowledgements

*   This tool utilizes the excellent `python-audio-separator` library: [https://github.com/nomadkaraoke/python-audio-separator](https://github.com/nomadkaraoke/python-audio-separator)
*   CLAP model and `transformers` library by Hugging Face and the original CLAP authors.
*   `ffmpeg` for audio processing.
*   `Gradio` for the user interface. 
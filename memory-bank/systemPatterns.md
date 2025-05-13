# System Patterns

## Architecture
- **Input**: User provides audio file via Gradio UI.
- **Configuration UI**: User selects audio separation model, CLAP prompts (via presets or direct input), and sets parameters (e.g., CLAP confidence threshold) via Gradio components.
- **Audio Separation**: Uses `python-audio-separator` with user-selected model and parameters to split tracks into a temporary location.
- **Resampling**: Uses `ffmpeg` (via `ffmpeg-python`) for efficient resampling of separated stems to 48kHz for CLAP, saving to a temporary location.
- **Annotation**: Uses CLAP (fused model) processing resampled audio in configurable chunks. Filters results based on the user-defined confidence threshold.
- **Preset Management**: Loads CLAP prompt presets from `_presets/clap_prompts/` on startup. Saves new presets to this directory via UI interaction.
- **Output Generation**: Creates a unique output directory (`BASE_OUTPUT_DIR/sanitized_input_filename_timestamp/`). Saves final annotation results as a JSON file within this directory. Optionally preserves audio files.
- **UI**: Gradio orchestrates the workflow, displays results, and provides download links.
- **Cleanup**: Temporary files (separated, resampled) are removed after processing, but final output files are preserved.

## Key Decisions
- **User-Centric Configuration**: Prioritize exposing relevant options for both audio separation and CLAP annotation to the user via the UI.
- **Sensible Defaults**: Provide reasonable default settings based on testing:
  - Default separator model: "Mel Band RoFormer Vocals"
  - CLAP confidence threshold: 0.55 (optimal based on testing)
  - CLAP chunk duration: 3 seconds (better granularity than 10 seconds)
  - Audio preservation: Enabled by default
- **Preset System**: Implement a file-based preset system for CLAP prompts (`_presets/clap_prompts/`) for extensibility and user convenience.
- **Standardized Output**: Use a consistent, timestamped output directory structure (`BASE_OUTPUT_DIR/filename_timestamp/`) for easy management of results.
- **Performance**: Use `ffmpeg` for resampling and chunking for CLAP to handle large files efficiently.
- **Modularity**: Separate concerns into distinct modules (separation, resampling, annotation, UI, utils).
- **Path Handling**: Use relative paths in JSON output for portability and security.
- **Audio Preservation**: Option to preserve separated audio files alongside the JSON results.

## Error Handling and Edge Cases
- **Input Validation**: Check for readable audio formats (via `ffmpeg`). Handle unreadable/corrupt files gracefully with UI feedback. Check for extremely short audio files (<0.1s) and skip processing or note in output.
- **File Naming**: Sanitize input filenames before using them in output directory paths to remove invalid characters.
- **Dependencies**: Check for `ffmpeg` system availability at app startup and warn the user if missing.
- **Model Failures**: Use `try-except` blocks for model loading (separation, CLAP) and processing steps. Catch network errors, OOM errors (specifically for GPU), and other exceptions. Report errors clearly in the UI.
- **Processing Robustness**: 
  - If separation fails to produce expected stems, skip subsequent steps for missing stems.
  - Wrap individual CLAP chunk processing in `try-except` to allow continuation even if one chunk fails.
- **Resource Issues**: Handle potential `PermissionError` or `OSError` (e.g., disk full) during file/directory operations.
- **UI Feedback**: Use dedicated status components in Gradio for progress updates and error messages. Disable interactive elements during processing. Provide clear feedback on preset load/save operations.
- **Cleanup**: Ensure temporary files are always cleaned up using a `try...finally` block around the main processing workflow.
- **Path Handling**: Handle cases where paths can't be made relative (different drives, etc.) by falling back to filenames. 
# geminilocal

A small GUI utility that wraps calls to the Gemini generative models and helps process long texts by splitting them into chunks, calling the model for each chunk with retry and timeout handling, and saving consolidated results to disk.

This README documents how the program works, what it can do, where it stores results, troubleshooting tips, and next steps for improvement.

## What this program does

- Presents a Tkinter GUI (`GeminiInterface` in `gem5.py`) to configure and run requests against Gemini models.
- Lets you choose a primary model, a language, and a splitting strategy for long texts.
- Splits input text into chapters or sized chunks, sends each chunk as a prompt to the selected model, and writes results to a timestamped file in `~/Downloads/gemini_results/`.
- Performs per-call timeout handling and will retry failed/timeouts with a fallback model (`FALLBACK_MODEL`) automatically.
- Runs the processing in a background thread so the UI stays responsive and shows progress and per-chunk results in the window.

## Important files and symbols

- `gem5.py` — the main application and GUI.
- `GeminiInterface` — main class implementing the UI and orchestration.
- `browse_api_key_file` — UI handler used to load an API key file (first non-empty non-comment line is used).
- `validate_inputs` — checks required inputs before running.
- `start_processing` / `stop_processing` — start/stop controls for processing.
- `split_text` / `smart_split_by_words` — chunking logic (chapter-based or size-based with sentence awareness).
- `_call_gemini_api` — performs a single synchronous call to the Gemini client wrapper (`google.generativeai`).
- `call_gemini_with_retry_and_timeout` — wraps `_call_gemini_api` with a per-call timeout and retry via `FALLBACK_MODEL` when the primary call fails or times out.
- `_process_request_thread` — main worker executed in a background thread: splits text, calls the API per chunk, updates progress, and writes results.
- `load_results` — loads the latest result file into the UI.
- `FALLBACK_MODEL` and `API_TIMEOUT_SECONDS` — constants controlling retry/fallback behavior and per-call timeout.

## How it works (detailed)

1. Start the GUI:

```powershell
python .\gem5.py
```

2. Load your Gemini API key:

- Click "Chọn File API Key" and pick a text file. The code reads the first non-empty, non-`#` line and uses it as `api_key`.

3. Configure the request in the GUI:
- Select a model from the dropdown (`self.model`).
- Choose language (`中文`, `ENG`, `Việt Nam`). This affects splitting semantics.
- Choose splitting method: "Theo chương (第X章/Chương X)" (chapter/section regex) or "Theo số ký tự" (size-based).
- For size-based splitting, set the split length (characters for non-ENG, words for ENG).
- Enter the prompt and the long text to process.

4. Click "Gửi yêu cầu":
- The app validates inputs (`validate_inputs`).
- Text is split by `split_text`:
  - Chapter mode uses a regex to detect chapter headings (supports patterns like `第...章`, `Chương <n>`, `Chapter <n>`). If no chapters found, it falls back to splitting on blank lines.
  - Size mode splits by words (English) via `smart_split_by_words`, or by characters for other languages while attempting to cut on sentence boundaries.
- For each chunk, the worker constructs `full_prompt = prompt + '\n\n' + chunk` and calls `call_gemini_with_retry_and_timeout(primary_model, full_prompt)`:
  - `_call_gemini_api` uses `google.generativeai` (imported as `genai`) to create a `GenerativeModel(model_name)` and call `generate_content(prompt_content)`; it returns the text or an error.
  - `call_gemini_with_retry_and_timeout` runs `_call_gemini_api` in a ThreadPoolExecutor with a timeout (`API_TIMEOUT_SECONDS`). If the primary call errors or times out, it retries once with `FALLBACK_MODEL`. Errors are logged and written to the result file; processing continues for the next chunk.
- Progress and per-chunk results are displayed in the UI and aggregated status lines are appended to the completion area.
- Results are appended to a file in `~/Downloads/gemini_results/` named like `gemini_result_YYYY-MM-DD-HH-MM-SS_{model}.txt`.

5. Stopping:
- Clicking "Dừng" sets `self.should_stop`, and the worker checks that flag between chunks to stop cleanly. Partial results remain in the output file.

## Where results are stored

- Path: `~/Downloads/gemini_results/` (created if missing).
- Filenames: `gemini_result_{timestamp}_{model}.txt`, where `{timestamp}` is the local time when processing started and `{model}` is the primary model name (slashes replaced to produce a safe filename).

## Requirements

- Python 3.8 or newer
- Tkinter (usually bundled with standard Python installers)
- `google-generative-ai` client (imported in code as `google.generativeai` and used as `genai`).

You can create a minimal `requirements.txt` like:

```
google-generative-ai
```

Note: installation and package name/versioning depends on the client distribution; if you use a different client or a pinned version, add it to `requirements.txt`.

## Usage examples (PowerShell)

Start the GUI from repository root:

```powershell
python .\gem5.py
```

Tips for API key file:
- Put your API key as the first non-empty, non-`#` line in a `.txt` file. Example `my_api_key.txt`:

```
# My Gemini API key
AIza...your_key_here...
```

## Troubleshooting

- If you get timeouts often, increase `API_TIMEOUT_SECONDS` in `gem5.py` or choose a different model.
- If the UI freezes, ensure your Python has `tkinter` and the app isn't blocked by an external dependency; the heavy work runs in a background thread.
- If the program cannot write files, check permissions for `~/Downloads/gemini_results/`.
- If `_call_gemini_api` raises errors, inspect console logs printed by the script for exception messages and stack traces.

## Edge cases & behavior

- Long sentences (longer than the chunk size) are broken into sub-chunks by `smart_split_by_words` to avoid infinite loops.
- If no chapter markers and no blank-line breaks are found for chapter mode, the text is processed as a single chunk with a warning.
- Each chunk is attempted even if previous chunks fail; errors are written to the results file and processing continues.

## Suggested next steps (I can implement any of these)

- Add a `requirements.txt` and a short `install` section showing `pip install -r requirements.txt`.
- Add a sample API key file (e.g. `sample_api_key.txt.example`) showing the expected format.
- Add a small automated test for `split_text` and `smart_split_by_words`.
- Add a `LICENSE` file and CONTRIBUTING guide.
- Add a small screenshot or short GIF demonstrating the UI.

If you'd like, I can implement any of these items now — tell me which one and I'll update the repo.

---

Last updated: October 25, 2025
# geminilocal

A small repository containing `gem5.py`.

This README provides quick information about the project, how to run the script, and where to go next.

## Overview

The repository currently contains a single script, `gem5.py`. The script's purpose and behavior depend on its contents — if you need, I can extend this README with details about the script's expected inputs, outputs, and configuration after you share more information or allow me to inspect the file.

## Prerequisites

- Python 3.8 or newer is recommended. Verify with:

```powershell
python --version
```

If your system uses `python3` as the executable, use that instead.

## Usage (PowerShell)

Run the script from the repository root. Examples:

```powershell
# Run the script with the default Python interpreter
python .\gem5.py

# If your system requires python3
python3 .\gem5.py
```

If the script accepts command-line arguments, run it with `-h` or `--help` to see usage information (if implemented):

```powershell
python .\gem5.py -h
```

## Files

- `gem5.py` — main script in this repository.

## Development

- If you want to add dependencies, create a `requirements.txt` or `pyproject.toml` and list the packages there.
- For editing, open the repository in your favorite editor and run the script locally.

## Contributing

If you'd like me to expand the README (examples, arguments, sample input/output), or to add a license, tests, or a minimal CI workflow, tell me what you'd like included and I’ll implement it.

## License

No license file is included by default. Add a `LICENSE` file if you want to make the project explicit about reuse terms.

## Contact

If you want help improving or documenting `gem5.py`, reply here or grant permission for me to inspect `gem5.py` so I can add accurate usage examples.

# FAQ

## Installation and Setup

### I get a `ModuleNotFoundError` when running `python -m src.main`
Make sure you have installed dependencies:
```bash
pip install -r requirements.txt
```
Also verify your current working directory is the project root (the folder containing `src/`).

### The application starts but `custom-algorithms/` are not loaded
- For **local runs**: ensure `CUSTOM_ALGORITHMS_DIR` is set in `.env` (default is `./custom-algorithms`). The `.env` file must be in the project root.
- For **Docker**: the volume mapping in `docker-compose.yml` must point to the correct host directory. Check that the host path defined in .env exists and contains `.py` files.
- Run with `DEBUG=true` to see logs about loaded algorithms.

### I changed an algorithm file but the UI doesn’t see it
The algorithm loader caches instances at startup. Restart the application. (There is a `AlgorithmLoader.reload()` class method that can be called programmatically, but it’s not exposed in the UI by default.)

### Docker build fails
Ensure your `requirements.txt` includes `python-dotenv`. The Dockerfile installs from that file. If you added new packages, run `docker-compose build --no-cache` to rebuild.

## Algorithms

### My algorithm doesn’t appear in the sidebar
Check:
1. The file is under the directory specified by `CUSTOM_ALGORITHMS_DIR`.
2. The class inherits from `BaseAlgorithm`.
3. The class has `enabled = True` (or omits it; default is `True`).
4. The file doesn’t contain syntax errors (enable `DEBUG=true` to see import errors).

### Why do I get a “Security Check Failed” error?
Your algorithm contains forbidden operations, such as `import`, `open`, `socket`, introspection (`__subclasses__`), or access to disallowed built‑ins. The sandbox only allows a minimal set of built‑ins (see the **Algorithm Implementation Guide**). Remove the forbidden code.

### Why do I get “Algorithm timed out after {X}s”?
Your algorithm took too long, likely due to an infinite loop or very large input. Check loop conditions and ensure the input size respects `ARRAY_MAX_LENGTH` (default 32).

### Why do I get “Error running algorithm: No result from subprocess”?
Your algorithm may have generated too many steps or used too much memory. Optimize your algorithm, restrict `ARRAY_MAX_LENGTH`, or adjust `ALGO_TIMEOUT` and `ALGO_MAX_MEMORY_MB`

### How do I use the `needs_target` property?
If your algorithm requires a target value (e.g., search), set:
```python
@property
def needs_target(self) -> bool:
    return True
```
The UI will show a “Target Value” input. Inside `execute`, the `input_data` argument will be a tuple `(array, target)`.

## Annotations and Visualization

### What annotation categories are available?
`default`, `success`, `failure`, `compare`, `swap`, `pivot`, `partition`, `merge`, `discard`. They map to predefined colors. You can override with a `color` key.

### How do I highlight a range of elements?
Use a tuple `(start, end)` in the `annotations` dict:
```python
annotations={(0, 5): {'category': 'sorted'}}
```
This applies the category to all indices from 0 to 5 inclusive.

### How do I draw a vertical separator?
Add `'separator_after': True` to an annotation for a given index. The separator appears after that element.

## Data Structures

### How do I add a new built‑in data structure?
Implement `BaseDataStructure` in `src/datastructures/`, add it to `BUILTIN_DS_CLASSES` in `page_coordinator.py`, and register a factory in `_register_components`. The existing `DataStructureVisualizer` and `OperationPanel` will work immediately.

### The “Generate Random Mutations” button does nothing
Ensure the data structure has elements. Mutations only operate when the structure is non‑empty. Also check that the `generate_random_mutation_steps` method is implemented.

## Development and Debugging

### How do I see debug logs?
Set `DEBUG=true` in `.env` (or the container environment). Debug messages print to the console with the client IP.

### How do I run only specific tests?
```bash
python -m unittest tests.test_e_bubble_sort
```

### How do I regenerate the class diagrams?
```bash
pip install pylint
pyreverse src/ -o puml -f ALL -m y -L --colorized -p DSA-Visualizer
```
Then convert `.puml` files to images using PlantUML.

### The code mentions `mp_main` in `main.py`. Why?
NiceGUI uses multiple workers in production; the `if __name__ == '__mp_main__'` guard prevents the server from starting twice per worker. This is normal.

## General

### Can I run this on Windows?
Yes, but resource limits (CPU, memory) are only enforced on Linux or WSL. The sandbox still works; the subprocess timeout will block infinite loops.

### Can I expose this to the internet?
The application includes a security sandbox, but it is always recommended to run behind a reverse proxy with HTTPS in production. Do not expose the raw NiceGUI server. Ensure that file permissions are configured correctly such that only authorized users can add custom algorithm implementations.

### Where can I ask more questions?
Contact {{config.extra.team_contact}}.
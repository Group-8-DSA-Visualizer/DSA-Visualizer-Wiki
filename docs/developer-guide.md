# Developer Guide

This guide is for maintainers and contributors who want to understand the codebase, run tests, debug, and generate diagrams.

## Project Structure

```
.
├── .env.example                       # Environment variable template
├── docker-compose.yml                 # Docker Compose file
├── Dockerfile                         # Container image definition
├── requirements.txt                   # Core dependencies
├── test.py                            # Test runner script
├── compile_all.py                     # Bytecode compilation of custom algorithms
├── custom-algorithms/                 # Instructor‑provided algorithm source code
│   ├── demo/                           # Malicious test algorithms (enabled with DEMO=true)
│   ├── search/                         # Example search algorithms
│   └── sorting/                        # Example sorting algorithms
├── src/
│   ├── main.py                         # Application entry point
│   ├── config.py                       # Central configuration, env loading
│   ├── styles.css                      # Global CSS styles
│   ├── algorithms/                         # Compiled algorithm bytecode (.pyc)
│   │   └── base_algorithm.py                   # Abstract base class for algorithms
│   ├── components/                     # Reusable UI components
│   │   ├── common/                         # Abstract bases, playback, slider, sidebar, steps list
│   │   ├── algorithms/                     # Linear algorithm run config & visualizer
│   │   └── datastructures/                 # Data structure visualizer, run config, operation panel
│   ├── datastructures/                 # Built‑in data structures (ArrayList, Stack, Queue)
│   ├── pages/                          # NiceGUI page definitions
│   │   ├── home.py                         # Main page layout
│   │   └── page_coordinator.py             # Coordinates sidebar, run configs, visualizers
│   ├── services/                       # Core services
│   │   ├── algorithm_loader.py             # Discovery and loading of algorithms
│   │   ├── algorithm_executor.py           # Sandboxed subprocess execution
│   │   ├── algorithm_runner.py             # Async execution with thread/queue polling
│   │   ├── animation_player.py             # Step rendering via pluggable renderers
│   │   ├── component_registry.py           # Singleton registry for all components
│   │   └── renderers/                      # Step renderers (array, future tree, etc.)
│   └── utils/                          # Helpers (array validation, debug logging)
└── tests/                             # Unit tests
```

## Key Files and Their Roles

| File                                             | Purpose                                                                                                                               |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| `src/config.py`                                  | Loads `.env` variables; provides defaults for all configurable values.                                                                |
| `src/services/algorithm_loader.py`               | Scans `custom-algorithms/` and `src/algorithms/` for Python files, instantiates algorithm classes, stores source code for sandboxing. |
| `src/services/algorithm_executor.py`             | AST verification, resource limits, subprocess sandbox, safe built‑ins.                                                                |
| `src/services/algorithm_runner.py`               | Wraps sandboxed execution in a background thread with a pollable queue.                                                               |
| `src/components/common/step_based_visualizer.py` | Visualizer for linear-data-structure-based with playback, slider, steps list, and long description support.                           |
| `src/pages/page_coordinator.py`                  | Wires sidebar, run config, visualizer; registers all components into `ComponentRegistry`.                                             |
| `src/components/common/sidebar.py`               | Sidebar UI for selection of loaded algorithms.                                                                                        |
| `src/services/animation_player.py`               | Dispatches step rendering to type‑specific renderers.                                                                                 |

## Environment Variables

All configurable runtime parameters are set via environment variables. For local development, copy `.env.example` to `.env` and edit. In Docker, they are passed via `docker-compose.yml` (with the exception of `HOST` and `PORT`).

| Variable                | Default               | Description                                                  |
|-------------------------|-----------------------|--------------------------------------------------------------|
| `CUSTOM_ALGORITHMS_DIR` | `./custom-algorithms` | Path to custom algorithm source files                        |
| `HOST`                  | `0.0.0.0`             | Host address for the application                             |
| `PORT`                  | `8080`                | Port to bind the application to                              |
| `ARRAY_MAX_LENGTH`      | `32`                  | Maximum length of generated or input arrays                  |
| `ARRAY_MAX_VALUE`       | `128`                 | Maximum integer value allowed in arrays                      |
| `ARRAY_MIN_VALUE`       | `0`                   | Minimum integer value allowed                                |
| `DS_MAX_LENGTH`         | `32`                  | Maximum size of data structures (Stack, Queue, etc.)         |
| `ALGO_MAX_CONCURRENT`   | `8`                   | Maximum concurrent algorithm executions                      |
| `ALGO_TIMEOUT`          | `10`                  | Maximum seconds an algorithm may run before being terminated |
| `ALGO_MAX_MEMORY_MB`    | `256`                 | Maximum memory (in MB) allowed per sandbox subprocess        |
| `DEBUG`                 | `false`               | Enable verbose debug logging                                 |
| `DEMO`                  | `false`               | Enable display of algorithms in `demo/` directory            |

## Running Tests

From the project root:

```bash
python test.py
```

This discovers and runs all unit tests in `tests/`.

To run a single test file:

```bash
python -m unittest tests.test_algorithm_executor
```

### Test structure

- `tests/test_algorithm_executor.py` - sandbox security verification, subprocess execution.
- `tests/test_algorithm_loader_core.py` - loading algorithms from directories, code filtering for sandbox.
- `tests/test_e_bubble_sort.py`, `test_e_insertion_sort.py`, `test_e_quicksort_sort.py` - verify algorithm correctness, step format, and edge cases.

## Debug Logging

Set `DEBUG=true` in `.env` or pass it as an environment variable. Debug messages are printed to the console with the client IP address (if available).

In code, use:
```python
from src.utils.debug_logger import debug_log
debug_log("message")
```

## Generating Diagrams

The repository includes PlantUML class and package diagrams. To regenerate them:

```bash
pip install pylint
pyreverse src/ -o puml -f ALL -m y -L --colorized -p DSA-Visualizer
```

This outputs `classes_DSA-Visualizer.puml` and `packages_DSA-Visualizer.puml`. You can render them to images using the PlantUML tool or online server.

## Architecture Notes (SOLID)

- **Single Responsibility**: UI, execution, and rendering are separated into distinct classes.
- **Open/Closed**: New algorithms and visualizations are added via registration, not modification.
- **Liskov Substitution**: All implementations of `BaseAlgorithm`, `BaseVisualizer`, `BaseDataStructure` respect their interface contracts.
- **Interface Segregation**: Abstract classes expose minimal methods (e.g., `BaseRenderer` only requires `render`).
- **Dependency Inversion**: High‑level modules (`PageCoordinator`, `Sidebar`) rely on abstractions (`ComponentRegistry`, `BaseVisualizer`).

## Adding a New Algorithm

1. Place a `.py` file in `custom-algorithms/<category>/`.
2. Subclass `BaseAlgorithm` and implement all abstract properties and `execute`.
3. Restart the application (or call `AlgorithmLoader.reload()`).
4. The algorithm appears automatically in the sidebar.

## Adding a New Data Structure

1. Create a class in `src/datastructures/` that inherits `BaseDataStructure`.
2. Register it in `PageCoordinator.BUILTIN_DS_CLASSES` and add a factory in `_register_components`.
3. The existing `DataStructureVisualizer` and `OperationPanel` work out of the box.

## Contributing

- Strive to follow PEP 8 style.
- Write tests for any new feature or bug fix.
- Update the documentation in this repository for new features or breaking changes.
- Run `python test.py` before submitting a pull request.

For architecture details and extension guides, see the [Architecture Overview](architecture.md) and [Extending the System](extending-the-system.md) pages.
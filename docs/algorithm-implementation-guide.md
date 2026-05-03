# Algorithm Implementation Guide

This guide is for **instructors** (or students) who want to write their own algorithm implementations for the DSA Visualizer.  
You’ll learn how to structure your code, define visualization steps, and integrate your algorithm into the platform.

## 1. Where to put your code

Place your Python files inside the **`custom-algorithms/`** directory (or the directory specified by the `CUSTOM_ALGORITHMS_DIR` environment variable).  
You can organize your files in subdirectories (e.g., `sorting/`, `search/`, `trees/`). The loader scans all `.py` files recursively.

For example:
```
custom-algorithms/
  sorting/
    my_awesome_sort.py
  search/
    my_custom_search.py
```

## 2. The BaseAlgorithm interface

Every algorithm must inherit from `BaseAlgorithm` and implement the following abstract properties and methods.

### Import the base class
```python
from algorithms.base_algorithm import BaseAlgorithm
from typing import List, Any, Dict
```

### Required properties

| Property        | Type   | Description                                                                                                |
|-----------------|--------|------------------------------------------------------------------------------------------------------------|
| `name`          | `str`  | Unique identifier (e.g., `"my_sort"`). Used internally.                                                    |
| `display_name`  | `str`  | Human‑readable name shown in the UI (e.g., `"My Awesome Sort"`).                                           |
| `input_type`    | `str`  | The type of input the algorithm expects. Currently only `"array"` is supported by the built‑in run config. |
| `category`      | `str`  | Grouping label for the sidebar (e.g., `"Array Sorting"`, `"Array Searching"`, `"Custom"`).                 |

### Optional properties

- `needs_target` (`bool`, default `False`) – if `True`, the UI will show an extra “Target Value” input. Use this for search algorithms that need a target element.
- `enabled` (`bool`, default `True`) – set to `False` to hide the algorithm from the UI (e.g., during development).
- `show_code` (`bool`, default `False`) – reserved for future use; currently ignored.

### The `execute` method
```python
def execute(self, input_data: Any) -> List[Dict[str, Any]]:
    ...
```
- **`input_data`** – For `input_type == "array"` and `needs_target == False`, this is a `List[int]`.  
  For `needs_target == True`, this is a tuple `(List[int], int)` where the second element is the target value.
- **Returns** a list of **step dictionaries** (described below).

## 3. The Step Dictionary

Each step describes one 'frame' of the visualization. The renderer (currently `ArrayRenderer`) uses the keys to draw bars, colors, labels, and separators.

### Required keys
- **`type`** (`str`) – Must be `"array"` for array‑based visualizations.
- **`data`** (`List[int]`) – The array values at this step.
- **`annotations`** (`dict`) – A dictionary describing how to style each index or range.
- **`description`** (`str`) – Short text shown below the visualization (current step description).

### Optional keys
- **`long_description`** (`str`) – Detailed explanation visible when the step is displayed.

### The `annotations` dictionary

Keys can be:
- **Single index** (int) – applies the properties to that element.
- **Range** (tuple `(start, end)`, inclusive) – applies the properties to all indices from `start` to `end`. The renderer expands ranges automatically.

Properties are a dict that can contain:

| Property          | Type    | Description                                                                        |
|-------------------|---------|------------------------------------------------------------------------------------|
| `category`        | `str`   | Semantic role; maps to a predefined color (see below).                             |
| `color`           | `str`   | Explicit CSS color (overrides `category`).                                         |
| `offset_x`        | `int`   | Pixel offset for the bar (horizontal).                                             |
| `offset_y`        | `int`   | Pixel offset for the bar (vertical).                                               |
| `separator_after` | `bool`  | Draws a vertical separator line after this element.                                |
| `removed`         | `bool`  | Marks the element as removed (shows `[X]` index, does not count in logical index). |

### Category colors

| Category      | Color   |
|---------------|---------|
| `default`     | Blue    |
| `success`     | Green   |
| `failure`     | Red     |
| `compare`     | Yellow  |
| `swap`        | Yellow  |
| `pivot`       | Purple  |
| `partition`   | Orange  |
| `merge`       | Teal    |
| `discard`     | Gray    |

## 4. Writing your first algorithm – a complete example

### Example 1: Bubble Sort with detailed annotations

```python
from algorithms.base_algorithm import BaseAlgorithm
from typing import List, Any, Dict

class MyBubbleSort(BaseAlgorithm):
    enabled = True
    show_code = True  # for future use

    @property
    def name(self) -> str:
        return "my_bubble_sort"

    @property
    def display_name(self) -> str:
        return "My Bubble Sort"

    @property
    def input_type(self) -> str:
        return "array"

    @property
    def category(self) -> str:
        return "Array Sorting"

    def execute(self, arr: List[int]) -> List[Dict[str, Any]]:
        steps = []
        n = len(arr)
        data = arr.copy()

        # Initial state
        steps.append({
            'type': 'array',
            'data': data.copy(),
            'annotations': {(0, n - 1): {'category': 'default'}},
            'description': 'Initial array'
        })

        for i in range(n - 1):
            for j in range(n - i - 1):
                # Comparison
                steps.append({
                    'type': 'array',
                    'data': data.copy(),
                    'annotations': {
                        (0, n - i - 1): {'category': 'default'},
                        (n - i, n - 1): {'category': 'success'},
                        (j, j + 1): {'category': 'compare'}
                    },
                    'description': f'Comparing arr[{j}]={data[j]} with arr[{j+1}]={data[j+1]}'
                })

                if data[j] > data[j + 1]:
                    data[j], data[j + 1] = data[j + 1], data[j]
                    # Swap
                    steps.append({
                        'type': 'array',
                        'data': data.copy(),
                        'annotations': {
                            (0, n - i - 1): {'category': 'default'},
                            (n - i, n - 1): {'category': 'success'},
                            j: {'category': 'swap'},
                            j + 1: {'category': 'swap'}
                        },
                        'description': f'Swapping arr[{j}] and arr[{j+1}]'
                    })

            # Mark sorted section
            steps.append({
                'type': 'array',
                'data': data.copy(),
                'annotations': {
                    (n - i - 1, n - 1): {'category': 'success'}
                },
                'description': f'Positions {n - i - 1} through {n - 1} are now sorted'
            })

        # Final sorted state
        steps.append({
            'type': 'array',
            'data': data.copy(),
            'annotations': {(0, n - 1): {'category': 'success'}},
            'description': 'Array is fully sorted'
        })

        return steps
```
!!! note "Mutable Data Types"
    It is important that the data key passes a *copy* of the data structure, rather than the object
    that you are operating on directly. Otherwise, the visualizer will display the final state
    of the object for every step.

### Example 2: Linear Search with target

```python
class MyLinearSearch(BaseAlgorithm):
    @property
    def name(self) -> str:
        return "my_linear_search"

    @property
    def display_name(self) -> str:
        return "My Linear Search"

    @property
    def input_type(self) -> str:
        return "array"

    @property
    def category(self) -> str:
        return "Array Searching"

    @property
    def needs_target(self) -> bool:
        return True

    def execute(self, input_data) -> List[Dict[str, Any]]:
        arr, target = input_data
        steps = []
        data = arr.copy()

        steps.append({
            'type': 'array',
            'data': data.copy(),
            'annotations': {},
            'description': f'Searching for target {target}'
        })

        for i, val in enumerate(data):
            steps.append({
                'type': 'array',
                'data': data.copy(),
                'annotations': {i: {'category': 'compare'}},
                'description': f'Checking index {i}: value {val}'
            })
            if val == target:
                steps.append({
                    'type': 'array',
                    'data': data.copy(),
                    'annotations': {i: {'category': 'success'}},
                    'description': f'Found {target} at index {i}'
                })
                return steps

        steps.append({
            'type': 'array',
            'data': data.copy(),
            'annotations': {(0, len(data)-1): {'category': 'failure'}},
            'description': f'{target} not found'
        })
        return steps
```

### Example 3: Binary Search with range discarding

See the file `custom-algorithms/search/e_binary_search.py` in the repository for a full implementation that uses the `discard` category and `separator_after` to show discarded halves.

## 5. Using `long_description` for study mode

Add a `long_description` key to a step to provide extra detail for students.

```python
step = {
    'type': 'array',
    'data': data.copy(),
    'annotations': ...,
    'description': 'Swapping elements 2 and 3',
    'long_description': 'This swap moves the larger element upward, gradually building the sorted portion at the end of the array.'
}
```

## 6. Security and sandbox restrictions

Algorithms execute in a **restricted subprocess** with limited built‑in functions. The following operations are **forbidden** and will cause a runtime error:

- File I/O (`open`, `read`, `write`)
- Network access (`socket`, `urllib`)
- Subprocess creation (`subprocess`, `os.system`)
- Import of external modules (`import`, `from ... import ...`) – only the `BaseAlgorithm` stub and `typing` are available inside the sandbox.
- Introspection (`__subclasses__`, `__globals__`, `getattr`, `dir`, etc.)
- Infinite loops (enforced by a CPU time limit of 10 seconds)

**Allowed built‑ins** include: `abs`, `all`, `any`, `bool`, `dict`, `enumerate`, `float`, `int`, `len`, `list`, `max`, `min`, `range`, `round`, `sorted`, `str`, `sum`, and class‑related internals like `property`, `super`.

This is an extra security layer to protect against a file write permission misconfiguration to the `custom-algorithms` directory. Particularly, for if the project is hosted on public-facing infrastructure, such as CSU's Grail server.

If your algorithm relies on an external library, you must implement the logic manually using only the allowed built‑ins.

## 7. Testing your algorithm

### Using the provided test framework

From the project root, run:
```bash
python test.py
```
This discovers and runs all tests in the `tests/` directory.

### Adding your own tests

1. Create a new test file in `tests/` (e.g., `test_my_algorithm.py`).
2. Add `sys.path` adjustments to locate the source (see existing tests for reference).
3. Write standard `unittest` test cases that call `algorithm.execute()` and verify:
   - The return type is a list.
   - Each step has the required keys (`type`, `data`, `annotations`, `description`).
   - The final `data` is correctly sorted/searched.
   - Edge cases: empty array, single element, duplicates, already sorted input.

Example test skeleton:
```python
import unittest
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).parent.parent / "custom-algorithms"))
from sorting.my_sort import MySort

class TestMySort(unittest.TestCase):
    def test_sorts_array(self):
        algo = MySort()
        steps = algo.execute([3, 1, 2])
        final = steps[-1]
        self.assertEqual(final['data'], [1, 2, 3])
```

## 8. Deploying

All algorithms are loaded once upon service start. To see any changes to existing algorithms or additions of new ones, you will need to restart the main application server.

### Docker deployment

When using Docker, the `custom-algorithms/` directory on the host is mounted directly into the container (`/custom-algorithms`). 

The `CUSTOM_ALGORITHMS_DIR` environment variable is automatically set to `/custom-algorithms` inside the container. You don’t need to adjust it.

## 9. Troubleshooting

| Symptom                                 | Likely Cause                                                                                       | Solution                                                                                                          |
|-----------------------------------------|----------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Algorithm not appearing in sidebar      | File not in the correct directory, or `enabled = False`, or the application hasn't been restarted. | Check the path under `CUSTOM_ALGORITHMS_DIR`, and ensure the class has `enabled = True`. Restart the application. |
| "Forbidden operation" error             | Used disallowed built‑in (e.g., `open`, `import`)                                                  | Remove the forbidden call; re‑implement the logic manually.                                                       |
| "Algorithm timed out"                   | Infinite loop or very slow execution                                                               | Check for loops that may not terminate; add early exits.                                                          |
| Steps not rendering                     | Missing required keys in step dict                                                                 | Verify each step has `type`, `data`, `annotations`, `description`.                                                |
| Target field not showing                | `needs_target` not set                                                                             | Add `@property\ndef needs_target(self): return True`.                                                             |


For further help, enable `DEBUG=true` in `.env` to see detailed logs, or consult the [Developer Guide](developer-guide.md).
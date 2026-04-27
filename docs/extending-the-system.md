# Extending the System

The DSA Visualizer is built to be extended with new visualization types (trees, graphs, tables, etc.) without modifying core code. This guide walks you through the process.

## 1. Extensibility Points

The system provides three main extension hooks:

| Hook                  | What it does                                                           | Where to implement                                                                     |
|-----------------------|------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| **Renderer**          | Draws a single step of a given `type` (e.g., `"array"`, `"tree"`).     | Subclass `BaseRenderer` and register in `AnimationPlayer`.                             |
| **Visualizer**        | Manages steps, playback, description, and UI layout for a component.   | Subclass `StepBasedVisualizer` (or `BaseVisualizer`) and define `_build_display_area`. |
| **Run Configuration** | UI for input and execution control of a component.                     | Subclass `BaseRunConfig`.                                                              |

All components are registered in the **ComponentRegistry**, a singleton that the sidebar and page coordinator read. Registration is done in `PageCoordinator._register_components()`.

---

## 2. Adding a New Renderer

A renderer translates a step dictionary into NiceGUI elements. For a new visualization type (e.g., `"tree"`), you need a renderer that understands the step format.

### Step 1: Define the step format

Document what keys your renderer expects. A tree may look like:

```python
{
    'type': 'tree',
    'root': 5,
    'nodes': {1: {'left': 2, 'right': 3}, 2: {}, 3: {}},
    'annotations': {1: {'category': 'compare'}, 2: {'category': 'success'}},
    'description': 'Comparing node 1 with target'
}
```

### Step 2: Implement the renderer

Create a new file, e.g., `src/services/renderers/tree_renderer.py`:

```python
from nicegui import ui
from typing import Dict, Any
from src.services.renderers.base_renderer import BaseRenderer

class TreeRenderer(BaseRenderer):
    def render(self, step: Dict[str, Any]) -> None:
        root = step.get('root')
        nodes = step.get('nodes', {})
        annotations = step.get('annotations', {})
        # Use NiceGUI elements (ui.html, ui.svg, etc.) to draw the tree.
        # You could recursively build rows/columns or use an HTML canvas.
        with ui.column().classes('items-center'):
            self._draw_node(root, nodes, annotations)

    def _draw_node(self, node_id, nodes, annotations):
        ann = annotations.get(node_id, {})
        color = ann.get(...)
        with ui.row().classes('items-center'):
            ui.label(str(node_id)).style(...)
        # ... draw children recursively
```

### Step 3: Register the renderer

In `src/services/animation_player.py`, add the renderer:

```python
from src.services.renderers.tree_renderer import TreeRenderer

class AnimationPlayer:
    _renderers: Dict[str, Any] = {
        'array': ArrayRenderer(),
        'tree': TreeRenderer(),
    }
```

Alternatively, use the class method `AnimationPlayer.add_renderer('tree', TreeRenderer())` from anywhere (e.g., in `PageCoordinator._register_components`).

---

## 3. Creating a New Visualizer

If the new visualization needs a different layout or extra UI elements (e.g., an operations panel for interactive trees), create a visualizer.

### Subclass `StepBasedVisualizer`

```python
from nicegui import ui
from src.components.common.step_based_visualizer import StepBasedVisualizer

class TreeVisualizer(StepBasedVisualizer):
    def __init__(self):
        super().__init__()
        self.finish_initialization()   # builds the UI

    def _build_display_area(self):
        self.display_area = ui.column().classes('w-full h-full items-center justify-center')
        self.display_area.style('min-height: 512px;')
```

You can override `_add_extra_ui_after_display` or `_add_extra_ui_after_description` to insert custom buttons (e.g., "Insert Node", "Delete Node") that interact with a data structure.

### Data structure integration

If the visualizer works with a stateful tree object (like a `BaseDataStructure` subclass), pass it in the constructor and reuse the operation panel pattern from `DataStructureVisualizer`. See the existing `DataStructureVisualizer` for an example.

---

## 4. Creating a Run Configuration (Optional)

If the new component requires custom input (e.g., edges for a graph, initial tree values), subclass `BaseRunConfig` and build the necessary UI.

```python
from nicegui import ui
from src.components.common.base_run_config import BaseRunConfig

class TreeRunConfig(BaseRunConfig):
    def __init__(self, tree_instance):
        super().__init__()
        self.tree = tree_instance
        self._build_ui()

    def _build_ui(self):
        ui.separator().classes('q-my-xs')
        with ui.column().classes('w-full items-start gap-0 q-mb-md'):
            # Add inputs, buttons, etc.
            self.run_btn = ui.button('Run Tree Algorithm', on_click=self._run)
```

---

## 5. Register the Component

Edit `src/pages/page_coordinator.py` in the `_register_components` method. Add a block for the new type:

```python
# Inside PageCoordinator._register_components():
def make_tree_build(loader):
    from my_tree_module import MyTree  # or use BUILTIN_DS_CLASSES pattern
    inst = MyTree()
    rc = TreeRunConfig(inst)
    vis = TreeVisualizer(inst)
    rc.set_visualizer(vis)
    return rc, vis

registry.register('my_tree', 'datastructure', 'My Tree', 'Trees', make_tree_build)
```

If you also registered a renderer, the visualizer will automatically use it because `AnimationPlayer` now knows about `'tree'` steps.

---

## 6. Adding Built-in Data Structures

To add a new interactive data structure (like a heap or deque), implement the `BaseDataStructure` abstract class. The existing `ArrayList`, `Stack`, and `Queue` are good references.

### Step 1: Create a class in `src/datastructures/`

```python
from src.datastructures.base_data_structure import BaseDataStructure
from typing import List, Any, Dict

class MinHeap(BaseDataStructure):
    # implement all abstract methods
    ...
```

### Step 2: Register in `PageCoordinator.BUILTIN_DS_CLASSES` and add an `OperationPanel`-compatible visualizer.

The `DataStructureVisualizer` already supports any `BaseDataStructure` via the `OperationPanel`. So you can reuse it directly, just passing the new instance.

---

## 7. Example: Adding a Binary Search Tree

Here’s a minimal end‑to‑end example.

### BST step format

```python
step = {
    'type': 'tree',
    'root': 10,
    'nodes': {10: {'left': 5, 'right': 15}, 5: {}, 15: {}},
    'annotations': {10: {'color': 'blue'}, 5: {'category': 'success'}},
    'description': 'Inserted 5'
}
```

### Renderer (simplified)

```python
class BSTRenderer(BaseRenderer):
    def render(self, step):
        root = step['root']
        nodes = step['nodes']
        ann = step.get('annotations', {})
        # Build an HTML tree using ui.html or ui.svg.
        html = self._build_html_tree(root, nodes, ann)
        ui.html(html).classes('w-full')

    def _build_html_tree(self, node_id, nodes, ann):
        if node_id is None:
            return ''
        info = nodes.get(node_id, {})
        color = ann.get(node_id, {}).get('color', '#000')
        left = self._build_html_tree(info.get('left'), nodes, ann)
        right = self._build_html_tree(info.get('right'), nodes, ann)
        return f'<div style="color:{color}">{node_id}</div><div style="display:flex">{left}{right}</div>'
```

### Registration

```python
AnimationPlayer._renderers['tree'] = BSTRenderer()
registry.register('bst_example', 'datastructure', 'Binary Search Tree', 'Trees', make_bst_build)
```

Now a new “Trees” category appears in the sidebar, and selecting it loads the BST visualizer with custom input.

---

## 8. Making Extensions Discoverable (Future)

Currently, the `PageCoordinator` hard‑codes a set of known components. For a fully dynamic system, you could:

- Use entry points (Python `setuptools` `entry_points`) to discover extensions.
- Scan a dedicated `plugins/` directory for classes implementing a `Plugin` interface that registers itself.
- Add a `register` method on `ComponentRegistry` that accepts class references and instantiates them lazily.

These enhancements are optional and can be added when multiple external teams need to contribute components.

---

Need help? Refer to the [Developer Guide](developer-guide.md) for project structure and debugging tips.
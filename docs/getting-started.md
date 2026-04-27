# Getting Started

This guide walks you through setting up DSA Visualizer on your local machine.

## Prerequisites

- **Python 3.10+** – [Download](https://www.python.org/downloads/)
- **Git** – [Download](https://git-scm.com/downloads)
- **Docker** (optional) – [Docker Desktop](https://www.docker.com/products/docker-desktop) for containerized deployment

!!! note "Source Code Access"
    The main application repository is private. Make sure your SSH key or personal access token is configured to clone it. Contact {{config.extra.team_contact}} if you need access.
    If you are a contributor as part of a future capstone team, it is recommended to contact Mr. Robert Fiske for his copy of the source files.

## Installation (Local)

### 1. Clone the repository
```bash
git clone {{config.extra.source_repo_git}}
cd DSA-Visualizer
```

### 2. Create a virtual environment (recommended)
```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```
This installs the core framework ([NiceGUI](https://nicegui.io/)), `python-dotenv`, and all other required libraries.

### 4. Configure environment variables
Copy the example environment file and adjust it to your needs:
```bash
cp .env.example .env
```
Open `.env` in your editor. Key variables are explained below.

#### Custom algorithms location
`CUSTOM_ALGORITHMS_DIR` tells the application where your custom algorithm files live.
- **Default:** `./custom-algorithms` (the directory shipped with the project)
- **Local development:** leave as default, or set an absolute path to a different folder.
- **Docker:** this value is overridden automatically inside the container. The host path is still used for the volume mount.

#### Other important variables

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

All limits are enforced server‑side to keep the visualizer responsive and safe.

## Running the Application

### Option 1 – Directly with Python
```bash
python -m src.main
```
The server starts on **http://localhost:8080** (or your configured address). Open that address in your browser.

!!! tip "Custom algorithms outside the project folder"
    If you store your custom algorithms elsewhere, set `CUSTOM_ALGORITHMS_DIR` in `.env` to the full path. For example:
    ```ini
    CUSTOM_ALGORITHMS_DIR=/home/user/my-algorithms
    ```
    The application will scan that directory on startup.

### Option 2 – Using Docker (recommended for production)
```bash
docker-compose up
```
The first build may take a minute. Once ready, the application is available at the same URL: **http://localhost:8080**.

#### How the volume mapping works
The `docker-compose.yml` automatically mounts your custom algorithms directory into the container:
```yaml
volumes:
  - ${CUSTOM_ALGORITHMS_DIR:-./custom-algorithms}:/custom-algorithms:ro
```
- The host path is taken from the `CUSTOM_ALGORITHMS_DIR` variable in your `.env` file (or defaults to `./custom-algorithms`).
- Inside the container it always appears at `/custom-algorithms`.
- No further configuration is needed. Simply update the `.env` variable if your algorithms are stored elsewhere.

## First Algorithm Run

1. Open **http://localhost:8080** in your browser.
2. In the left sidebar, expand a category (e.g., *Array Sorting*) and click **Example Bubble Sort**.
3. In the *Run Configuration*, Enter an array of your choosing or click **Random Array**.
4. Click **Run Algorithm**.
5. The visualization area draws the initial array as colored bars and numbered boxes. Use the playback buttons or the slider to step through the sorting process.
6. Click **Reset** to clear the visualization and try another algorithm.

## What’s Next?

- Learn how to write your own algorithm in the [Algorithm Implementation Guide](algorithm-implementation-guide.md).
- Understand the high‑level design in the [Architecture Overview](architecture.md).
- Read about the codebase structure in the [Developer Guide](developer-guide.md).
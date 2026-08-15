# CrewAI Jupyter Project

A Jupyter Notebook learning project that demonstrates how to build and run a multi-agent workflow using [CrewAI](https://www.crewai.com/).

The project uses specialized AI agents to plan, write, and edit an article based on a user-supplied topic. It also demonstrates how to configure CrewAI locally with Python, `uv`, Jupyter, and the OpenAI API.

## Project Overview

The notebook builds a CrewAI workflow consisting of three agents:

* **Content Planner** — researches and plans relevant, factually accurate content for the selected topic.
* **Content Writer** — uses the planner's output to create an article.
* **Editor** — reviews and refines the article into the final response.

The agents execute a corresponding sequence of tasks:

```text
Topic
  |
  v
Content Planner
  |
  v
Content Writer
  |
  v
Editor
  |
  v
Final Article
```

A topic is supplied when the crew is started, for example:

```python
result = await crew.akickoff(
    inputs={"topic": "Artificial Intelligence"}
)
```

The final CrewAI output can then be rendered as Markdown inside Jupyter:

```python
from IPython.display import Markdown, display

display(Markdown(result.raw))
```

## Project Files

The primary notebook is:

```text
L2_research_write_article.ipynb
```

Other important project files include:

```text
.
├── L2_research_write_article.ipynb
├── pyproject.toml
├── uv.lock
├── .python-version
├── .gitignore
└── .env                 # Local only — not committed to Git
```

The `.venv` virtual environment is also kept locally and excluded from source control.

## Technology Stack

This project uses:

* Python 3.13
* CrewAI
* CrewAI Tools
* Jupyter Notebook
* `uv` for Python package and environment management
* OpenAI API
* `python-dotenv`
* Git and GitHub

## Prerequisites

Before running the project, you will need:

* Git
* Python 3.13
* `uv`
* An OpenAI API key

Python versions used by CrewAI should be checked against the current CrewAI requirements before upgrading the project environment.

## Clone the Repository

Clone the repository:

```bash
git clone https://github.com/harvey-magana/crewai-jupyter-project.git
```

Move into the project directory:

```bash
cd crewai-jupyter-project
```

## Install the Project Environment

This project uses `uv` for dependency management.

Synchronize the environment from `pyproject.toml` and `uv.lock`:

```bash
uv sync
```

Verify the Python interpreter:

```bash
uv run python --version
```

The project is currently designed to use Python 3.13.

You can also verify that CrewAI is available:

```bash
uv run python -c "import crewai; print(crewai.__version__)"
```

## Configure the OpenAI API Key

The OpenAI API key should **not** be stored directly inside the notebook or committed to GitHub.

Create a `.env` file in the project root:

```bash
touch .env
```

Add your API key:

```text
OPENAI_API_KEY=your_openai_api_key_here
```

The `.env` file is excluded from Git through `.gitignore`.

The notebook loads the key with `python-dotenv`:

```python
import os
from dotenv import load_dotenv

load_dotenv()

if not os.getenv("OPENAI_API_KEY"):
    raise ValueError("OPENAI_API_KEY is not configured")
```

Never commit a real API key to the repository.

## Configure the Jupyter Kernel

CrewAI must be available in the same Python environment used by the notebook.

If necessary, register the project's virtual environment as a Jupyter kernel:

```bash
uv run python -m ipykernel install \
  --user \
  --name crewai-jupyter-project \
  --display-name "Python 3.13 - CrewAI"
```

Then select:

```text
Python 3.13 - CrewAI
```

as the notebook kernel.

You can verify which interpreter Jupyter is using from inside the notebook:

```python
import sys

print(sys.executable)
print(sys.version)
```

The interpreter should point to the project's `.venv`.

## Launch Jupyter

One option is to launch Jupyter through `uv`:

```bash
uv run --with jupyter jupyter lab
```

Open:

```text
L2_research_write_article.ipynb
```

and select the project kernel.

## CrewAI Execution

The crew consists of agents and tasks such as:

```python
from crewai import Agent, Task, Crew
```

Example crew configuration:

```python
crew = Crew(
    agents=[planner, writer, editor],
    tasks=[plan, write, edit],
    verbose=True
)
```

Because Jupyter runs an asynchronous event loop, the crew can be executed using CrewAI's asynchronous API:

```python
result = await crew.akickoff(
    inputs={"topic": "Artificial Intelligence"}
)
```

The generated content is available through the `CrewOutput` object:

```python
print(result.raw)
```

or rendered as Markdown:

```python
from IPython.display import Markdown, display

display(Markdown(result.raw))
```

## Compatibility Notes

This project was adapted from tutorial material written against an earlier version of CrewAI. Several changes were required to run the project with the current Python and CrewAI environment.

### Boolean Verbose Setting

Older examples may use:

```python
verbose=2
```

Modern CrewAI expects a Boolean value:

```python
verbose=True
```

### Asynchronous Execution in Jupyter

Calling the synchronous method:

```python
crew.kickoff(...)
```

may result in an error related to a running event loop inside Jupyter.

The notebook instead uses:

```python
result = await crew.akickoff(...)
```

### CrewOutput Instead of a String

Modern CrewAI returns a `CrewOutput` object rather than a plain string.

Instead of:

```python
Markdown(result)
```

use:

```python
Markdown(result.raw)
```

### Environment Variables

Some older tutorial environments provide helper functions such as:

```python
from utils import get_openai_api_key
```

That helper is not part of this local project.

This project instead uses a standard `.env` file and `python-dotenv` to load:

```text
OPENAI_API_KEY
```

## Troubleshooting

### `ModuleNotFoundError: No module named 'crewai'`

Confirm that Jupyter is using the project's virtual environment:

```python
import sys
print(sys.executable)
```

From Terminal, compare it with:

```bash
uv run python -c "import sys; print(sys.executable)"
```

Both should reference the same project environment.

### `OPENAI_API_KEY is not configured`

Confirm that:

1. `.env` is a **file**, not a directory.
2. `.env` is located in the project directory.
3. It contains:

```text
OPENAI_API_KEY=your_openai_api_key_here
```

4. `load_dotenv()` is called before reading the environment variable.

### Jupyter Event Loop Error

If CrewAI reports that synchronous execution was invoked from a running event loop, use:

```python
result = await crew.akickoff(
    inputs={"topic": "Artificial Intelligence"}
)
```

### Markdown `TypeError`

If Jupyter reports:

```text
Markdown expects text, not CrewOutput
```

use:

```python
display(Markdown(result.raw))
```

instead of:

```python
Markdown(result)
```

## Security

The repository intentionally excludes local secrets and environment files, including:

```text
.env
.venv/
.ipynb_checkpoints/
__pycache__/
```

API credentials should always be loaded from environment variables and must never be committed to Git.

## Purpose

This repository is a hands-on learning project for understanding:

* AI agent orchestration
* CrewAI agents and tasks
* Sequential multi-agent workflows
* LLM integration
* OpenAI API authentication
* Asynchronous AI execution in Jupyter
* Python virtual environment management with `uv`
* Dependency troubleshooting
* Git and GitHub project management

## Future Improvements

Potential enhancements include:

* Adding custom CrewAI tools
* Experimenting with additional LLM providers
* Adding structured agent outputs
* Separating agents and tasks into reusable Python modules
* Adding additional CrewAI workflows
* Adding automated tests
* Adding an `.env.example` file
* Improving notebook documentation and examples

## Author

**Harvey Magana**

GitHub: [harvey-magana](https://github.com/harvey-magana)

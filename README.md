# Knowledge Base - Runtime Revolution

Welcome to the Runtime Revolution Knowledge Base. Here you should find all the necessary information and documentation to get you started.

## Live Documentation

You can access the live documentation at [https://runtimerevolution.github.io/knowledge-base/](https://runtimerevolution.github.io/knowledge-base/).

## Setup Instructions

### Install Poetry

To set up this project, you need to have Poetry installed on your system. Poetry is a dependency manager for Python projects. If you do not have Poetry installed, you can install it using Homebrew.

```bash
brew install poetry
```

### Clone the Repository

Clone the project repository to your local machine and navigate to the project directory

```bash
git clone https://github.com/RuntimeRevolution/knowledge-base.git && cd knowledge-base
```

### Install Dependencies

Use Poetry to install the project dependencies

```bash
poetry install
```
### Serve the Documentation Locally

Start the MkDocs development server to serve the documentation locally:

```bash
poetry run mkdocs serve
```

This command will start a local web server. You can view the documentation by navigating to `http://127.0.0.1:8000/` in your web browser.

## Additional Resources

- [Poetry Documentation](https://python-poetry.org/)
- [MkDocs Material Documentation](https://squidfunk.github.io/mkdocs-material/)

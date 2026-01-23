# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **nbdev-based machine learning blog** that uses literate programming. Notebooks in the `nbs/` directory serve a dual purpose:
1. Blog posts and documentation (rendered as a website via Quarto)
2. Python module source code (exported to the `machine_learning_blog/` package)

The blog is published to GitHub Pages and covers machine learning use cases, examples, and tutorials.

## Development Workflow

### Initial Setup
```bash
pip install nbdev
nbdev_install_quarto  # Install Quarto for rendering
nbdev_install_hooks   # Install git hooks for automatic notebook cleaning
pip install -e ".[dev]"  # Install package in editable mode with dev dependencies
```

### Standard Development Cycle
1. **Edit notebooks** in `nbs/` directory
   - Use `#|default_exp module_name` at the top to specify which module code exports to
   - Use `#|export` before code cells to mark them for export to Python modules
   - Code without `#|export` is treated as examples/documentation only

2. **Prepare changes** before committing:
   ```bash
   nbdev_prepare  # One command that does: export, test, clean, build docs
   ```

3. **Preview locally**:
   ```bash
   nbdev_preview  # Start local server to preview the blog
   ```

4. **Commit and push** - deployment to GitHub Pages happens automatically on push to main/master

### Individual Commands

When you need more granular control:
- `nbdev_export` - Export notebook code to `machine_learning_blog/` Python modules
- `nbdev_test` - Run tests from notebooks (cells are tested during execution)
- `nbdev_clean` - Clean notebook outputs and metadata (required before commit)
- `nbdev_docs_build` - Build documentation site
- `nbdev_readme` - Generate README.md from index.ipynb

## Critical Rules

### DO NOT Edit Generated Files
**NEVER** directly edit files in these directories - they are auto-generated from notebooks:
- `machine_learning_blog/*.py` (except when adding new modules not from notebooks)
- `_docs/`
- `README.md` (generated from `nbs/index.ipynb`)

To change code in Python modules, edit the source notebook in `nbs/` and run `nbdev_export`.

### Notebook Hygiene
- Git hooks automatically clean notebooks on commit, but CI will fail if notebooks aren't cleaned
- Always run `nbdev_prepare` before committing to ensure notebooks are properly cleaned
- The CI checks for unstripped notebook outputs - commits will fail if outputs remain

## Project Structure

```
nbs/
├── index.ipynb              # Homepage (generates README.md)
├── 00_about_me.ipynb        # About page
├── 00a_references.ipynb     # References and resources
├── 00b_publications.ipynb   # Publications list
├── 02_cli.ipynb            # CLI module source (exports to machine_learning_blog/cli.py)
├── 01_*.ipynb              # Blog posts on various ML topics
├── 03_*.ipynb              # (Difference-in-Differences, monitoring, etc.)
├── datasets/               # Data files used in notebooks
├── _quarto.yml            # Quarto website configuration
├── sidebar.yml            # Website sidebar configuration
└── styles.css             # Custom styling

machine_learning_blog/      # Auto-generated Python package
├── __init__.py
├── cli.py                  # Exported from nbs/02_cli.ipynb
└── core.py                 # Core utilities

settings.ini                # nbdev configuration (project metadata, dependencies, etc.)
```

## Key nbdev Directives

Use these special comments in notebook cells to control nbdev behavior:

- `#|default_exp module_name` - Set the default module for code exports (first cell of notebook)
- `#|export` - Export this code cell to the Python module
- `#|hide` - Hide cell in documentation
- `#|eval: false` - Don't execute during testing

## Dependencies

**Core:** fastcore, pandas, feature-engine, matplotlib
**Development:** jupyter, nbdev, Quarto

Install all: `pip install -e ".[dev]"`

## CI/CD

### Test Workflow (.github/workflows/test.yaml)
Runs on all pushes and PRs:
1. Checks notebooks are cleaned (no outputs/metadata)
2. Runs `nbdev_test` to execute notebook tests

### Deploy Workflow (.github/workflows/deploy.yaml)
Runs on push to main/master:
- Automatically builds and deploys blog to GitHub Pages using Quarto

## Console Scripts

Defined in settings.ini, available after install:
- `hello_blog` - Example CLI command
- `build_lib` - Build library command

## Making Changes

### Adding a New Blog Post
1. Create `nbs/XX_topic_name.ipynb` (XX = number for ordering)
2. Write content as markdown and code cells
3. Run `nbdev_prepare` to test and build
4. Use `nbdev_preview` to check locally
5. Commit and push

### Adding a New Python Module
1. Create `nbs/XX_module_name.ipynb`
2. Add `#|default_exp module_name` in first cell
3. Mark cells to export with `#|export`
4. Run `nbdev_export` to generate `machine_learning_blog/module_name.py`
5. Run `nbdev_test` to test

### Updating Existing Code
1. Find the source notebook in `nbs/` (DO NOT edit .py files directly)
2. Edit the notebook
3. Run `nbdev_prepare`
4. Commit and push

## Testing

Tests are embedded in notebooks. Any executable code in exported cells is tested by `nbdev_test`.
To skip testing on specific notebooks, they're marked with `tst_flags = notest` in settings.ini.

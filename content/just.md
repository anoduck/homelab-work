[Just](https://just.systems/) is a workflow runner and tool for the command line. It is a convenient way to run automated tasks and commands. Think of it as a simpler more basic type of shell script. 

While it is true you could do everything you needed just to do with writing a shell script, using just avoids the complexity and pitfalls of shell scripts. It's "just easier" to use.

## Just Template

Below is my exhaustive template for the creation of just files. It includes way more than you will probably ever need. So, don't just copy it and expect it to run. Only use what you need to use.

```justfile
# =============================================================================
# General-Purpose Justfile Template
# =============================================================================
# A clean, modern starting point for project-specific justfiles.
# Copy this file to your project root as `justfile` (or `Justfile`) and customize.
#
# Quick start:
#   just              → list all available recipes
#   just <recipe>     → run a recipe
#   just --list       → same as default
#   just --groups     → list recipe groups
#
# Docs: https://just.systems
# =============================================================================

# -----------------------------------------------------------------------------
# Settings
# -----------------------------------------------------------------------------
# Use bash for consistent behavior across platforms.
set shell := ["bash", "-euo", "pipefail", "-c"]

# Load .env file automatically (if present).
set dotenv-load := true

# Allow later recipes/variables to override earlier ones (useful with imports).
# set allow-duplicate-recipes := true
# set allow-duplicate-variables := true

# Pass recipe arguments as positional arguments to the shell.
# set positional-arguments := true

# Enable unstable features if needed (user-defined functions, etc.).
# set unstable := true

# -----------------------------------------------------------------------------
# Variables
# -----------------------------------------------------------------------------
# Override any of these from the command line, e.g.:
#   just build PROFILE=release
#   just run ARGS="--verbose"

project_name := `basename $(pwd)`
# Or hardcode: project_name := "my-project"

# Common paths
src_dir     := "src"
build_dir   := "build"
dist_dir    := "dist"
scripts_dir := "scripts"

# Tooling / environment (prefer env() so they can be overridden)
export PATH := env("PATH")
# Example of a tool wrapper (uncomment & adapt as needed):
# mise_exec := "mise exec --"

# -----------------------------------------------------------------------------
# Default / Help
# -----------------------------------------------------------------------------

# List available recipes (this is the default when you just run `just`)
[group('meta')]
default:
    @just --list --unsorted

# Show help / list recipes with descriptions
[group('meta')]
help:
    @just --list --unsorted

# -----------------------------------------------------------------------------
# Setup & Installation
# -----------------------------------------------------------------------------

# Install project dependencies / bootstrap the environment
[group('setup')]
install:
    @echo "→ Installing dependencies..."
    # Example (replace with your package manager):
    # {{mise_exec}} npm install
    # {{mise_exec}} uv sync
    # {{mise_exec}} cargo fetch
    @echo "✓ Done. Customize the 'install' recipe for your project."

# One-time project setup (hooks, tools, etc.)
[group('setup')]
setup: install
    @echo "→ Running one-time setup..."
    # Example:
    # pre-commit install
    # {{mise_exec}} just install-tools
    @echo "✓ Setup complete."

# -----------------------------------------------------------------------------
# Development
# -----------------------------------------------------------------------------

# Start development server / watch mode
[group('dev')]
dev *args:
    @echo "→ Starting development mode..."
    # Example:
    # {{mise_exec}} npm run dev -- {{args}}
    # {{mise_exec}} cargo watch -x run
    # {{mise_exec}} uv run uvicorn app:app --reload
    @echo "(Customize the 'dev' recipe)"

# Run the application (production-like)
[group('dev')]
run *args:
    @echo "→ Running {{project_name}}..."
    # Example:
    # ./{{build_dir}}/{{project_name}} {{args}}
    @echo "(Customize the 'run' recipe)"

# -----------------------------------------------------------------------------
# Quality: Format, Lint, Test
# -----------------------------------------------------------------------------

# Format source code
[group('quality')]
fmt:
    @echo "→ Formatting..."
    # Example:
    # {{mise_exec}} cargo fmt
    # {{mise_exec}} ruff format .
    # {{mise_exec}} prettier --write .
    @echo "✓ Formatted."

# Check formatting (CI-friendly, no writes)
[group('quality')]
fmt-check:
    @echo "→ Checking formatting..."
    # Example:
    # {{mise_exec}} cargo fmt -- --check
    # {{mise_exec}} ruff format --check .
    @echo "✓ Format check passed."

# Lint source code
[group('quality')]
lint:
    @echo "→ Linting..."
    # Example:
    # {{mise_exec}} cargo clippy -- -D warnings
    # {{mise_exec}} ruff check .
    # {{mise_exec}} eslint .
    @echo "✓ Lint passed."

# Run tests
[group('quality')]
test *args:
    @echo "→ Running tests..."
    # Example:
    # {{mise_exec}} cargo test {{args}}
    # {{mise_exec}} pytest {{args}}
    # {{mise_exec}} npm test -- {{args}}
    @echo "✓ Tests passed."

# Run all quality checks (great for CI / pre-commit)
[group('quality')]
check: fmt-check lint test
    @echo "✓ All checks passed."

# -----------------------------------------------------------------------------
# Build & Package
# -----------------------------------------------------------------------------

# Build the project
[group('build')]
build:
    @echo "→ Building {{project_name}}..."
    # Example:
    # {{mise_exec}} cargo build --release
    # {{mise_exec}} npm run build
    # mkdir -p {{build_dir}} && ...
    @echo "✓ Build complete → {{build_dir}}/"

# Create distribution artifacts
[group('build')]
dist: build
    @echo "→ Creating distribution..."
    # Example: package, generate checksums, etc.
    @echo "✓ Distribution ready → {{dist_dir}}/"

# -----------------------------------------------------------------------------
# Cleanup
# -----------------------------------------------------------------------------

# Remove build artifacts and temporary files
[group('util')]
clean:
    @echo "→ Cleaning..."
    rm -rf {{build_dir}} {{dist_dir}}
    # Add more as needed:
    # rm -rf target/ node_modules/ .pytest_cache/ __pycache__/ *.egg-info
    @echo "✓ Clean."

# Deep clean (including caches, lockfiles if desired — use with care)
[group('util')]
[confirm("This will remove more than just build artifacts. Continue?")]
clean-all: clean
    @echo "→ Deep cleaning..."
    # rm -rf .cache/ node_modules/ target/ vendor/ etc.
    @echo "✓ Deep clean complete."

# -----------------------------------------------------------------------------
# Utilities & Helpers
# -----------------------------------------------------------------------------

# Print useful project information
[group('util')]
info:
    @echo "Project : {{project_name}}"
    @echo "Justfile: {{justfile()}}"
    @echo "OS      : {{os()}} / {{os_family()}}"
    @echo "Arch    : {{arch()}}"
    @echo "CPUs    : {{num_cpus()}}"

# Open an interactive shell with the project environment
[group('util')]
shell:
    @echo "→ Opening project shell..."
    # Example: exec into a container, activate venv, etc.
    $SHELL

# -----------------------------------------------------------------------------
# Private helpers (not shown in `just --list`)
# -----------------------------------------------------------------------------

# Example private recipe (prefix with _ )
_banner message:
    @printf '\n\033[1;32m==> %s\033[0m\n' "{{message}}"

# -----------------------------------------------------------------------------
# Notes for customization
# -----------------------------------------------------------------------------
# 1. Replace the placeholder echo statements with real commands.
# 2. Prefer small, composable recipes + dependencies over long scripts.
# 3. Use [group('name')] to organize related recipes.
# 4. Use [confirm("message")] for destructive actions.
# 5. Use [private] or leading _ for internal helpers.
# 6. For complex logic, put scripts in {{scripts_dir}}/ and call them.
# 7. Modules: you can split into multiple files with `mod name 'path/to/file.just'`
# 8. Run `just --fmt --unstable` to auto-format this justfile.
#
# Happy building!
```
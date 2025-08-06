# Suggested Commands for Development

## Environment Setup
```bash
# Create and activate Python virtual environment with uv (recommended)
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate
uv init

# Install dependencies
uv add -r requirements.txt --active

# Start Jupyter notebook instance
uv run --with jupyter jupyter lab
```

## Code Quality Commands (for projects with Makefile)
```bash
make test                    # Run pytest tests
make quality                 # Run all quality checks (format, lint, typecheck, security)
make format                  # Format code with black
make lint                    # Lint code with ruff
make typecheck               # Type check with mypy
make security                # Security scan with bandit
make install-dev             # Install development dependencies
make clean                   # Clean build artifacts and cache
```

## Code Quality Commands (for projects with pyproject.toml)
```bash
uv run pytest              # Run tests
uv run black .              # Format code
uv run ruff check .         # Lint code
uv run ruff check . --fix   # Auto-fix linting issues
uv run mypy .               # Type checking
uv run bandit -r . -f console  # Security scan
```

## Project-Specific Commands
```bash
# Setup scripts (common pattern)
./setup.sh                  # Initial project setup
./deploy.sh                 # Deploy to AWS
./cleanup.sh                # Cleanup resources

# Development servers
./start.sh                  # Start development environment
./start_manual.sh           # Manual start (where available)
```

## Integration Testing
```bash
# Integration testing (where available)
./test-sdk-simple.sh        # Test SDK agent functionality
./test-diy-simple.sh        # Test DIY agent functionality
```

## Development Workflow Commands
```bash
# Build and deployment for SRE projects
./build_and_deploy.sh       # Build and deploy agent

# Environment variables for debugging
DEBUG=true LLM_PROVIDER=anthropic ./build_and_deploy.sh
LOCAL_BUILD=true ./build_and_deploy.sh
```

## System Utilities (macOS/Darwin)
```bash
ls                          # List files
cd <directory>              # Change directory  
grep <pattern> <files>      # Search in files
find <path> -name <pattern> # Find files
git <command>               # Git version control
docker <command>            # Docker containerization
```

## AWS Configuration
```bash
aws configure               # Configure AWS credentials
aws configure set region us-east-1  # Set AWS region
```
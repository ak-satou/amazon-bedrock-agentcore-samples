# Task Completion Checklist

## After Code Changes
When you complete coding tasks, follow these steps to ensure code quality:

### For Projects with Makefile (e.g., SRE-agent)
1. **Run Quality Checks**: `make quality`
   - This runs format, lint, typecheck, and security checks
2. **Run Tests**: `make test`
3. **Individual checks if needed**:
   - Format: `make format`
   - Lint: `make lint` or `make lint-fix` (auto-fix)
   - Type check: `make typecheck`
   - Security: `make security`

### For Projects with pyproject.toml
1. **Format Code**: `uv run black .`
2. **Lint Code**: `uv run ruff check .` (or `uv run ruff check . --fix` for auto-fix)
3. **Type Check**: `uv run mypy .`
4. **Security Scan**: `uv run bandit -r . -f console`
5. **Run Tests**: `uv run pytest`

### For Repository-wide Changes
If working at the repository root level:
1. Check if individual project requirements exist
2. Follow project-specific patterns
3. Run quality checks in affected subdirectories

## Before Deployment
1. **Test Integration**: Run project-specific integration tests if available
2. **Verify Configuration**: Check configuration files are properly set
3. **Security Review**: Ensure no secrets or keys are exposed
4. **Documentation**: Update relevant README files if functionality changed

## Git Workflow
1. **Stage Changes**: `git add <files>`
2. **Check Status**: `git status`
3. **Commit**: Only commit when explicitly asked by user
4. **Review Diffs**: `git diff` to verify changes

## Quality Standards
- **Zero linting errors** from ruff
- **Type checking passes** with mypy
- **Security scan clean** with bandit
- **Code formatted** with black
- **Tests pass** with pytest
- **No secrets committed** to repository

## Common Issues to Check
- AWS credentials not committed
- Environment variables properly configured
- Import statements organized correctly
- Type hints added for new functions
- Docstrings present for new classes/functions
- Error handling implemented appropriately
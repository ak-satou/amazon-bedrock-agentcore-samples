# Code Style and Conventions

## Python Code Style
- **Formatter**: Black with line length 88 characters
- **Linter**: Ruff for fast Python linting
- **Type Checking**: MyPy with Python 3.10+ type hints
- **Target Version**: Python 3.10+
- **Docstrings**: Standard Python docstring conventions
- **Import Style**: Organized imports with Ruff rules (E, F, I, N, W)

## Agent Framework Patterns

### BedrockAgentCoreApp Entrypoint Pattern
All agent implementations follow this common pattern:
```python
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
async def invoke(payload, context):
    # Extract user message from payload
    user_message = payload.get("prompt", "No prompt provided")
    # Process with agent framework
    # Return formatted response
```

### Configuration Management
Projects use hierarchical YAML configuration:
- `config/static-config.yaml`: Version-controlled base configuration
- `config/dynamic-config.yaml`: Deployment-generated configuration
- Configuration merging handled by `AgentCoreConfigManager` utility

### MCP Tool Integration Pattern
```python
from strands.tools.mcp.mcp_client import MCPClient

def create_mcp_client(gateway_url, access_token):
    def create_transport():
        headers = {"Authorization": f"Bearer {access_token}"}
        return streamablehttp_client(gateway_url, headers=headers)
    return MCPClient(create_transport)
```

### Authentication Patterns
- OAuth2/OIDC integration using `@requires_access_token` decorator
- Support for Cognito, Okta, and other identity providers
- Token management utilities in `shared/auth.py` modules

## File Organization
### Standard Project Structure
```
project/
├── README.md              # Project-specific documentation
├── requirements.txt       # Python dependencies
├── pyproject.toml        # Python project configuration (where applicable)
├── config/               # Configuration files
├── shared/               # Shared utilities and helpers
├── scripts/              # Setup and deployment scripts
├── tests/ or test/       # Test files
└── [project-specific]/   # Main application code
```

## Naming Conventions
- **Files**: snake_case for Python files
- **Classes**: PascalCase
- **Functions/Methods**: snake_case
- **Constants**: UPPER_CASE
- **Variables**: snake_case

## Security Best Practices
- Never expose or log secrets and keys
- Never commit secrets or keys to repository
- Use environment variables for sensitive configuration
- Implement proper authentication and authorization patterns
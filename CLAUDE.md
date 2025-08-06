# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Amazon Bedrock AgentCore Samples repository containing examples, tutorials, and use cases for building AI agents using Amazon Bedrock AgentCore capabilities. The repository demonstrates integration patterns across multiple frameworks including Strands Agents, LangGraph, LangChain, CrewAI, and OpenAI Agents.

## Development Commands

### Environment Setup
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

### Testing Commands
```bash
# For projects with Makefile (e.g., SRE-agent)
make test                    # Run pytest tests
make quality                 # Run all quality checks (format, lint, typecheck, security)
make format                  # Format code with black
make lint                    # Lint code with ruff
make typecheck               # Type check with mypy
make security                # Security scan with bandit

# For projects with pyproject.toml
uv run pytest              # Run tests
uv run black .              # Format code
uv run ruff check .         # Lint code
uv run mypy .               # Type checking

# Integration testing (where available)
./test-sdk-simple.sh        # Test SDK agent functionality
./test-diy-simple.sh        # Test DIY agent functionality
```

### Project-Specific Commands
```bash
# Setup scripts (common pattern)
./setup.sh                  # Initial project setup
./deploy.sh                 # Deploy to AWS
./cleanup.sh                # Cleanup resources

# Development servers
./start.sh                  # Start development environment
./start_manual.sh           # Manual start (where available)
```

## Repository Architecture

### Core Structure
```
01-tutorials/          # Interactive Jupyter notebook tutorials
├── AgentCore-runtime/    # Runtime capability examples
├── AgentCore-gateway/    # Gateway tool integration examples  
├── AgentCore-identity/   # Identity and access management
├── AgentCore-memory/     # Memory management examples
├── AgentCore-tools/      # Built-in tools (Code Interpreter, Browser)
└── AgentCore-observability/ # Monitoring and tracing

02-use-cases/          # End-to-end practical applications
├── AWS-operations-agent/    # AWS operations automation
├── SRE-agent/              # Site reliability engineering
├── customer-support-assistant/ # Support ticket management
└── [other-use-cases]/

03-integrations/       # Framework integration examples
├── agentic-frameworks/     # Strands, LangGraph, LangChain, CrewAI
└── bedrock-agent/          # Bedrock Agent integration

04-my-projects/        # User project workspace
```

### Key Dependencies
- **bedrock-agentcore**: Core AgentCore SDK
- **strands-agents**: Primary agent framework used in examples  
- **langgraph**: Multi-agent workflow framework
- **boto3**: AWS SDK for Python
- **opentelemetry**: Observability and tracing

## Common Patterns

### Agent Framework Integration
All agent implementations follow a common entrypoint pattern:
```python
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
async def invoke(payload):
    # Extract user message from payload
    # Process with agent framework (Strands, LangGraph, etc.)
    # Return formatted response
```

### Configuration Management
Projects use hierarchical YAML configuration:
- `config/static-config.yaml`: Version-controlled base configuration
- `config/dynamic-config.yaml`: Deployment-generated configuration
- Configuration merging handled by `AgentCoreConfigManager` utility

### MCP Tool Integration
Standard pattern for MCP (Model Context Protocol) tool integration:
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

### AWS Service Integration
- Consistent boto3 client management across projects
- Region configuration via environment variables or YAML config
- Standard Bedrock model initialization patterns

## Development Guidelines

### Project Structure
Most projects follow this structure:
```
project/
├── README.md              # Project-specific documentation
├── requirements.txt       # Python dependencies
├── config/               # Configuration files
├── shared/               # Shared utilities and helpers
├── scripts/              # Setup and deployment scripts
├── tests/ or test/       # Test files
└── [project-specific]/   # Main application code
```

### Testing Approach
- **pytest** for Python unit and integration tests
- **Docker-based testing** for full integration scenarios
- **Shell scripts** for end-to-end testing workflows
- Test file naming: `test_*.py` for Python, `*.test.js` for JavaScript

### Code Quality Tools
- **black**: Code formatting (line length 88)
- **ruff**: Fast Python linting
- **mypy**: Static type checking
- **bandit**: Security vulnerability scanning

### Observability
- OpenTelemetry instrumentation enabled by default
- LangSmith integration for agent tracing
- AWS X-Ray compatible tracing
- Structured logging throughout applications

## Working with Different Components

### Tutorials (01-tutorials/)
Jupyter notebooks demonstrating specific AgentCore capabilities. To work with these:
1. Start Jupyter: `uv run --with jupyter jupyter lab`
2. Navigate to specific tutorial directory
3. Follow README instructions for prerequisites
4. Run notebooks cell by cell

### Use Cases (02-use-cases/)  
Complete applications with deployment scripts. Typical workflow:
1. Review project README for specific requirements
2. Run `./setup.sh` for initial setup
3. Configure AWS credentials and region
4. Deploy with `./deploy.sh`
5. Test functionality
6. Cleanup with `./cleanup.sh`

### Integrations (03-integrations/)
Framework-specific examples. Each subdirectory contains:
- Framework-specific agent implementation
- Requirements file with framework dependencies
- README with integration instructions

## Common Issues and Solutions

### AWS Configuration
- Ensure AWS credentials are configured: `aws configure`
- Set AWS_REGION environment variable if not using us-east-1
- Verify Bedrock model access in your AWS account

### Docker Requirements
- Many examples require Docker or Finch to be running
- Integration tests often use containerized services
- Ensure sufficient disk space for container images

### Python Environment
- Use Python 3.10+ (required for most examples)
- Prefer `uv` for package management over pip
- Activate virtual environment before running commands

### Token Management
- Authentication tokens may expire during development
- Use refresh token scripts where provided
- Check token permissions for gateway access

This repository emphasizes hands-on learning through working examples rather than abstract documentation. Start with tutorials, then explore use cases that match your requirements.
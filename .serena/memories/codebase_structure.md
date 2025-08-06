# Codebase Structure and Navigation

## High-Level Repository Organization

### 01-tutorials/ - Interactive Learning
- **AgentCore-runtime/**: Runtime capability examples and hosting tutorials
- **AgentCore-gateway/**: Gateway tool integration examples
- **AgentCore-identity/**: Identity and access management tutorials
- **AgentCore-memory/**: Memory management examples
- **AgentCore-tools/**: Built-in tools (Code Interpreter, Browser) tutorials
- **AgentCore-observability/**: Monitoring and tracing examples
- **end-to-end-example/**: Combined capability demonstrations

### 02-use-cases/ - Production Applications
- **AWS-operations-agent/**: AWS operations automation
- **SRE-agent/**: Site reliability engineering (multi-agent, advanced)
- **customer-support-assistant/**: Support ticket management
- **asynchronous-shopping-assistant/**: Background shopping tasks
- **device-management-agent/**: IoT device management
- **healthcare-appointment-agent/**: Healthcare scheduling
- **DB-performance-analyzer/**: Database performance analysis
- **text-to-python-ide/**: Code generation and execution
- **video-games-sales-assistant/**: Sales data analysis
- **local-prototype-to-agentcore/**: Migration from local to cloud

### 03-integrations/ - Framework Examples
- **agentic-frameworks/**: 
  - strands-agents/: Strands framework integration
  - langgraph/: LangGraph multi-agent workflows
  - openai-agents/: OpenAI agent patterns
  - autogen/: AutoGen conversation agents
  - llamaindex/: RAG and data-aware agents
  - adk/: Agent Development Kit examples
- **bedrock-agent/**: Amazon Bedrock Agent integration

### 04-my-projects/ - User Workspace
- User-created projects and examples
- Currently contains agentcore-with-confluence-mcp example

## Key File Patterns

### Entry Points
- Main application files typically named:
  - `main.py`: FastAPI or direct entry points
  - `app.py`: BedrockAgentCoreApp applications
  - `agent.py`: Agent implementation files
  - CLI entry points in `pyproject.toml` scripts section

### Configuration Files
- `requirements.txt`: Python dependencies (repository-wide)
- `pyproject.toml`: Python project configuration (project-specific)
- `Makefile`: Quality assurance commands (advanced projects)
- `config/static-config.yaml`: Base configuration
- `config/dynamic-config.yaml`: Deployment configuration

### Scripts and Automation
- `setup.sh`: Project initialization
- `start.sh`: Development server startup
- `deploy.sh`: AWS deployment
- `cleanup.sh`: Resource cleanup
- `build_and_deploy.sh`: Build and deployment automation

### Documentation Structure
- `README.md`: Project overview and setup instructions
- `README-JA.md`: Japanese documentation
- `docs/`: Detailed documentation (for complex projects)
  - `system-components.md`
  - `configuration.md`
  - `deployment-guide.md`
  - `security.md`

### Testing Structure
- `tests/` or `test/`: Test files
- `test_*.py`: pytest test files
- Integration test scripts: `test-*.sh`

## Common Import Patterns
```python
# AgentCore imports
from bedrock_agentcore.runtime import BedrockAgentCoreApp

# Framework imports
from strands import Agent
from langgraph import StateGraph
from langchain_aws import BedrockLLM

# AWS imports
import boto3
from boto3 import Session

# Utility imports
import yaml  # Configuration loading
import logging  # Structured logging
from opentelemetry import trace  # Observability
```

## Navigation Tips
1. **Start with README.md** in each directory for overview
2. **Check for Makefile or pyproject.toml** to understand project structure
3. **Look for app.py or main.py** for entry points
4. **Review config/** directory for configuration patterns
5. **Check scripts/** or root-level *.sh files** for automation
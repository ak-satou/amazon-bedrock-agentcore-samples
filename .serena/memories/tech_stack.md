# Technology Stack

## Core Dependencies
- **bedrock-agentcore**: Core AgentCore SDK
- **strands-agents**: Primary agent framework used in examples
- **langgraph**: Multi-agent workflow framework
- **langchain[aws]**: LangChain with AWS integrations
- **boto3**: AWS SDK for Python
- **opentelemetry-instrumentation-langchain**: Observability and tracing

## Agent Frameworks Supported
- **Strands Agents**: Primary framework for agent development
- **LangGraph**: Multi-agent workflow orchestration
- **LangChain**: Chain-based agent construction
- **CrewAI**: Multi-agent collaboration
- **OpenAI Agents**: OpenAI-based agent patterns
- **AutoGen**: Multi-agent conversation framework
- **LlamaIndex**: RAG and data-aware agents

## Development Tools
- **Python 3.10+**: Required runtime version
- **uv**: Recommended package manager (high-speed Python package manager)
- **Docker/Finch**: Required for containerized deployments
- **Jupyter Lab**: For tutorial notebooks
- **AWS CLI**: For AWS service integration

## Code Quality Tools
- **black**: Code formatting (line length 88)
- **ruff**: Fast Python linting
- **mypy**: Static type checking  
- **bandit**: Security vulnerability scanning
- **pytest**: Testing framework

## Infrastructure & Deployment
- **AWS Bedrock**: LLM model hosting
- **AWS Lambda**: Serverless compute
- **Amazon EC2**: For demo environments
- **Docker**: Containerization
- **SSL certificates**: Required for HTTPS endpoints (production deployments)

## Protocols & Standards
- **MCP (Model Context Protocol)**: Tool integration standard
- **OpenTelemetry**: Observability and tracing
- **OAuth2/OIDC**: Authentication and authorization
- **REST APIs**: Service communication
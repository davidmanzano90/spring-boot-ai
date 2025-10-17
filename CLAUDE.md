# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a Spring Boot AI project demonstrating an AI agent application with two main components:

### MCP Server (`mcp-server/`)
- Spring Boot WebFlux application providing MCP (Model Context Protocol) tools
- Implements `BookingTool` for accommodation reservations
- Runs on port 8081 by default
- Uses Spring AI's `@Tool` and `@ToolParam` annotations for tool definitions

### Chat Server (`chat-server/`)
- Main Spring Boot Web application serving the AI chat interface
- Integrates multiple AI capabilities:
  - **MCP Tools**: Local (`WeatherTool`, `ClockTool`) and remote (`BookingTool` via MCP client)
  - **RAG**: Uses PGVector for document retrieval with city information
  - **Chat Memory**: In-memory conversation history per chat session
  - **Spring AI Chat Client**: Configured with Ollama models and advisors
- REST API endpoints for chat interactions
- Swagger UI available at `/swagger-ui.html`

### Key Components
- **ChatClient**: Spring AI client with system prompt and tool callbacks
- **ChatService**: Orchestrates chat with advisors (QuestionAnswerAdvisor, PromptChatMemoryAdvisor, SimpleLoggerAdvisor)
- **ChatController**: REST endpoint handling chat requests
- **Vector Store Initializer**: Loads city data into PGVector on startup

## Development Commands

### Build and Test
```bash
# Build MCP server
cd mcp-server && ./gradlew build

# Build chat server
cd chat-server && ./gradlew build

# Run tests
cd mcp-server && ./gradlew test
cd chat-server && ./gradlew test
```

### Running the Application
```bash
# 1. Start MCP server
cd mcp-server && ./gradlew bootRun

# 2. Start infrastructure (PostgreSQL + Ollama)
cd chat-server && docker compose up -d

# 3. Start chat server
cd chat-server && ./gradlew bootRun
```

### Testing the API
The chat server exposes a REST API at `http://localhost:8080`:
```bash
curl -X POST "http://localhost:8080/{chatId}/chat" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "question=Your question here"
```

## Configuration

### Models and Profiles
- Default profile: `ollama` (configured in `application-ollama.yml`)
- Ollama models: `llama3.1:8b` (chat), `nomic-embed-text` (embeddings)
- Alternative profiles available (e.g., `bedrock` branch for AWS Bedrock)

### Database
- PostgreSQL with PGVector extension for vector storage
- 768 dimensions matching the embedding model
- Auto-schema initialization enabled

### MCP Configuration
- MCP client connects to booking tool via SSE at `http://localhost:8081`
- Local tools registered as `MethodToolCallbackProvider` beans

## Testing Strategy

### MCP Server Tests
- Uses `McpClient` with `HttpClientSseClientTransport` for integration testing
- Mocks downstream services with `@MockitoBean`

### Chat Server Tests
- **AI Model Evaluation**: Uses the AI model itself to evaluate responses (self-evaluation approach)
- Custom `TestEvaluator` for response validation
- Testcontainers for Ollama and PostgreSQL integration
- Disabled MCP client in tests, uses local test tools instead
- Fixed `Clock` bean for deterministic date handling

### CI Configuration
- Separate workflows for each module (`ci-chat-server.yml`, `ci-mcp-server.yml`)
- Ollama model caching for faster CI builds
- Minimal test suite in CI due to performance constraints
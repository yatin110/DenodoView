# DenodoView


You are helping me build a new application called "Vue Pilot".

This is the FIRST implementation task. Assume you know nothing about the application other than what is written below.

Before making any changes:
1. Inspect the entire existing repository.
2. Understand what files, code, configuration, and dependencies already exist.
3. Do not delete or rewrite useful working code without a clear reason.
4. If the repository is mostly empty, create the structure described below.
5. If an equivalent structure already exists, adapt it rather than creating duplicate architecture.

============================================================
PRODUCT OVERVIEW
============================================================

Vue Pilot is an enterprise conversational AI application focused initially on Denodo.

The user should eventually be able to open Vue Pilot and have a conversation with an AI assistant about Denodo data, metadata, views, schemas, relationships, queries, lineage, documentation, and other information available through connected tools.

Example future questions might include:

- "What views contain counterparty information?"
- "Explain this Denodo view to me."
- "What is the source of this field?"
- "Which views depend on this base view?"
- "Find a view containing trade date and legal entity."
- "Why might this query be slow?"
- "Show me the metadata for this view."
- "Which fields can I use to calculate exposure?"
- "Compare these two views."
- "Where does this column originate from?"

Vue Pilot is NOT intended to be just a simple LLM prompt box.

It will eventually have an orchestration layer that can:

1. Understand the user's intent.
2. Determine which information is required.
3. Decide which available tool or data source should be used.
4. Call one or more tools.
5. Inspect tool results.
6. Potentially make additional tool calls.
7. Build an evidence-backed response.
8. Stream useful progress information to the UI.
9. Return a final natural-language answer.

Denodo will initially be one of the primary information sources, potentially accessed through MCP or another integration mechanism.

However, the architecture MUST NOT hard-code the application around Denodo or MCP.

In the future Vue Pilot may consume:

- Denodo MCP tools
- Denodo REST/API services
- metadata databases
- documentation repositories
- internal APIs
- knowledge bases
- Git repositories
- other MCP servers
- search services
- future enterprise tools

Therefore integrations must eventually be pluggable.

Another important requirement is persistent chat.

Each authenticated user will eventually have:

- their own conversations
- conversation history
- messages belonging to each conversation
- user-specific settings
- possibly saved/favourite conversations

A user must never be able to access another user's conversations.

Persistent conversation storage will be implemented in a later task. For now, design the architecture so this can be added cleanly.

============================================================
HIGH-LEVEL USER EXPERIENCE
============================================================

The target experience should eventually resemble modern AI assistants such as ChatGPT or GitHub Copilot Chat.

The main application will eventually contain:

LEFT SIDE
- application logo/name: Vue Pilot
- New Chat button
- conversation history
- potentially grouped by date
- user/settings section

MAIN AREA
- current conversation
- user messages
- assistant messages
- rich Markdown rendering
- code blocks
- tables
- citations/source references
- tool/activity indicators
- streaming response
- thinking/progress status where appropriate

BOTTOM
- message input
- send button
- future attachment/tool options

Do NOT build all of this functionality now.

This first task is about establishing the correct architecture and project foundation so future prompts can implement individual capabilities safely.

============================================================
TECHNOLOGY DIRECTION
============================================================

Unless the repository already contains an equivalent sensible stack, use the following architecture.

FRONTEND

Use:

- React
- TypeScript
- Vite
- modern component-based architecture
- React Router if routing is required
- a clean API client/service layer
- no business logic directly embedded inside UI components

Do not over-engineer the UI framework at this stage.

The project should make it easy to introduce a component library later if needed.

BACKEND

Use:

- Python 3.12+
- FastAPI
- Pydantic
- pydantic-settings for configuration
- async APIs wherever appropriate

DATABASE

Design the persistence layer so:

Development:
- SQLite can be used easily.

Production:
- PostgreSQL can be used without redesigning the application.

Use:

- SQLAlchemy 2.x
- Alembic migrations

Do not couple application services directly to SQLite-specific behaviour.

AI / AGENT LAYER

Do NOT implement the actual AI agent yet.

However create clear architectural boundaries where these future concepts will live:

- LLM provider
- agent/orchestrator
- tools
- data sources
- conversation context
- prompt management
- source/evidence tracking

The future application may use different LLM providers.

Therefore do NOT tightly couple the application to OpenAI, Azure OpenAI, GitHub Copilot, or any single LLM provider.

There should eventually be an interface such as:

LLMProvider

with implementations added later.

Similarly external integrations should eventually follow abstractions such as:

Tool
ToolRegistry
DataSource
DataSourceRegistry

Do not implement unnecessary framework complexity now. Establish sensible extension points only.

============================================================
PROPOSED REPOSITORY STRUCTURE
============================================================

Use a clean monorepo-style structure similar to:

/
  README.md
  .gitignore
  .env.example

  backend/
    pyproject.toml

    app/
      __init__.py
      main.py

      api/
        __init__.py
        router.py
        v1/

      core/
        config.py
        logging.py
        exceptions.py

      auth/
        __init__.py

      models/
        __init__.py

      schemas/
        __init__.py

      repositories/
        __init__.py

      services/
        __init__.py

      db/
        __init__.py
        base.py
        session.py

      ai/
        __init__.py
        providers/
        orchestration/
        context/

      tools/
        __init__.py

      integrations/
        __init__.py
        denodo/

    tests/

    alembic/
    alembic.ini

  frontend/
    package.json
    tsconfig.json

    src/
      main.tsx
      App.tsx

      api/
      components/
      features/
        chat/
        auth/
      hooks/
      layouts/
      pages/
      routes/
      services/
      types/
      utils/

    public/

  docs/
    architecture.md
    development.md

You may adjust this structure if there is a strong technical reason, but document the reason.

Avoid an excessive number of tiny abstraction layers.

============================================================
ARCHITECTURAL PRINCIPLES
============================================================

Follow these principles throughout the project.

1. Separation of concerns

Frontend, API, service logic, persistence, AI orchestration, and integrations should be clearly separated.

2. Dependency direction

API routes should not contain major business logic.

Prefer:

API route
    -> service
        -> repository/integration

rather than:

API route
    -> database/API directly.

3. Provider independence

Avoid hard-coding:

- a particular LLM
- Denodo
- MCP
- a database vendor

into core business logic.

4. Async-friendly

The architecture must support long-running AI/tool operations later.

5. Streaming-ready

The backend should eventually be capable of streaming events/results to the frontend.

Do not implement full AI streaming now, but avoid architecture that would prevent Server-Sent Events or another appropriate streaming mechanism later.

6. Testability

Services and integrations should be testable independently.

External systems must eventually be mockable.

7. Security

Never put secrets in source code.

Use environment-based configuration.

Provide .env.example containing variable names only and harmless example values.

8. Observability

Design for structured application logging.

Do not log:

- passwords
- authentication tokens
- secrets
- sensitive conversation data unnecessarily

9. Error handling

Establish a consistent pattern for:

- API errors
- validation errors
- unexpected server errors

Do not expose Python stack traces to API consumers.

10. API versioning

Use:

/api/v1/...

for application APIs.

============================================================
CODING STANDARDS
============================================================

PYTHON

Use:

- type hints
- modern Python syntax
- async/await where beneficial
- small focused functions
- meaningful naming
- docstrings where they add value

Prefer dependency injection over global state.

Do not use broad:

except Exception:

unless it is at a clearly defined application boundary where errors are logged and transformed appropriately.

FRONTEND

Use:

- TypeScript strict mode
- functional React components
- typed API responses
- reusable components
- custom hooks where appropriate

Avoid:

- "any" unless absolutely necessary
- API calls scattered directly throughout UI components
- very large React components

GENERAL

Avoid:

- unnecessary complexity
- premature microservices
- unnecessary design patterns
- placeholder implementations pretending to work
- hard-coded credentials
- hard-coded URLs
- duplicated constants

============================================================
CONFIGURATION
============================================================

Create a centralized backend configuration approach.

Prepare configuration categories for future use such as:

APP_NAME
APP_ENV
DEBUG
API_PREFIX
DATABASE_URL
LOG_LEVEL
FRONTEND_ORIGIN

Also leave room for future values such as:

AUTH_PROVIDER
OIDC settings
LLM_PROVIDER
MCP settings
DENODO settings

Do not require those future settings yet.

Frontend API base URLs should also come from environment configuration.

============================================================
DOCUMENTATION
============================================================

Create:

docs/architecture.md

It should explain:

- purpose of Vue Pilot
- architecture overview
- frontend/backend responsibilities
- persistence approach
- future AI orchestration layer
- tool/integration abstraction
- expected request flow
- directory structure
- key architectural principles

Also update README.md with:

- what Vue Pilot is
- prerequisites
- backend startup instructions
- frontend startup instructions
- basic development workflow

============================================================
WHAT TO IMPLEMENT IN THIS TASK
============================================================

Implement ONLY the project foundation.

Create:

1. Repository/folder structure.
2. Backend FastAPI skeleton.
3. Frontend React/TypeScript skeleton.
4. Backend configuration system.
5. Basic structured logging setup.
6. Database infrastructure skeleton.
7. Initial Alembic setup if appropriate.
8. AI/integration extension-point folders/interfaces where useful.
9. Documentation.
10. Development startup instructions.

The frontend may simply display a clean placeholder Vue Pilot page proving the frontend works.

The backend should at minimum start successfully.

Do NOT yet implement:

- authentication
- users
- persistent conversations
- chat functionality
- Denodo connectivity
- MCP connectivity
- LLM calls
- agent orchestration
- production UI
- tool calling

Those will be separate implementation stages.

============================================================
VALIDATION
============================================================

After implementing:

1. Verify the Python application imports successfully.
2. Verify FastAPI starts.
3. Verify the frontend TypeScript build succeeds.
4. Verify configuration loads correctly.
5. Verify there are no obvious circular dependencies.
6. Run any tests you create.
7. Fix errors you encounter rather than merely documenting them.

============================================================
FINAL RESPONSE
============================================================

After completing the work, give me:

1. A concise summary of what you created.
2. The resulting directory tree.
3. Important architectural decisions.
4. Files created or modified.
5. Commands for running backend and frontend.
6. Tests/checks that were executed.
7. Any assumptions made.
8. Anything that needs to be addressed in the NEXT implementation stage.

Do not start implementing future stages.





















# Vibe-Trading Architecture Summary

## Overview

Vibe-Trading is a comprehensive AI-powered trading agent system that combines financial data analysis, machine learning, and automated trading capabilities. The project supports multiple markets including A-shares (China), US stocks, and Hong Kong stocks.

**Version:** 0.1.10  
**Core Technologies:** Python 3.11+, FastAPI, React 19, LangChain/LangGraph

## Project Structure

```
Vibe-Trading/
├── agent/                      # Main Python package
│   ├── src/                    # Core source code
│   │   ├── agent/              # Agent orchestration
│   │   ├── api/                # API layer
│   │   ├── config/             # Configuration management
│   │   ├── core/               # Core business logic
│   │   ├── factors/            # Factor analysis
│   │   ├── goal/               # Goal management
│   │   ├── hypotheses/         # Hypothesis testing
│   │   ├── live/               # Live trading
│   │   ├── memory/             # Memory management
│   │   ├── providers/          # Provider integrations
│   │   ├── scheduled_research/ # Scheduled research tasks
│   │   ├── security/           # Security features
│   │   ├── session/            # Session management
│   │   ├── shadow_account/     # Shadow trading accounts
│   │   ├── skills/             # Agent skills
│   │   ├── swarm/              # Swarm intelligence
│   │   ├── tools/              # Trading tools
│   │   ├── trading/            # Trading logic
│   │   ├── market_data.py      # Market data handling
│   │   ├── preflight.py        # Pre-flight checks
│   │   └── ui_services.py      # UI services
│   ├── backtest/               # Backtesting engine
│   ├── cli/                    # Command-line interface
│   ├── scripts/                # Utility scripts
│   ├── skills/                 # Skill definitions
│   └── tests/                  # Test suite
├── frontend/                   # React 19 frontend
│   └── src/
│       ├── components/         # UI components
│       ├── hooks/              # Custom React hooks
│       ├── i18n/               # Internationalization
│       ├── pages/              # Page components
│       ├── stores/             # State management
│       └── types/              # TypeScript types
├── docs/                       # Documentation
├── tools/                      # Development tools
└── wiki/                       # Wiki/documentation site
```

## Technology Stack

### Backend
- **Language:** Python 3.11+
- **Web Framework:** FastAPI with Uvicorn
- **AI Orchestration:** LangChain >=1.0.0, LangGraph >=1.0.10
- **Data Processing:** pandas, DuckDB
- **Machine Learning:** scikit-learn
- **CLI Framework:** Rich

### Frontend
- **Framework:** React 19
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Build Tool:** Vite
- **State Management:** Zustand (stores/)
- **Internationalization:** i18next

### External Integrations
- **MCP Server:** fastmcp >=2.14.0
- **Data Sources:** 18 connectors, 68 tools
  - tushare (A-shares)
  - yfinance (US stocks)
  - akshare (multi-market)
  - ccxt (cryptocurrency)
  - Plus 14 additional read-only data tools

## Core Components

### 1. Agent System (`agent/src/agent/`)

The agent orchestration layer manages AI-driven trading decisions:

- **context.py:** Agent context management
- **frontmatter.py:** Document processing
- **loop.py:** Main agent loop
- **memory.py:** Agent memory management
- **progress.py:** Progress tracking
- **skills.py:** Skill registration and execution
- **tools.py:** Tool integration
- **trace.py:** Execution tracing

### 2. Trading Engine (`agent/src/trading/`)

Handles trading operations and market interactions:

- **connectors/:** Market data connectors
- **profiles.py:** Trading profiles
- **service.py:** Trading service
- **types.py:** Trading type definitions

### 3. ç (`agent/backtest/`)

Comprehensive backtesting capabilities:

- **benchmark.py:** Benchmark comparisons
- **correlation.py:** Correlation analysis
- **engines/:** Backtesting engines
- **loaders/:** Data loaders
- **metrics.py:** Performance metrics
- **models.py:** Backtesting models
- **optimizers/:** Portfolio optimizers
- **validation.py:** Strategy validation

### 4. CLI Interface (`agent/cli/`)

Rich command-line interface with subcommands:

- **main.py:** Main CLI entry point
- **completer.py:** Command completion
- **input.py:** Input handling
- **intro.py:** Introduction sequence
- **onboard.py:** Onboarding flow
- **stream.py:** Streaming output
- **theme.py:** Theme management
- **commands/:** CLI command implementations
- **components/:** CLI components
- **ui/:** CLI UI elements
- **utils/:** CLI utilities

### 5. API Server (`agent/api_server.py`)

FastAPI-based REST API server for external integrations.

### 6. MCP Server (`agent/mcp_server.py`)

Model Context Protocol server for AI agent communication.

## Frontend Architecture

The React 19 frontend provides a web interface for monitoring and controlling the trading agent:

```
frontend/src/
├── components/    # Reusable UI components
├── hooks/         # Custom React hooks
├── i18n/          # Internationalization files
├── pages/         # Page components
├── stores/        # Zustand state stores
├── types/         # TypeScript type definitions
├── lib/           # Utility libraries
├── main.tsx       # Application entry point
└── router.tsx     # Routing configuration
```

## Key Features

- **Multi-Market Support:** A-shares, US stocks, Hong Kong stocks, cryptocurrency
- **18 Data Connectors:** Comprehensive market data access
- **68 Trading Tools:** Extensive toolset for analysis and execution
- **AI-Powered Analysis:** LangChain/LangGraph orchestration
- **Shadow Trading:** Risk-free strategy testing
- **Scheduled Research:** Automated research tasks
- **Swarm Intelligence:** Collaborative agent decision-making
- **Security Features:** Built-in security measures
- **Internationalization:** Multi-language support

## Package Dependencies

### Core Dependencies (40+)
- langchain >=1.0.0,<2
- langgraph >=1.0.10,<1.1
- fastapi
- uvicorn
- pandas
- duckdb
- scikit-learn
- rich (CLI)
- fastmcp >=2.14.0

### Optional Dependencies
- ibkr (Interactive Brokers integration)
- deepseek (LangChain adapter)
- tushare (A-share data)
- yfinance (US market data)
- akshare (multi-market data)
- ccxt (cryptocurrency)

## CLI Commands

The project provides two main CLI tools:

1. **vibe-trading:** Main trading agent interface
2. **vibe-trading-mcp:** MCP server interface

## Configuration

- **pyproject.toml:** Project configuration and dependencies
- **.env.example:** Environment variable template
- **docker-compose.yml:** Container orchestration
- **Dockerfile:** Container build configuration

## Testing

Comprehensive test suite in `agent/tests/` with 50+ test files covering:
- Agent loop functionality
- Data loaders (akshare, alphavantage, etc.)
- API integration
- Security features
- Backtesting validation
- Tool calls

## Development

- **Build System:** Vite (frontend), standard Python packaging
- **Testing:** pytest
- **Containerization:** Docker/Docker Compose
- **Documentation:** Markdown, Wiki site

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         CLI Interface                        │
│                    (Rich-based, Subcommands)                 │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                      Agent Core                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Agent     │  │  Trading   │  │  Backtest  │            │
│  │  Loop      │  │  Engine    │  │  Framework │            │
│  └────────────┘  └────────────┘  └────────────┘            │
│                           │                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Memory    │  │  Skills    │  │  Tools     │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                    Data Connectors                           │
│  ┌─────────┐ ┌────────┐ ┌───────┐ ┌──────┐ ┌───────────┐  │
│  │ Tushare │ │YFinance│ │Akshare│ │CCXT  │ │ 14 More  │  │
│  └─────────┘ └────────┘ └───────┘ └──────┘ └───────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                    External Markets                          │
│              A-Shares │ US │ HK │ Crypto │ ...              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    API Layer                                 │
│  ┌────────────┐  ┌────────────┐                            │
│  │ FastAPI    │  │  MCP       │                            │
│  │ Server     │  │  Server    │                            │
│  └────────────┘  └────────────┘                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                    React 19 Frontend                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ Components │  │  Stores    │  │  Pages     │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
```

## Conclusion

Vibe-Trading is a comprehensive, enterprise-grade trading AI system with sophisticated agent orchestration, extensive market data integration, and robust backtesting capabilities. The architecture separates concerns clearly between agent logic, trading operations, backtesting, and user interface, enabling modular development and maintenance.

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
│   │   ├── channels/           # Messaging channel adapters (24+)
│   │   ├── channelsui/         # Channel UI gateway services
│   │   ├── config/             # Configuration management
│   │   ├── core/               # Core business logic
│   │   ├── factors/            # Factor analysis (with factor zoo)
│   │   ├── goal/               # Goal management
│   │   ├── hypotheses/         # Hypothesis testing
│   │   ├── live/               # Live trading
│   │   ├── memory/             # Memory management
│   │   ├── providers/          # LLM provider integrations
│   │   ├── scheduled_research/ # Scheduled research tasks
│   │   ├── security/           # Security features
│   │   ├── session/            # Session management
│   │   ├── shadow_account/     # Shadow trading accounts
│   │   ├── skills/             # Agent skills (79 skills)
│   │   ├── swarm/              # Swarm intelligence (29 presets)
│   │   ├── tools/              # Trading tools (51 modules)
│   │   ├── trading/            # Trading logic (11 connectors)
│   │   ├── market_data.py      # Market data handling
│   │   ├── preflight.py        # Pre-flight checks
│   │   └── ui_services.py      # UI services
│   ├── backtest/               # Backtesting engine
│   ├── cli/                    # Command-line interface
│   ├── scripts/                # Utility scripts
│   ├── skills/                 # Global skill definitions
│   └── tests/                  # Test suite (223+ files)
├── frontend/                   # React 19 frontend
│   └── src/
│       ├── components/         # UI components
│       ├── hooks/              # Custom React hooks
│       ├── i18n/               # Internationalization (5 locales)
│       ├── lib/                # Utility libraries
│       ├── pages/              # Page components
│       ├── stores/             # Zustand state management
│       ├── tests/              # Test helpers
│       ├── types/              # TypeScript type definitions
│       ├── main.tsx            # Application entry point
│       └── router.tsx          # Routing configuration
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
- **Data Sources:** 23 backtest data loaders + 11 trading connectors
  - tushare (A-shares)
  - yfinance (US stocks)
  - akshare (multi-market)
  - ccxt (cryptocurrency)
  - plus 19 additional backtest data loaders (Alphavantage, Baostock, EastMoney, Finnhub, Tiingo, etc.)
- **Trading Connectors:** 11 broker integrations
  - Alpaca, Binance, Dhan, Futu, IBKR, Longbridge, OKX, Robinhood, Shoonya, Tiger, Trading212

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

### 3. Backtesting Framework (`agent/backtest/`)

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

### 7. Messaging Channels (`agent/src/channels/`)

Multi-platform IM (Instant Messaging) channel adapters for receiving commands and sending notifications:

- **24+ channel adapters:** DingTalk, Discord, Email, Feishu, Matrix, MoChat, MS Teams, NapCat, QQ, Slack, Telegram, WeCom, Weixin, WhatsApp
- **bus/:** Event bus and queue for async message processing
- **pairing/:** IM sender pairing store
- **signal.py:** Channel signal handling
- **websocket.py:** WebSocket channel support

### 8. Factor Analysis (`agent/src/factors/`)

Quantitative factor research system with built-in factor zoo:

- **zoo/:** Factor libraries including academic factors, Alpha101, GTJA191, and Qlib158
- **registry.py:** Factor registration and discovery
- **bench_runner.py / compare_runner.py:** Factor benchmarking and comparison
- **factor_analysis_core.py:** Core factor analysis engine

### 9. Swarm Intelligence (`agent/src/swarm/`)

Collaborative multi-agent decision-making system:

- **29 preset team configurations** (commodity research, convertible bond, credit research, technical analysis, etc.)
- **runtime.py:** Swarm execution runtime
- **worker.py:** Individual swarm worker agent
- **grounding.py:** Fact-grounded decision making
- **store.py / task_store.py:** Swarm state and task persistence

### 10. LLM Providers (`agent/src/providers/`)

Multi-provider LLM integration layer:

- **llm.py:** Core LLM abstraction
- **llm_providers.json:** Provider configuration registry
- **chat.py:** Chat completion handling
- **capabilities.py:** Provider capability discovery
- **content_filter.py:** Content filtering
- **openai_codex.py:** OpenAI Codex integration

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
- **34 Data Sources:** 23 backtest data loaders + 11 trading broker connectors
- **51 Tool Modules:** Extensive toolset for analysis and execution
- **79 Agent Skills:** Domain-specific expertise modules
- **24+ Messaging Channels:** Multi-platform IM integration
- **AI-Powered Analysis:** LangChain/LangGraph orchestration
- **Shadow Trading:** Risk-free strategy testing
- **Scheduled Research:** Automated research tasks
- **Swarm Intelligence:** Collaborative multi-agent decision-making (29 presets)
- **Factor Analysis Zoo:** Academic, Alpha101, GTJA191, Qlib158 factor libraries
- **Security Features:** Built-in security measures
- **Internationalization:** Multi-language support (en, zh-CN, ja, ko, ar)

## Package Dependencies

### Core Dependencies (40+)

- langchain >=1.0.0,<2 / langchain-core / langchain-openai
- langgraph >=1.0.10,<1.1 / langgraph-checkpoint
- fastapi / uvicorn[standard]
- pandas / numpy / scipy / scikit-learn / duckdb
- tushare (A-share data)
- yfinance (US market data)
- akshare (multi-market data)
- ccxt (cryptocurrency)
- rich (CLI framework)
- fastmcp >=2.14.0
- httpx / websockets / pydantic / pydantic-settings
- Pillow / matplotlib / weasyprint
- jinja2 / python-dotenv / pyyaml
- prompt_toolkit / ddgs / sse-starlette
- openpyxl / python-docx / python-pptx / pypdfium2
- joblib / smartmoneyconcepts / python-multipart
- defusedxml / oauth-cli-kit

### Optional Dependencies

- ibkr (Interactive Brokers integration, `ib_async`)
- deepseek (LangChain DeepSeek adapter)
- ashare (baostock)
- harmonic (pyharmonics)
- Messaging channel adapters (dingtalk, discord, feishu, matrix, mochat, msteams, napcat, qq, slack, telegram, wecom, weixin, whatsapp)

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

Comprehensive test suite in `agent/tests/` with 223+ test files covering:

- Agent loop functionality
- Data loaders (akshare, alphavantage, etc.)
- API integration
- Security features
- Backtesting validation
- Tool calls
- Frontend components and hooks

## Development

- **Build System:** Vite (frontend), standard Python packaging
- **Testing:** pytest
- **Containerization:** Docker/Docker Compose
- **Documentation:** Markdown, Wiki site

## Architecture Diagram

```
┌───────────────────────────────────────────────────────────────────┐
│                       CLI Interface / Channels                     │
│          (Rich-based, Subcommands)  (24+ IM Adapters)             │
└────────────────────────────┬──────────────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────────────┐
│                         Agent Core                                │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌───────────┐  │
│  │  Agent     │  │  Trading   │  │  Backtest  │  │  Swarm    │  │
│  │  Loop      │  │  Engine    │  │  Framework │  │  (29      │  │
│  └────────────┘  └────────────┘  └────────────┘  │  Presets)  │  │
│                                                   └───────────┘  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐                  │
│  │  Memory    │  │  Skills    │  │  Tools     │                  │
│  │            │  │  (79)      │  │  (51 mods) │                  │
│  └────────────┘  └────────────┘  └────────────┘                  │
└────────────────────────────┬──────────────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────────────┐
│              Data Loaders & Trading Connectors                     │
│  ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌──────┐ ┌──────────┐  │
│  │ 23 Loaders│ │ 11 Trading│ │ 79 Skills│ │Factor│ │ 24+     │  │
│  │ (tushare, │ │Connectors │ │ (akshare,│ │ Zoo  │ │Channels │  │
│  │  yfinance,│ │(IBKR,     │ │  yfinance│ │      │ │(Slack,  │  │
│  │  akshare, │ │ Alpaca,   │ │  ...)    │ │      │ │Telegram)│  │
│  │  ccxt...) │ │ Binance..)│ │          │ │      │ │         │  │
│  └──────────┘ └───────────┘ └──────────┘ └──────┘ └──────────┘  │
└────────────────────────────┬──────────────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────────────┐
│                      External Markets                             │
│         A-Shares │ US │ HK │ Crypto │ Futures │ Forex │ ...      │
└───────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────┐
│                        API Layer                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │ FastAPI      │  │  MCP Server  │  │  LLM Providers         │  │
│  │ (REST API)   │  │  (Model      │  │  (OpenAI, DeepSeek,    │  │
│  │              │  │   Context)   │  │   etc.)                │  │
│  └──────────────┘  └──────────────┘  └────────────────────────┘  │
└────────────────────────────┬──────────────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────────────┐
│                      React 19 Frontend                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │
│  │ Components   │  │   Stores     │  │   Pages      │            │
│  │ (Chat,       │  │  (Zustand)   │  │  (Agent,     │            │
│  │  Charts,     │  │              │  │   Reports,   │            │
│  │  Layout)     │  │              │  │   Settings)  │            │
│  └──────────────┘  └──────────────┘  └──────────────┘            │
└───────────────────────────────────────────────────────────────────┘
```

## Conclusion

Vibe-Trading is a comprehensive, enterprise-grade trading AI system with sophisticated agent orchestration, extensive market data integration, and robust backtesting capabilities. The architecture separates concerns clearly between agent logic, trading operations, backtesting, and user interface, enabling modular development and maintenance.

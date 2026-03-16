# SQL-MCP-Server

An MCP (Model Context Protocol) server that connects Claude to a PostgreSQL database, providing both raw SQL access and pre-built analytics tools for manufacturing data insights.

Built with [FastMCP](https://github.com/jlowin/fastmcp) and designed to work with [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Prerequisites

- Python 3.10+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip
- PostgreSQL database running locally (or accessible remotely)

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/your-username/SQL-MCP-Server.git
cd SQL-MCP-Server
```

### 2. Install dependencies

**Using uv (recommended):**

```bash
uv sync
```

This reads `pyproject.toml` and creates a virtual environment with all dependencies automatically.

**Using pip (alternative):**

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. Configure the database connection

The server reads database credentials from environment variables. Copy the example file and fill in your values:

```bash
cp .env.example .env
```

Then edit `.env`:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=bhaiya_and_company
DB_USER=postgres
DB_PASSWORD=your_password_here
```

All variables have sensible defaults, so you only need to set the ones that differ from your setup.

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_HOST` | `localhost` | PostgreSQL host |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_NAME` | `bhaiya_and_company` | Database name |
| `DB_USER` | `postgres` | Database user |
| `DB_PASSWORD` | `root` | Database password |

> **Note:** Never commit `.env` to version control. Add it to `.gitignore`.

### 4. Test the server locally

```bash
uv run fastmcp run main.py
```

If the server starts without errors, it's ready to connect to Claude Code.

## Connecting to Claude Code

### Option A: Project-level config (recommended)

Create a `.mcp.json` file in your project root:

```json
{
  "mcpServers": {
    "sql_mcp": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/SQL-MCP-Server",
        "run",
        "fastmcp",
        "run",
        "main.py"
      ],
      "env": {
        "PYTHONUNBUFFERED": "1",
        "DB_HOST": "localhost",
        "DB_PORT": "5432",
        "DB_NAME": "bhaiya_and_company",
        "DB_USER": "postgres",
        "DB_PASSWORD": "your_password_here"
      }
    }
  }
}
```

> **Important:** Replace `/absolute/path/to/SQL-MCP-Server` with the actual absolute path to this project on your machine. On Windows use `\\` as path separators (e.g. `C:\\MyRepos\\SQL-MCP-Server`).
>
> The `env` block in `.mcp.json` is where you pass your database credentials. These are set as environment variables when Claude Code starts the MCP server.

### Option B: Global config

To make the server available in all your Claude Code sessions, add the same config to your global settings:

```bash
claude mcp add sql_mcp -- uv --directory /absolute/path/to/SQL-MCP-Server run fastmcp run main.py
```

### Verify the connection

After configuring, start Claude Code in the project directory:

```bash
claude
```

Then ask Claude to list available tools or run a query:

```
> list all tables in the database
> show me the OEE report for Q1 2024
```

## Available Tools

### Core Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `test_sql_query` | Execute any SQL query and return results | `query` |
| `get_table_schema` | Get column details for a table | `table_name` |
| `list_tables` | List all public tables in the database | none |

### Analytics / Insight Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `get_production_summary` | Total units produced, rejected, rejection rate, energy & material usage | `start_date`, `end_date` |
| `get_oee_report` | OEE (Availability x Performance x Quality) per machine | `start_date`, `end_date` |
| `get_sales_summary` | Revenue breakdown by product, region, customer, or month | `start_date`, `end_date`, `group_by` |
| `get_inventory_alerts` | Items that are critical or below reorder level | none |
| `get_downtime_analysis` | Downtime events grouped by category, machine, or root cause | `start_date`, `end_date`, `group_by` |
| `get_quality_report` | Quality scores and defect type breakdown | `start_date`, `end_date` |
| `get_operator_performance` | Operators ranked by output and rejection rate | `start_date`, `end_date` |
| `get_maintenance_summary` | Maintenance cost and frequency per machine | `start_date`, `end_date` |
| `get_top_customers` | Top customers by revenue | `limit`, `start_date`, `end_date` |
| `get_machine_status` | Current machine status with last 30 days production stats | none |

> All date parameters use `YYYY-MM-DD` format and default to the full year 2024.

## Database Schema

The server is designed for a manufacturing database with 13 tables:

```
customers          - Customer info (industry, location, type)
products           - Product catalog (PET bottles, glass bottles, etc.)
machines           - Machine registry (injection molding machines)
operators          - Operator profiles (experience, efficiency rating)
shifts             - Shift definitions (morning, afternoon, night)
materials          - Raw materials (PET resin, HDPE, glass, etc.)
suppliers          - Supplier details and ratings
productiondata     - Hourly production records (8,784 rows for 2024)
salesdata          - Sales transactions (820 orders for 2024)
qualitychecks      - Quality inspection results and defect types
downtimeevents     - Machine downtime events with root causes
maintenancerecords - Maintenance logs with costs and parts replaced
inventory          - Current stock levels and reorder alerts
```

## Example Prompts for Claude

```
# Production
"Give me a production summary for March 2024"
"What's the total rejection rate for Q1?"

# OEE
"Show OEE for all machines in 2024"
"Which machine has the best OEE?"

# Sales
"Break down sales by region for 2024"
"Show monthly revenue trend"
"Who are the top 5 customers?"

# Operations
"Are there any inventory alerts?"
"What caused the most downtime this year?"
"Show me the quality report for last quarter"
"Rank operators by performance"
"How much did we spend on maintenance per machine?"
"What's the current status of all machines?"
```

## Project Structure

```
SQL-MCP-Server/
├── main.py              # MCP server with all tool definitions
├── pyproject.toml       # Project metadata and dependencies (uv)
├── requirements.txt     # Dependencies (pip)
├── uv.lock              # Lock file (auto-generated by uv)
├── .mcp.json            # Claude Code MCP configuration
├── .env.example         # Example environment variables
├── .python-version      # Python version pin (3.10)
└── README.md
```

## License

MIT

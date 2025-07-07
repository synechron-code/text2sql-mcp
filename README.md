# Synechron Text2SQL MCP Server

Synechron's Text2SQL MCP Server is a [Model Context Protocol](https://modelcontextprotocol.io/introduction) (MCP)
server. It provides MCP clients like Cursor, VSCode, and Claude Desktop, with natural language access to relational
databases of various types – enabling users to ask questions of their data using natural language capabilities.

The server optimizes the quality of the results it generates by using a combination of schema and row-aware semantic
search. Schemas and a sample of rows are indexed, enabling the server to more effectively generate suitable queries.

## Features and Capabilities

The Text2SQL MCP server provides the following capabilities:

- **Natural Language to SQL:** Natural language questions are converted into optimized SQL queries using advanced
  language models.
- **Multi-Database Support:** Compatibility with PostgreSQL, MySQL, SQLite and Microsoft Fabric databases.
- **Retrieval Augmented Generation (RAG):** Enhanced query generation by combining schema metadata and sampled row
  content.
- **Table Indexing:** Enables embedding for database schemas and sample data, improving query relevance.
- **Response Synthesis:** Generates natural language summaries of query results.
- **Markdown Formatting:** Creates more readable responses.
- **Multiple Model Support:** Works with OpenAI, Azure OpenAI, AWS Bedrock, and Ollama models.

## Sample Data

The server ships with two example datasets.

- Cyber Threats
- ESG Ratings

## Sample Client

A sample REPL is also included.

---

## Quick Start with Docker

> Note: To use your own databases, further configuration is required, see the [Configuration](#configuration) section
> below.

### Running the Server with Docker

```bash
docker run -it -p8000:8000 \
  -v ~/.aws:/home/appuser/.aws:ro \
  -e MODEL_API_TYPE=Bedrock \
  -e AWS_PROFILE=AWSAdministratorAccess-880502554482 \
  -e FASTMCP_HOST=0.0.0.0 \
  709825985650.dkr.ecr.us-east-1.amazonaws.com/synechron/mcp-text2sql:0.0.10-2025.07.04-rc3 server
````

### Running the Sample Client with Docker

```bash
docker run -it \
    -v ~/.aws:/home/appuser/.aws:ro \
    -e AWS_PROFILE=AWSAdministratorAccess-880502554482 \
    -e BEDROCK_MODEL_ID=us.anthropic.claude-3-7-sonnet-20250219-v1:0 \
    -e BEDROCK_API_VERSION=2025-01-01-preview \
    -e MCP_SERVER_HOST=host.docker.internal \
     709825985650.dkr.ecr.us-east-1.amazonaws.com/synechron/mcp-text2sql:0.0.10-2025.07.04-rc3 client
```

## Example Client Usage

```text
Connected to server with tools: ['Text2Sql_Cyber_Threats', 'Text2Sql_ESG_Ratings']

MCP Client Started!
Type your queries or 'quit' to exit.

Query: What kind of data is in the Cyber Threat datasource?

The Cyber Threat datasource contains comprehensive information about cybersecurity incidents with the following data fields:

1. country - The country where the cyber attack took place
2. year - The year when the cyber attack occurred
3. attack_type - The type of cyber attack (e.g., Phishing, Ransomware, DDoS, Man-in-the-Middle, SQL Injection)
4. target_industry - The industry that was targeted (e.g., Education, Retail, IT, Telecommunications, Healthcare, Government, Banking)
5. financial_loss_(in_million_$) - The financial impact of the attack in millions of dollars
6. number_of_affected_users - How many users were affected by the attack
7. attack_source - Where the attack originated from (e.g., Hacker Group, Nation-state, Insider, Unknown)
8. security_vulnerability_type - The type of security vulnerability exploited (e.g., Unpatched Software, Weak Passwords, Social Engineering)
9. defense_mechanism_used - What defense was in place (e.g., VPN, Firewall, Antivirus, AI-based Detection)
10. incident_resolution_time_(in_hours) - How long it took to resolve the incident in hours

The database tracks cybersecurity incidents across different countries, industries, and years, including details about attack methods, financial impact, affected users, vulnerabilities exploited, and resolution times.
```

## Using the Server with an Existing MCP Client

The server should be run with Docker. Use either of the following configurations to add it to a MCP client:

```json
{
  "servers": {
    "text2sql": {
      "type": "streamable-http",
      "url": "http://localhost:8000/mcp"
    }
  }
}
```

```json
{
  "mcpServers": {
    "text2sql": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--env-file",
        ".env",
        "709825985650.dkr.ecr.us-east-1.amazonaws.com/synechron/mcp-text2sql:0.0.10-2025.07.04-rc3",
        "server"
      ]
    }
  }
}
```

## Configuration

The Text2SQL MCP server can be configured with a combination of environment variables and JSON configuration.

### Server Configuration

| Valves Config               | Environment Variable  | Default                     | Description                                                                                                   |
|-----------------------------|-----------------------|-----------------------------|---------------------------------------------------------------------------------------------------------------|
| ENABLED                     |                       | False                       | Enable tool                                                                                                   |
| ENABLE_RAG                  |                       | True                        | Use AI to synthesize a response using data collected.                                                         |
| ENABLE_MARKDOWN             |                       | False                       | Format response in Markdown                                                                                   |
| ENABLE_PANDAS               |                       | True                        | Use pandas to execute and parse SQL query, otherwise use llama_index                                          |
| ENABLE_TABLE_INDEXING       |                       | True                        | Use table retriever to create embeddings for database schema and sample data to optimize context for LLM      |
| FORCE_REINDEXING            |                       | False                       | Force tool to regenerate vector indexes for tables and sample data                                            |
| DATABASE_CONFIG             | DB_CONFIG             | Default db config           | JSON formatted tool configuration with database connection string, tables, context, and prompts               |
| IGNORE_SCHEMA               |                       | Default system tables       | Comma separated list of schemas to ignore                                                                     |
| CREDENTIAL_SERVICE          | CREDENTIAL_SERVICE    |                             | Select authentication type (Azure, AWS, Google, None)                                                         |
| MODEL_API_TYPE / MODEL_TYPE | MODEL_TYPE            | OpenAI                      | Select model API type (OpenAI, Azure OpenAI, Bedrock, Ollama)                                                 |
| API_ENDPOINT                | OPENAI_ENDPOINT       | <http://litellm-proxy:4000> | OpenAI API endpoint                                                                                           |
| API_ID                      | API_ACCESS_ID         |                             | Access Key ID                                                                                                 |
| API_KEY                     | OPENAI_API_KEY        |                             | OpenAI API key                                                                                                |
| API_REGION                  | AWS_REGION            | us-east-1                   | Region for cloud-hosted models                                                                                |
| API_VERSION                 |                       | 2025-01-01-preview          | OpenAI API version                                                                                            |
| LLM_MODEL_NAME              |                       | gpt-4.1-mini                | Text-to-SQL model                                                                                             |
| EMBED_MODEL_NAME            |                       | text-embedding-3-small      | Embedding model                                                                                               |
| MAX_RETRY                   |                       | 3                           | Maximum number of retries for failing queries                                                                 |
| MAX_RESULTS                 |                       | 200                         | Maximum number of rows in a response                                                                          |
| MAX_DATAFRAME               |                       | 1000                        | Maximum number of rows in a dataframe sent to LLM to synthesize a response                                    |
| SAMPLE_RATIO                |                       | 0.0001                      | Percentage of dataset to index. Higher values will take longer to index but give better results.              |
| SAMPLE_MAX_SIZE             |                       | 50                          | Maximum number of rows to sample. Supersedes SAMPLE_RATIO                                                     |
| DEBUG                       |                       | False                       | Debug mode                                                                                                    |
| MODE                        | MODE                  | stdio                       | Mode in which the MCP server should operate. Use `shttp` for Streamable HTTP.                                 |
| AUTH_ENABLED                | AUTH_ENABLED          | false                       | Whether or not to require MCP clients to be authenticated.                                                    |
| AUTH_ISSUER                 | AUTH_ISSUER           |                             | Required if `AUTH_ENABLED` is `true`.                                                                         |
| AUTH_JWKS_URI               | AUTH_JWKS_URI         |                             | Required if `AUTH_ENABLED` is `true`.                                                                         |
| AWS_ACCESS_KEY_ID           | AWS_ACCESS_KEY_ID     |                             | Required if using AWS Bedrock for embeddings                                                                  |
| AWS_SECRET_ACCESS_KEY       | AWS_SECRET_ACCESS_KEY |                             | Required if using AWS Bedrock for embeddings                                                                  |
| LOG_LEVEL                   | LOG_LEVEL             | INFO                        | Logging level (DEBUG, INFO, WARNING, ERROR).                                                                  |
| MCP_SERVER_API_KEY          | MCP_SERVER_API_KEY    |                             | API Key for access to LLM / Embeddings. Configure with AWS_SECRET_ACCESS_KEY, LiteLLM key, or OpenAI API key. |
| MCP_SERVER_DATA             | MCP_SERVER_DATA       | data                        | Directory for temporary data files                                                                            |
| TEXT2SQL_VALVES_JSON        | TEXT2SQL_VALVES_JSON  | valves.json                 | Full path to configuration JSON, see above for example.                                                       |

### Example .env File

```env
LOG_LEVEL=INFO
PYTHONUNBUFFERED=1
TEXT2SQL_VALVES_JSON=/home/appuser/config/config.json

MODE=shttp
FASTMCP_HOST=0.0.0.0
FASTMCP_PORT=8000

AUTH_ENABLED=true
AUTH_JWKS_URI=https://cognito-idp.us-east-1.amazonaws.com/us-east-1_/.well-known/jwks.json
AUTH_ISSUER=https://cognito-idp.us-east-1.amazonaws.com/us-east-1_

AWS_PROFILE=default
```

### Example TEXT2SQL_VALVES_JSON (valves.json)

The Text2SQL MCP server must be configured using a JSON file that specifies database connections, model settings, and
other
parameters. Here's an example configuration:

```json
{
  "ENABLED": true,
  "ENABLE_RAG": false,
  "ENABLE_MARKDOWN": true,
  "ENABLE_PANDAS": true,
  "ENABLE_TABLE_INDEXING": true,
  "FORCE_REINDEXING": false,
  "DATABASE_CONFIG": {
    "Cyber_Threats": {
      "description": "A comprehensive dataset tracking cybersecurity incidents, attack vectors, threat types, and affected countries.",
      "topics": [
        "Cyber Threats",
        "Attacks",
        "Targets"
      ],
      "url": "sqlite:///file:/home/appuser/config/cyber_threats.db?uri=true",
      "tables": {
        "cyber_threats": "The Global Cybersecurity Threats Dataset (2015-2024) provides extensive data on cyberattacks, malware types, targeted industries, and affected countries."
      },
      "prompts": [
        {
          "name": "biggest_cyber_threat",
          "description": "Which type of cyber threat has caused the biggest financial loss?"
        }
      ]
    },
    "ESG_Ratings": {
      "description": "S&P 500 Companies ESG Insights & Risk Scores for Informed Decisions.",
      "topics": [
        "ESG Ratings",
        "ESG Risk",
        "Sustainability"
      ],
      "url": "sqlite:///file:/home/appuser/config/esg_ratings.db?uri=true",
      "tables": {
        "esg_ratings": "A comprehensive dataset tracking ESG ratings for S&P 500 companies."
      }
    }
  },
  "IGNORE_SCHEMA": "information_schema, INFORMATION_SCHEMA, _rsc, db_accessadmin, db_backupoperator, db_datareader, db_datawriter, db_ddladmin, db_denydatareader, db_denydatawriter, db_owner, db_securityadmin, guest, queryinsights, sys, pg_catalog",
  "MODEL_API_TYPE": "Bedrock",
  "AWS_PROFILE": "default",
  "API_VERSION": "2025-01-01-preview",
  "LLM_MODEL_NAME": "us.anthropic.claude-3-7-sonnet-20250219-v1:0",
  "EMBED_MODEL_NAME": "amazon.titan-embed-text-v2:0",
  "MAX_RETRY": 3,
  "MAX_RESULTS": 100,
  "MAX_DATAFRAME": 2000,
  "LOG_LEVEL": "INFO"
}
```

### Client Configuration

The following environment variables can be provided to the sample MCP client:

| Option                     | Default | Description                                                                                                                                                                                                                                       |
|----------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `JWT_ACCESS_TOKEN`         | `false` | Token used to authenticate client with server if the server has `AUTH_ENABLED` true                                                                                                                                                               |
| `MCP_SERVER_HOST`          |         | Host for MCP server. For a server running in Docker on the same host use `host.docker.internal`                                                                                                                                                   |
| `MCP_SERVER_PORT`          | `8000`  | Port for MCP server.                                                                                                                                                                                                                              |
| `BEDROCK_MODEL_ID`         |         | Required if using AWS Bedrock, e.g. `us.anthropic.claude-3-7-sonnet-20250219-v1:0`                                                                                                                                                                |
| `BEDROCK_API_VERSION`      |         | Required if using AWS Bedrock, e.g. `2023-06-01-preview`                                                                                                                                                                                          |
| `AWS_PROFILE`              |         | Required if using AWS Bedrock, e.g. `default`                                                                                                                                                                                                     |
| `AZURE_DEPLOYMENT_MODEL`   |         | Required if using Azure OpenAI. Ignored if `BEDROCK_MODEL_ID` is set. For Azure authentication see [DefaultAzureCredential](https://learn.microsoft.com/en-us/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python). |
| `AZURE_API_VERSION`        |         | Required If using Azure OpenAI. Ignored if `BEDROCK_MODEL_ID` is set.                                                                                                                                                                             |
| `OTEL_SDK_DISABLED`        | `false` | Enable telemetry for Crew.AI client.                                                                                                                                                                                                              |
| `CREWAI_DISABLE_TELEMETRY` | `false` | Enable telemetry for Crew.AI client.                                                                                                                                                                                                              |

## Tools

The Text2SQL MCP server dynamically creates MCP Tools and Prompts for the configured databases in the following format:

- **Text2Sql_DatabaseName**: Query a specific database using natural language
    - Parameters:
        - `query`: Natural language query to execute against the database

## License

This project is licensed under the MIT License.

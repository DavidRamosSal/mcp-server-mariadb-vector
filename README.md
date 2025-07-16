# Note:
**The MariaDB team took this Hackathon project and greatly improved on it, you can check it out at: https://github.com/MariaDB/mcp.**

**I leave this repo visible to showcase my work, link to a blog they wrote about it: https://mariadb.org/model-context-protocol-mcp-hackathon-integration-track-winner/**

# mcp-server-mariadb-vector

The MariaDB Vector MCP server provides tools that LLM agents can use to interact with a MariaDB database [with vector support](https://mariadb.org/projects/mariadb-vector/), providing users with a natural language interface to store and interact with their data. Thanks to the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction), this server is compatible with any MCP client, including those provided by applications like Claude Desktop and Cursor/Windsurf, as well as LLM Agent frameworks like LangGraph and PydanticAI.

Using the MariaDB Vector MCP server, users can for example:

- Provide context from a knowledge-base to their conversations with LLM agents
- Store and query their conversations with LLM agents

## Features

- **Vector Store Management**

  - Create and delete vector stores in a MariaDB database
  - List all vector stores in a MariaDB database

- **Document Management**

  - Add documents with optional metadata to a vector store
  - Query a vector store using semantic search

- **Embedding Provider**

  - Use OpenAI's embedding models to embed documents

## MCP Tools

- `mariadb_create_vector_store`: Create a vector store in a MariaDB database
- `mariadb_delete_vector_store`: Delete a vector store in a MariaDB database
- `mariadb_list_vector_stores`: List all vector stores in a MariaDB database
- `mariadb_insert_documents`: Add documents with optional metadata to a vector store
- `mariadb_search_vector_store`: Query a vector store using semantic search

## Setup

> **Note:** From here on, it is assumed that you have a running MariaDB instance with vector support (version **11.7** or higher). If you don't have one, you can quickly spin up a MariaDB instance using Docker:
>
> ```bash
> docker run -p 3306:3306 --name mariadb-instance -e MARIADB_ROOT_PASSWORD=password -e MARIADB_DATABASE=database_name mariadb:11.7
> ```

First clone the repository:

```bash
git clone https://github.com/DavidRamosSal/mcp-server-mariadb-vector.git
```

There are two ways to run the MariaDB Vector MCP server: as a Python package using **uv** or as a **Docker container** built from the provided Dockerfile.

### Requirements for running the server using uv

- MariaDB Connector/C - [installation instructions](https://mariadb.com/docs/server/connect/programming-languages/c/install)
- uv - [installation instructions](https://docs.astral.sh/uv/#installation)

### Requirements for running the server as a Docker container

- Docker - [installation instructions](https://docs.docker.com/get-docker/)

### Configuration

The server needs to be configured with the following environment variables:

| Name                 | Description                              | Default Value            |
| -------------------- | ---------------------------------------- | ------------------------ |
| `MARIADB_HOST`       | host of the running MariaDB database     | `127.0.0.1`              |
| `MARIADB_PORT`       | port of the running MariaDB database     | `3306`                   |
| `MARIADB_USER`       | user of the running MariaDB database     | None                     |
| `MARIADB_PASSWORD`   | password of the running MariaDB database | None                     |
| `MARIADB_DATABASE`   | name of the running MariaDB database     | None                     |
| `EMBEDDING_PROVIDER` | provider of the embedding models         | `openai`                 |
| `EMBEDDING_MODEL`    | model of the embedding provider          | `text-embedding-3-small` |
| `OPENAI_API_KEY`     | API key for OpenAI's platform            | None                     |

### Running the server using uv

Using uv, you can add a `.env` file to the root of the cloned repository with the environment variables and run the server with the following command:

```bash
uv run --dir path/to/mcp-server-mariadb-vector/ --env-file path/to/mcp-server-mariadb-vector/.env mcp_server_mariadb_vector
```

The dependencies will be installed automatically. An optional `--transport` argument can be added to specify the transport protocol to use. The default value is `stdio`.

### Running the server as a Docker container

Build the Docker container from the root directory of the cloned repository by running the following command:

```bash
docker build -t mcp-server-mariadb-vector .
```

Then run the container (replace with your own configuration):

```bash
docker run -p 8000:8000 \
  --add-host host.docker.internal:host-gateway \
  -e MARIADB_HOST="host.docker.internal" \
  -e MARIADB_PORT="port" \
  -e MARIADB_USER="user" \
  -e MARIADB_PASSWORD="password" \
  -e MARIADB_DATABASE="database" \
  -e EMBEDDING_PROVIDER="openai" \
  -e EMBEDDING_MODEL="embedding-model" \
  -e OPENAI_API_KEY="your-openai-api-key" \
  mcp-server-mariadb-vector
```

The server will be available at `http://localhost:8000/sse`, using the SSE transport protocol. **Make sure** to leave `MARIADB_HOST` set to `host.docker.internal` if you are running the MariaDB database as a Docker container on your host machine.

### Integration with Claude Desktop | Cursor | Windsurf

Claude Desktop, Cursor and Windsurf can run and connect to the server automatically using stdio transport. To do so, add the following to your configuration file (`claude_desktop_config.json` for Claude Desktop, `mcp.json` for Cursor or `mcp_config.json` for Windsurf):

```json
{
  "mcpServers": {
    "mariadb-vector": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "path/to/mcp-server-mariadb-vector/",
        "--env-file",
        "path/to/mcp-server-mariadb-vector/.env",
        "mcp-server-mariadb-vector"
      ]
    }
  }
}
```

Alternatively, Cursor and Windsurf can connect to an already running server on your host machine (e.g. if you are running the server as a Docker container) using SSE transport. To do so, add the following to the corresponding configuration file:

```json
  "mcpServers": {
    "mariadb-vector": {
      "url": "http://localhost:8000/sse"
    }
  }
}
```

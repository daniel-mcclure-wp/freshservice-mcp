# Freshservice MCP Server — Warby Parker Fork

[![smithery badge](https://smithery.ai/badge/@effytech/freshservice_mcp)](https://smithery.ai/server/@effytech/freshservice_mcp)

> **Warby Parker fork notice**
>
> This is a fork of [`forterro/freshservice_mcp`](https://github.com/forterro/freshservice_mcp) (which itself forks [`effytech/freshservice_mcp`](https://github.com/effytech/freshservice_mcp)) with two sets of patches:
>
> 1. **Gemini Enterprise schema compatibility** (`wp/gemini-schema-compat`) — Gemini's function-calling spec does not support `anyOf`, `oneOf`, or `allOf` JSON Schema constructs, which the upstream auto-generates from `Union[int, str]` and `Dict[str, Any]` parameter type hints. This fork tightens those types (`Union[int, str]` -> `int`, `Dict[str, Any]` -> `Dict[str, str]`) so registration succeeds and tools become callable from Gemini Enterprise Custom MCP Server data stores.
>
> 2. **`FORCE_API_KEY` opt-in** (`wp/api-key-fallback`) — when the `FORCE_API_KEY` env var is truthy (`true`/`1`/`yes`), the server always uses the env-var `FRESHSERVICE_APIKEY` for outbound calls and ignores any forwarded `Authorization` header. This is required when the upstream gateway forwards a Freshworks OAuth Bearer token, which is valid for embedded apps but rejected by the Freshservice REST API at `/api/v2/*`.
>
> Upstream is tracked via the `upstream` git remote and we rebase periodically. Patches are mechanical and contained to `src/freshservice_mcp/tools/*.py` parameter signatures and `src/freshservice_mcp/http_client.py`.

## Overview

A powerful MCP (Model Context Protocol) server that integrates with Freshservice, enabling AI models to perform IT service management operations. This integration bridge empowers your AI assistants to manage tickets, changes, problems, releases, assets, projects, and more.

## Key Features

- **36 tools** organized into 13 independently loadable scopes
- **Dual authentication**: per-user OAuth2 (via MCP gateway) with API key fallback
- **Multiple transports**: stdio (local), SSE, and streamable-http
- **Dynamic form discovery**: auto-discover custom fields for any entity type
- **Scope-based loading**: load only the tool modules you need
- **Docker & Helm**: ready for Kubernetes deployment

## Architecture

| Scope | Tools | Description |
| ----- | ----- | ----------- |
| `tickets` | `manage_ticket`, `manage_ticket_conversation`, `manage_service_catalog` | Ticket CRUD, conversations, service catalog |
| `changes` | `manage_change`, `manage_change_note`, `manage_change_task`, `manage_change_time_entry`, `manage_change_approval` | Change requests with full sub-resource support |
| `problems` | `manage_problem`, `manage_problem_note`, `manage_problem_task`, `manage_problem_time_entry` | Problem management |
| `releases` | `manage_release`, `manage_release_note`, `manage_release_task`, `manage_release_time_entry` | Release management |
| `assets` | `manage_asset`, `manage_asset_details`, `manage_asset_relationship` | Assets/CMDB |
| `status_page` | `manage_status_page`, `manage_maintenance_window` | Status pages, maintenance, incidents |
| `departments` | `manage_department`, `manage_location` | Departments & locations |
| `agents` | `get_me`, `manage_agent`, `manage_agent_group` | Identity, agents & groups |
| `requesters` | `manage_requester`, `manage_requester_group` | Requesters & groups |
| `solutions` | `manage_solution` | Knowledge base |
| `projects` | `manage_project`, `manage_project_task` | Project management (NewGen) |
| `products` | `manage_product` | Product catalog |
| `misc` | `manage_canned_response`, `manage_workspace` | Canned responses, workspaces |

Plus 2 **discovery tools** always loaded: `discover_form_fields` and `clear_field_cache`.

## Quick Start

### Local Development (API key + stdio)

```bash
FRESHSERVICE_APIKEY=<key> FRESHSERVICE_DOMAIN=yourcompany.freshservice.com freshservice-mcp
```

### Kubernetes / Gateway (OAuth2 + SSE)

```bash
helm install freshservice-mcp oci://ghcr.io/forterro/charts/freshservice-mcp \
  --version 0.2.0 \
  --set config.FRESHSERVICE_DOMAIN=yourcompany.freshservice.com
```

No API key needed — the MCP gateway forwards per-user OAuth2 tokens.

## Documentation

| Document | Description |
|----------|-------------|
| [Authentication](docs/authentication.md) | API key setup, OAuth2 via gateway, creating a Freshservice OAuth app, JWT token structure |
| [Deployment](docs/deployment.md) | Local, Docker, Helm chart, client configs (Claude Desktop, VS Code), transports, scopes, troubleshooting |
| [Tools Reference](docs/tools-reference.md) | Complete reference for all 36 tools with actions and parameters |
| [Examples](docs/examples.md) | Example prompts for common operations |

## License

MIT License. See the [LICENSE](LICENSE) file for details.

## Additional Resources

- [Freshservice API Documentation](https://api.freshservice.com/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [Claude Desktop Integration Guide](https://docs.anthropic.com/claude/docs/claude-desktop)

---

<p align="center">Built with ❤️ by effy</p>

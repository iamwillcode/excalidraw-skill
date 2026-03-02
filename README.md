# Excalidraw Diagrams

A Claude Code skill for generating architecture diagrams on a live Excalidraw canvas — from any codebase, in any language.

## What This Does

**Excalidraw Diagrams** lets you turn codebases into visual architecture diagrams without touching a design tool. Point Claude Code at your project, describe what you want to visualize, and watch it build the diagram in real time on a live canvas.

### Key Features

- **Zero Manual Drawing** — Describe what you want. Claude reads your code, identifies components, and draws the diagram.
- **Live Canvas** — Watch shapes and arrows appear in real time in your browser. Not a static image generator.
- **Auto-Routing Arrows** — Connect components by ID. Arrows find the shortest path to shape edges automatically.
- **Color-Coded by Role** — Databases are green, APIs are purple, frontends are blue. Consistent visual language across all diagrams.
- **Any Codebase** — Works with Python, TypeScript, Go, Rust, Java — whatever your project uses.
- **Multiple Exports** — PNG for docs, SVG for web, `.excalidraw` for editing, or a shareable URL.

## Examples

Each example shows the same prompt rendered two ways: **Markdown** (auto-generated from `create_from_mermaid`) vs **Excalidraw** (native canvas with `batch_create_elements`). Same input, different output quality.

### Microservices Architecture

> *"Draw a microservices architecture with: React frontend, API Gateway, Auth Service, User Service, Order Service, a RabbitMQ message queue, PostgreSQL database, and Redis cache. Use vertical flow layout."*

| Markdown | Excalidraw |
|:---:|:---:|
| ![Markdown](examples/microservices-markdown.png) | ![Excalidraw](examples/microservices-excalidraw.png) |

### CI/CD Pipeline

> *"Draw a CI/CD pipeline: Developer pushes to GitHub, triggers GitHub Actions, which runs lint, test, build stages, then deploys to staging, runs E2E tests, and promotes to production. Horizontal pipeline layout."*

| Markdown | Excalidraw |
|:---:|:---:|
| ![Markdown](examples/cicd-pipeline-markdown.png) | ![Excalidraw](examples/cicd-pipeline-excalidraw.png) |

### Event-Driven System

> *"Draw an event-driven architecture with a central Kafka event bus. Producers: Payment Service, Order Service, User Service. Consumers: Notification Service, Analytics Service, Audit Logger. Hub-and-spoke layout."*

| Markdown | Excalidraw |
|:---:|:---:|
| ![Markdown](examples/event-driven-markdown.png) | ![Excalidraw](examples/event-driven-excalidraw.png) |

### Data Pipeline

> *"Draw a data pipeline: data ingested from REST API and S3 bucket, processed by Apache Spark, stored in a data lake, transformed by dbt, loaded into Snowflake warehouse, served to a Grafana dashboard. Horizontal flow."*

| Markdown | Excalidraw |
|:---:|:---:|
| ![Markdown](examples/data-pipeline-markdown.png) | ![Excalidraw](examples/data-pipeline-excalidraw.png) |

## Installation

### For Claude Code Users (Plugin)

```bash
# Add the marketplace
/plugin marketplace add edwingao28/excalidraw-skill

# Install the plugin
/plugin install excalidraw@excalidraw-skill
```

### Manual Download

1. Download the `skills/excalidraw/` folder from this repo
2. Place it in `~/.claude/skills/excalidraw/`
3. Restart Claude Code

## Prerequisites

This skill requires an **Excalidraw MCP server**. The default is [yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw), but any MCP server that exposes compatible tools will work (see [Compatible MCP Servers](#compatible-mcp-servers) below).

**Important:** The server must be registered under the key `"excalidraw"` — the skill calls `mcp__excalidraw__*` tools, and this prefix is derived from the registration key name.

**If you installed via plugin marketplace**, the default MCP server is auto-registered — skip to the Canvas Server section.

**If you installed manually**, add this to your Claude Code MCP config:

```json
{
  "excalidraw": {
    "command": "npx",
    "args": ["-y", "mcp-excalidraw-server"],
    "env": {
      "EXPRESS_SERVER_URL": "http://localhost:3000"
    }
  }
}
```

### Canvas Server (Required)

The MCP server connects to a live Excalidraw canvas. You must start the canvas server before using the skill:

**Option A — Docker (recommended):**
```bash
docker run -d -p 3000:3000 ghcr.io/yctimlin/mcp_excalidraw-canvas:latest
```

**Option B — From source:**
```bash
git clone https://github.com/yctimlin/mcp_excalidraw.git
cd mcp_excalidraw && npm ci && npm run build
HOST=0.0.0.0 PORT=3000 npm run canvas
```

Then open `http://localhost:3000` in your browser. The canvas must stay open for screenshots and image export to work.

## Usage

### Visualize a Codebase

```
"Draw a diagram of this project's architecture"
```

The skill will:
1. Read your codebase to identify components and connections
2. Clear the canvas and read the design guide
3. Plan a layout with proper spacing and color coding
4. Create all shapes and arrows in a single batch call
5. Screenshot to verify, adjust if needed
6. Export to your preferred format

### Quick Diagram from Description

```
"Draw a microservices diagram with a React frontend, API gateway,
three backend services, a message queue, and a PostgreSQL database"
```

### Trace a Data Flow

```
"Trace how the auth token flows from login to API request to database query"
```

The skill supports parameter threading diagrams — tracing a value through architectural layers with split/converge paths and decision nodes.

### Convert from Mermaid

```
"Create an excalidraw diagram from this mermaid:
graph TD; A[Frontend] -->|REST| B[API]; B -->|SQL| C[Database]"
```

## How It Works

The skill uses **Excalidraw MCP tools** — no raw JSON manipulation needed:

| Tool | Purpose |
|------|---------|
| `batch_create_elements` | Create all shapes + arrows in one call |
| `get_canvas_screenshot` | Visual verification after each step |
| `read_diagram_guide` | Load color palette and sizing rules |
| `export_to_image` | Save as PNG or SVG |
| `export_scene` | Save as editable `.excalidraw` file |
| `export_to_excalidraw_url` | Generate a shareable link |
| `create_from_mermaid` | Quick diagram from Mermaid syntax |

## Color Palette

Every component type gets a consistent color:

| Component | Background | Stroke |
|-----------|------------|--------|
| Frontend/UI | `#a5d8ff` | `#1971c2` |
| Backend/API | `#d0bfff` | `#7048e8` |
| Database | `#b2f2bb` | `#2f9e44` |
| AI/ML | `#e599f7` | `#9c36b5` |
| Queue/Event | `#fff3bf` | `#fab005` |
| External API | `#ffc9c9` | `#e03131` |
| Storage | `#ffec99` | `#f08c00` |
| Zone/Group | `#e9ecef` | `#868e96` |

Cloud-specific palettes (AWS, Azure, GCP, Kubernetes) are included in `references/colors.md`.

## Layout Patterns

The skill knows four layout strategies and picks the right one for your diagram:

- **Vertical Flow** — Top-to-bottom layers. Best for web architectures, request flows.
- **Horizontal Pipeline** — Left-to-right stages. Best for CI/CD, data pipelines, ETL.
- **Hub and Spoke** — Central node with radiating connections. Best for event-driven systems.
- **Data Flow** — Multi-column with annotations. Best for parameter threading, call chain traces.

## Files

| File | Purpose |
|------|---------|
| `skills/excalidraw/SKILL.md` | Main skill — workflow, examples, layout patterns, sizing rules |
| `skills/excalidraw/references/colors.md` | Color palettes for default, AWS, Azure, GCP, Kubernetes |

## Compatible MCP Servers

Any Excalidraw MCP server that exposes the core tools will work. The key tools this skill requires:

| Tool | Usage |
|------|-------|
| `batch_create_elements` | Create all shapes + arrows (required) |
| `get_canvas_screenshot` | Visual verification (required) |
| `read_diagram_guide` | Load color/sizing rules |
| `export_to_image` | PNG/SVG export |
| `create_from_mermaid` | Mermaid conversion |

Known compatible servers:

| Package | Tools | Install |
|---------|-------|---------|
| [`mcp-excalidraw-server`](https://github.com/yctimlin/mcp_excalidraw) (default) | 26 — full coverage | `npx -y mcp-excalidraw-server` |
| [`excalidraw-mcp-server`](https://github.com/debu-sinha/excalidraw-mcp-server) | 16 — covers core set | `npx -y excalidraw-mcp-server` |

To use a different server, register it under the key `"excalidraw"` in your Claude Code MCP config.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- An Excalidraw MCP server registered as `"excalidraw"` (see [Compatible MCP Servers](#compatible-mcp-servers))
- A browser with the Excalidraw canvas open (for servers that require it)

## Credits

Created by [@edwingao28](https://github.com/edwingao28) with Claude Code.

## License

MIT — Use it, modify it, share it.

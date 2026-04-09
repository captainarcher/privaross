# Privar OSS Project - Private Intelligence. 
## Knows everything. Tells no one.

Privar is a personal intelligence system that runs entirely on your own hardware[1]. It knows your whole life -- health, finances, relationships, legal, home, spatial -- and acts on it through agents. All intelligence is local. All data is private. All storage is on-premise.

[1] Alpha release assumes a local Mac device with Apple Metal and Apple Silicon with sufficient resources to run local AI. There are plans to support running it in enterprise environments including cloud providers, Kubernetes, etc. A future release will enable segmentation of your AI infrastructure, application components, databases, and other infrastructure. 

## Roadmap
  - v0.1.0-alpha: Auto-ingest, intelligence foundation (Bayesian learning), multi-directory, agents, health life domain support, Apple Health import, Epic Health Portal import, supervisory agents, initial CLI, initial GUI, workflow engine, documents import and knowledge graph (rudimentary), memory, Argus security with GNN, service health, multi-node deployment (Apple Silicon and NVIDIA DGX Spark).
  - v0.2.0-alpha: April 9, 2026. Wiki synthesis, dual-mode editor, wiki-derived graph, L0-L3 memory, diagnostics, LLM playground, skills registry, documents auto-ingest (local). <---We are here
  - v0.3.0-alpha: Adaptive retrieval, LLM context injection, smart briefings, proactive notifications, wiki-aware intelligence, auto-ingest settings UI, conversation log persistence, documents Qdrant embedding dimension verification and alignment
  - v0.4.0-alpha: Initial public release - mid-April. documents auto-ingest (cloud sources), initial MCP support, image analysis, GNN-based adaptive security validation with adversarial agents, agent builder with natural language interface, multi-agent chaining improvements (co-pilot and YOLO modes)
  - v0.5.0-beta: late-April. ML-powered analysis of JPG, PNG, TIFF, and PDF files. OCR, object detection, scene understanding, and content extraction to feed into the knowledge graph and make visual content searchable and queryable.
  - v0.6.0-beta: mid-May. UI improvements. 
  - v0.7.0-gamma: late-May. Voice agent functional.
## Architecture

```
                          CLI / Web UI / Mobile
                                 |
                          +--------------+
                          |   Caddy TLS  |
                          +--------------+
                                 |
                          +--------------+
                          |   Gateway    |  (Go, rate limiting, auth, routing)
                          +--------------+
                           /            \
            +-------------+              +---------------+
            | Go Services |              | Python Services|
            +-------------+              +---------------+
            | Profiles    |              | Agents         |
            | Auth        |              | Memory         |
            | Scheduler   |              | LLM Bridge     |
            | Audit       |              | Ingestion      |
            | Argus       |              | Skills         |
            | Browser     |              | Embeddings     |
            | Sandbox     |              | Search         |
            +-------------+              +---------------+
                           \            /
                     +-------------------------+
                     |    Data & Infrastructure |
                     +-------------------------+
                     | PostgreSQL    |
                     |   + Apache AGE (graph)   |
                     |   + TimescaleDB (tsdb)   |
                     | Qdrant (vectors)         |
                     | Valkey (cache)           |
                     | OpenBao (secrets)        |
                     | NATS (event bus)         |
                     | RustFS (object storage)  |
                     +-------------------------+
                                 |
                     +-------------------------+
                     |    Host ML Services      |
                     +-------------------------+
                     | Ollama (LLM inference)   |
                     | Argus GNN (behavioral)   |
                     | Whisper (speech-to-text)  |
                     +-------------------------+
```

**Key design decisions:**

- **Podman** (rootless, daemonless) -- not Docker
- **OpenBao** manages all secrets and database credentials dynamically -- nothing hardcoded in env files
- **Argus** monitors agent behavior in real-time using a Graph Neural Network
- **[nono.sh]** provides kernel-level sandboxing (Landlock/Seatbelt) for agent containers
- **Ollama and ML services** run on the host for direct GPU access (Metal, CUDA). These can run on the same machine as other services or other machines. Both Apple Silicon and NVIDIA (tested on DGX Spark) are currently supported.

## Agent Security

Privar uses a multi-layer security model for agents:

1. **Cerbos ABAC policies** -- attribute-based permission boundaries per agent
2. **OpenBao transit encryption** -- all agent data encrypted at rest and in transit
3. **Argus GNN monitoring** -- real-time behavioral anomaly detection via graph neural network
4. **nono.sh kernel sandboxing** -- Landlock/Seatbelt isolation with atomic rollback on violations
5. **Rootless Podman containers** -- user namespace isolation, no root daemon (some services run directly on the host)
6. **Cryptographic audit trails** -- Sigstore-signed logs for forensic analysis

## CLI Examples

### Agents

```bash
# List all active agents with trust scores
privar agents list

# Start an interactive chat session
privar agent chat

# Execute a one-off agent command
privar agent execute "summarize my calendar for this week"

# Check an agent's trust score (from Argus)
privar agents trust <agent-id>
```

### Argus Security Monitoring

```bash
# Check overall security status (Argus + GNN service)
privar argus status

# View recent security events, optionally filtered by severity
privar argus events
privar argus events --severity critical --limit 5

# Run behavioral analysis on a specific agent
privar argus analyze <agent-id> --profile default

# View current detection thresholds
privar argus threshold

# Update behavioral baseline for a profile
privar argus baseline update <profile-id>

# Check ML/GNN service status and acceleration (Metal/CUDA)
privar ml status

# Test GNN inference with sample data
privar ml test
```

### Skills Registry

```bash
# Search for skill packages
privar registry search "finance"

# Install a skill package
privar registry install finance-tracker
```

### System

```bash
# Check system health
privar system status

# Manage profiles
privar profile list
privar profile create
privar profile switch <id>
```

## Requirements

- macOS
- Apple Silicon (tested on M4 Max with 36gb RAM and a M3 Ultra with 96gb RAM)
- Podman 4.4+
- ML components can also run on NVIDIA hardware (tested on a DGX Spark with 128gb RAM)

## License

Apache 2.0

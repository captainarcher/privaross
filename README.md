# Privar OSS Project - Private Intelligence. 
## Knows everything. Tells no one.

Privar is a personal intelligence system that runs entirely on your own hardware[1]. It knows your whole life -- health, finances, relationships, legal, home, spatial -- and acts on it through agents. All intelligence is local. All data is private. All storage is on-premise.

[1] Alpha release assumes a local Mac device with Apple Metal and Apple Silicon with sufficient resources to run local AI. There are plans to support running it in enterprise environments including cloud providers, Kubernetes, etc. A future release will enable segmentation of your AI infrastructure, application components, databases, and other infrastructure. 

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
                     | PostgreSQL + pgvector    |
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
- **nono.sh** provides kernel-level sandboxing (Landlock/Seatbelt) for agent containers
- **Ollama and ML services** run on the host for direct GPU access (Metal, CUDA)

## Agent Security

Privar uses a multi-layer security model for agents:

1. **Cerbos ABAC policies** -- attribute-based permission boundaries per agent
2. **OpenBao transit encryption** -- all agent data encrypted at rest and in transit
3. **Argus GNN monitoring** -- real-time behavioral anomaly detection via graph neural network
4. **nono.sh kernel sandboxing** -- Landlock/Seatbelt isolation with atomic rollback on violations
5. **Rootless Podman containers** -- user namespace isolation, no root daemon
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
- Apple Silicon (tested on M4 Max with 36gb RAM)
- Podman 4.4+

## License

Apache 2.0

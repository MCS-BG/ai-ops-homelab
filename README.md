# ai-ops-homelab

Sovereign Super Intelligence is a home lab for running and operating local AI on Kubernetes, built one day at a time. The point of the system is that a business keeps control of its own model, data, tools, and the loop that is allowed to call them.

It uses k3s, a local model on a small GPU, RAG that only supplies evidence, a chat UI that is not the agent loop, an MCP server for tools, observability, and prompt-injection guardrails that fail closed. The same patterns are practised here before they are used on AKS and EKS.

## The three machines

| Name in the docs | Role |
| --- | --- |
| terminal | The laptop everything is driven from: `kubectl`, `helm`, `git`, `gh`, browser. Not part of the cluster. |
| lab host | k3s server with the NVIDIA GPU. Runs Ollama, pgvector, Open WebUI, the MCP server, Prompt Guard. Kubernetes node name `d`. |
| monitoring node | CPU-only k3s agent (joined on Day 6b). Runs Prometheus, Grafana, Loki, Tempo, the OpenTelemetry Collector and Langfuse. Node name `surface`. |

The terminal reaches the Kubernetes API through an SSH tunnel to the lab host and uses
`KUBECONFIG=~/.kube/k3s-xps.yaml`. From Day 8b the tunnel can run over Tailscale.

## Day by day

| Day | Write-up | Step file |
| --- | --- | --- |
| 1 | Host hardening | [docs/day-01-host-hardening.md](docs/day-01-host-hardening.md) |
| 2 | Secure single-node k3s with the GPU | [docs/day-02-secure-k3s.md](docs/day-02-secure-k3s.md) |
| 3 | Ollama on the GPU | [docs/day-03-model-serving.md](docs/day-03-model-serving.md) |
| 4 | RAG with pgvector | [docs/day-04-rag.md](docs/day-04-rag.md) |
| 5 | Open WebUI (5b: more models) | [docs/day-05-open-webui.md](docs/day-05-open-webui.md), [models/add-models-steps.md](models/add-models-steps.md) |
| 6 | MCP server: notes search and a mock FO OData service | [docs/day-06-mcp-server.md](docs/day-06-mcp-server.md), [mcp/day-06-steps.md](mcp/day-06-steps.md) |
| 6b | Second node for monitoring | [docs/day-06b-second-node.md](docs/day-06b-second-node.md), [docs/day-06b-surface-join-steps.md](docs/day-06b-surface-join-steps.md) |
| 7 | Observability, Langfuse and Prompt Guard | [docs/day-07-observability.md](docs/day-07-observability.md), [docs/day-07-observability-steps.md](docs/day-07-observability-steps.md) |
| 8a | Git repo, GitHub Actions image builds, GHCR images | [docs/day-08a-steps.md](docs/day-08a-steps.md) |
| 8b | Prompt Guard in the MCP server and Open WebUI, Langfuse generations, Tailscale | [docs/day-08b-steps.md](docs/day-08b-steps.md) |

## Layout

| Path | Contents |
| --- | --- |
| `apps/<name>/` | Source and Dockerfile for each image built by GitHub Actions: `rag-worker`, `mcp-server`, `prompt-guard`. |
| `.github/workflows/` | One workflow per image plus the shared `_build-image.yml`. Images go to `ghcr.io/<owner>/<name>:<short-sha>`. |
| `k8s/` | Every manifest and Helm values file, named `day-NN-*.yaml` after the day that introduced it. |
| `openwebui/` | The Open WebUI Prompt Guard filter function (Day 8b). |
| `scripts/pin-images.sh` | Pins the image tags in `k8s/day-08*.yaml` to the commits that built them. |
| `staged/day-08b/` | Day 8b code changes. Day 8b step 2 moves them into `apps/` so they build in their own commit, and the folder is gone after that. |
| `rag/`, `mcp/`, `prompt-guard/` | The Day 4, 6 and 7 originals, kept as history. `apps/` is the current source. |
| `models/`, `research/`, `diagrams/`, `screenshots/` | Day 5b model notes, background research, architecture diagrams, redacted screenshots. |

## How to use it

Follow the step files in order. Each step says which machine it runs on (terminal, lab host or
monitoring node), gives the command, and shows the expected output. Secrets are never stored in the
repo. They are created with `openssl rand` or read with `read -rs` straight into Kubernetes Secrets.

## License

MIT. See [LICENSE](LICENSE).

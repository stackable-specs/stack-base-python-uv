# ADR-001: Adopt Docker and Docker Compose for Containerization

## Status

Accepted

## Context and Problem Statement

This project is a Python base template intended to be AI-agent friendly with strong quality and correctness controls. To ensure reproducibility, portability, and consistency across development, CI, and production environments, we need a containerization strategy that:

1. Produces minimal, secure container images
2. Enables local development with hot-reload capabilities
3. Supports single-host deployments (Compose) with a path to orchestration (Kubernetes)
4. Enforces quality gates on images (vulnerability scanning, SBOM, signing)
5. Follows the stackable-specs methodology for spec-driven development

The project must also support the docker.md and docker-compose.md specs from stackable-specs, which define strict rules for Dockerfile construction, Compose file structure, and image publication.

## Decision Drivers

- **Reproducibility:** Builds must produce identical images from the same source code
- **Security:** Images must run as non-root, have minimal attack surface, and be scannable for vulnerabilities
- **Spec compliance:** Must follow all rules in docs/specs/delivery/docker.md and docker-compose.md
- **Developer experience:** Must support fast iteration with local development overrides
- **AI-agent friendliness:** Configuration must be explicit, documented, and verifiable
- **Supply chain security:** Images must be pinned by digest, signed, and accompanied by SBOMs

## Considered Options

### Option A: Docker + Docker Compose (with spec compliance)

Use Docker with multi-stage Dockerfiles and Docker Compose for orchestration, following all stackable-specs rules:
- Multi-stage builds with minimal runtime images
- Base images pinned by SHA256 digest
- Non-root user execution
- Read-only root filesystem with tmpfs
- Bounded logging and resource limits
- Development overrides via compose.override.yaml

### Option B: Podman + Podman Compose

Use Podman as a daemonless alternative with Podman Compose:
- Rootless by default
- Compatible with Docker CLI
- Podman Compose as drop-in replacement

### Option C: Kaniko for CI-only builds

Use Kaniko for building images in CI without Docker daemon:
- No privileged container required
- Better for CI/CD security
- No local development story

### Option D: No containerization (bare metal)

Skip containerization and rely on virtualenv + uv for dependency management:
- Simpler initial setup
- No container orchestration knowledge required
- No reproducibility guarantees across environments

## Decision Outcome

Chosen option: **Option A: Docker + Docker Compose (with spec compliance)**

We will use Docker with multi-stage Dockerfiles and Docker Compose, following all rules from `docs/specs/delivery/docker.md` and `docs/specs/delivery/docker-compose.md`.

### Rationale

1. **Spec alignment:** The docker.md and docker-compose.md specs provide comprehensive rules that directly address security, reproducibility, and operational concerns
2. **Ecosystem maturity:** Docker and Docker Compose have the largest ecosystem, best tooling support, and most documentation
3. **AI-agent friendliness:** Docker and Compose are well-documented and widely understood by AI coding agents
4. **Path to production:** Compose provides a single-host deployment path with Kubernetes as an upgrade target
5. **Supply chain security:** Docker ecosystem has mature tools for image signing (cosign), vulnerability scanning (Trivy), and SBOM generation

## Decision Details

### Dockerfile Implementation

- Multi-stage build with `builder` and `runtime` stages
- Base image: `python:3.11-slim` pinned by SHA256 digest with tag comment
- UV installer: `ghcr.io/astral-sh/uv:0.5.11` pinned by digest
- Non-root user: `10001:10001`
- Healthcheck placeholder for long-running services
- BuildKit cache mounts for faster builds
- No secrets in image layers (BuildKit `--mount=type=secret` for build secrets)

### Docker Compose Implementation

- Named `compose.yaml` (not `docker-compose.yml`)
- No top-level `version:` key (Compose spec unversioned)
- Explicit project `name: stack-base-python-uv`
- Resource limits via `deploy.resources.limits`
- Read-only root filesystem with `tmpfs` for `/tmp`
- Bounded logging (json-file with max-size/max-file)
- Non-root `user:` directive
- Development overrides in `compose.override.yaml`
- Environment interpolation via `${VAR}` from `.env`
- `.env.example` documents all variables

### SHA Pinning Strategy

We will use **gh-pin** (GitHub CLI extension) for automated SHA pinning:

```bash
# Install gh-pin
gh extension install grantbirki/gh-pin

# Pin all Docker images in Dockerfile
gh pin Dockerfile

# Pin images in compose.yaml
gh pin compose.yaml

# Pin GitHub Actions in CI workflows
gh pin .github/workflows/ci.yml
```

**Rationale for gh-pin:**
- Pins both Docker images AND GitHub Actions to SHA digests
- Maintains compatibility with Dependabot for version updates
- Supports `--dry-run` for preview
- Supports `--platform` for architecture-specific digests
- SLSA Level 3 provenance for supply chain security
- Active maintenance and good documentation

### CI Integration (Future)

The following will be added in subsequent PRs:

1. **Vulnerability scanning:** Trivy scan in CI (docker.md:18)
2. **Image signing:** cosign for signed images (docker.md:16)
3. **SBOM generation:** Syft or Trivy for SBOMs (docker.md:17)
4. **Digest pinning CI:** Automated verification that images are pinned
5. **Compose validation:** `docker compose config --quiet` in CI (docker-compose.md:25)

## Consequences

### Positive

- **Reproducibility:** Images built from the same source produce identical outputs
- **Security:** Non-root execution, read-only filesystem, minimal attack surface
- **Spec compliance:** All rules from docker.md and docker-compose.md are followed
- **Developer experience:** `compose.override.yaml` enables hot-reload during development
- **Supply chain security:** SHA pinning prevents mutable tag attacks
- **AI-agent friendliness:** Docker and Compose are well-documented and widely understood

### Negative

- **Complexity:** Multi-stage Dockerfiles and Compose files require more initial setup
- **Learning curve:** Contributors must understand Docker and Compose concepts
- **Build time:** Multi-stage builds take longer than single-stage (mitigated by BuildKit cache)
- **Maintenance:** SHA digests must be updated when base images change (automated by Renovate + gh-pin)

### Neutral

- **Tool dependencies:** Requires Docker and Docker Compose for local development
- **CI dependencies:** Requires BuildKit, cosign, Trivy in CI pipeline
- **Registry requirements:** Images must be published to a container registry for production use

## References

- `docs/specs/delivery/docker.md` — Docker image spec
- `docs/specs/delivery/docker-compose.md` — Compose file spec
- [gh-pin GitHub repository](https://github.com/GrantBirki/gh-pin) — SHA pinning tool
- [crane documentation](https://github.com/google/go-containerregistry/blob/main/cmd/crane/README.md) — Manual digest lookup
- [MADR template](./000-template.md) — ADR format
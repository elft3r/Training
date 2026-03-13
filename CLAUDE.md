# CLAUDE.md

## Project Overview

This is the **56K.Cloud Training Repository** — an open-source, community-driven collection of Docker, Kubernetes, Cloud, and DevOps tutorials and hands-on labs. The content is hosted as a Jekyll static site at [training.56kcloud.io](https://training.56kcloud.io).

**License:** Apache 2.0

## Repository Structure

```
├── Docker/                   # Docker tutorials (primary content area)
│   ├── kickstart/            # Main Docker course with progressive chapters
│   │   ├── chapters/         # Course chapters (setup → containers → compose → swarm)
│   │   ├── flask-app/        # Python Flask example application
│   │   └── static-site/      # Static website example
│   ├── security/             # Security deep-dives (AppArmor, seccomp, capabilities, etc.)
│   ├── networking/           # Docker networking tutorials
│   ├── 12factor/             # 12-factor app methodology
│   ├── swarm-mode/           # Docker Swarm tutorials
│   ├── Docker-Orchestration/ # Orchestration tutorials
│   ├── registry/             # Docker registry tutorials
│   └── additional-ressources/# Supplementary Docker resources
├── Kubernetes/               # Kubernetes tutorials (kickstart, Helm, Minikube)
├── DockerCon/                # Logging & Monitoring workshops
├── DevOpsDays/               # Docker training materials
├── istio-workshop/           # Istio service mesh workshop (WIP)
├── Presentations/            # PDF presentations
├── img/                      # Repository images
├── _config.yml               # Jekyll site configuration
├── CNAME                     # DNS: training.56kcloud.io
├── contribute.md             # Contribution guidelines
├── REVIEW.md                 # Docker kickstart review/feedback
└── LICENSE                   # Apache 2.0
```

## Content Type

This is a **documentation and tutorial repository**, not a software application. There is:

- **No build system** (no package.json, Makefile, etc.)
- **No automated test suite**
- **No CI/CD pipeline**
- **No linter/formatter configuration**

Quality is managed through manual review and community feedback. `REVIEW.md` contains detailed content quality feedback.

## Key Technologies Covered

- **Markdown** — all tutorial content (168+ files)
- **Dockerfile** — example Dockerfiles throughout (Python, Node.js, .NET)
- **docker-compose.yml** — multi-container examples
- **YAML** — Kubernetes manifests, Istio configs
- **Python** — Flask example app in `Docker/kickstart/flask-app/`
- **Shell** — cleanup scripts, command examples

## Working with Content

### Adding or Editing Tutorials

- All tutorials are written in Markdown
- Follow the existing chapter structure in `Docker/kickstart/chapters/` as a reference
- Use fenced code blocks with language hints for all code examples
- Prefix shell commands with `$` prompt to distinguish commands from output
- Use modern Docker CLI syntax (`docker container run`, `docker image ls`) over legacy (`docker run`, `docker ps`)
- Keep base images up to date (avoid EOL versions like `ubuntu:12.04`, `alpine:3.5`)

### Jekyll Site

The site uses the **Cayman** Jekyll theme. Configuration in `_config.yml` excludes `Kubernetes/website/` and `istio-workshop/` from the build.

## Contribution Guidelines

- All contributions are licensed under Apache 2.0
- Follow the lightweight Docker contribution policies referenced in `contribute.md`
- Submit issues and pull requests for new tutorials or fixes to existing ones
- Use native Docker and Kubernetes tooling whenever possible

## Known Content Issues

See `REVIEW.md` for a detailed list including:
- Some broken navigation links between chapters
- Outdated base images in Dockerfile examples
- Inconsistent Docker command syntax (legacy vs modern CLI)
- Missing multi-stage build coverage in the kickstart course

## Git Conventions

- Branch naming: `<author>/<description>`
- Use descriptive commit messages summarizing the change
- Primary branches: `main`, `master`

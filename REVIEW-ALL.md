---
nav_exclude: true
---

# Training Repository — Whole-Repo Content Review (Triage)

**Date:** 2026-04-18
**Scope:** every training area in this repository
**Depth:** triage — rubric score, decision, top issues, rough effort. **Not** an exhaustive per-file audit.
**Companion document:** `Docker/kickstart/REVIEW.md` (kickstart-specific open issues, still valid and cross-referenced below).

The goal of this document is to give maintainers a single place to see what's worth keeping, what needs work, what should be consolidated, and what should be archived or retired.

---

## Rubric

Six dimensions, scored 0–3 each (max 18). Band → decision:

- **15–18 → KEEP** — publish as-is, cosmetic fixes only
- **11–14 → UPDATE** — stays live; specific tasks enumerated
- **7–10 → CONSOLIDATE** — merge into a named target area
- **4–6 → ARCHIVE** — exclude from site, keep in git
- **0–3 → RETIRE** — propose removal

Override: any **live-nav** area scoring 0 on Technical Accuracy is forced to UPDATE minimum.

| Dimension | What counts |
|---|---|
| **TA — Technical Accuracy** | Commands actually run; base images not EOL; compose/CLI syntax current |
| **Fr — Freshness** | No retired products (Docker for AWS/Azure, Docker Cloud/Store, Kitematic, Docker Machine) |
| **Co — Completeness** | Narrative + exercises + cleanup, not just external link dumps |
| **Di — Discoverability** | Listed in `_data/navigation.yml` and/or area README |
| **Un — Uniqueness** | Not redundant with another area in this repo |
| **Pe — Pedagogical Quality** | Learning objectives, "why" explanations, troubleshooting |

> **Note on commit dates:** most areas show a last-commit date of 2026-04-17, but these are Jekyll-layout/cosmetic fixes (H1 promotion, font-size tweaks). The *content* itself is older than the git timestamp suggests. Rubric scores reflect content, not layout.

---

## Summary Table

| # | Area | In nav? | TA | Fr | Co | Di | Un | Pe | **Score** | **Decision** | Effort |
|---|------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | `Docker/kickstart` | yes | 2 | 2 | 3 | 3 | 2 | 3 | **15** | KEEP (targeted UPDATE) | M |
| 2 | `Docker/networking` | yes | 2 | 2 | 3 | 3 | 1 | 2 | **13** | UPDATE | M |
| 3 | `Docker/security` | yes | 1 | 2 | 3 | 3 | 3 | 2 | **14** | UPDATE | M |
| 4 | `Docker/12factor` | yes | 1 | 1 | 2 | 3 | 2 | 2 | **11** | UPDATE | M |
| 5 | `Docker/swarm-mode` | yes | 1 | 0 | 2 | 2 | 1 | 2 | **8** | CONSOLIDATE | M |
| 6 | `Docker/registry` | yes | 1 | 0 | 3 | 3 | 3 | 2 | **12** | UPDATE | S |
| 7 | `Docker/Docker-Orchestration` | no | 1 | 0 | 1 | 0 | 1 | 1 | **4** | ARCHIVE | S |
| 8 | `Docker/additional-ressources` | yes | 0 | 0 | 2 | 1 | 1 | 1 | **5** | ARCHIVE | S |
| 9 | `Kubernetes/` | yes | 2 | 1 | 1 | 2 | 2 | 1 | **9** | CONSOLIDATE | M |
| 10 | `DockerCon/` | yes | 2 | 1 | 3 | 3 | 2 | 2 | **13** | UPDATE | S |
| 11 | `DevOpsDays/` | yes | 2 | 2 | 0 | 2 | 0 | 0 | **6** | ARCHIVE | S |
| 12 | `istio-workshop/` | no (excluded) | 0 | 0 | 0 | 0 | 2 | 0 | **2** | RETIRE | S |
| 13 | `Presentations/` | no (excluded) | n/a | 0 | 1 | 0 | 2 | 1 | — | ARCHIVE (keep) | — |

---

## Per-Area Evaluations

### 1. `Docker/kickstart` — flagship course
**Scope:** Progressive 16-chapter Docker course (setup → alpine → images/volumes → multistage → webapps → devops → compose → swarm → networking → secrets → mongo/node-red → nextsteps).
**Files:** 18 md · in nav as `docker-kickstart`.
**Rubric (TA/Fr/Co/Di/Un/Pe):** 2/2/3/3/2/3 = **15**
**Top issues:**
- [TA] `chapters/votingapp-swarm.md:94,231,244` pins `postgres:9.4` (EOL 2020-02)
- [TA] `chapters/mongo.md`, `chapters/mongo-2.md` use `docker-compose` v1 CLI (per existing REVIEW.md §1)
- [Fr] `votingapp-swarm.md` references Docker for AWS/Azure and "Docker Store" (per REVIEW.md §2)
- [Un] Networking-basics + bridge-network chapters overlap with `Docker/networking/` A1/A2
- [Pe] No `.dockerignore`, `USER`, or resource-limits coverage (REVIEW.md §4)
**Decision:** KEEP with targeted UPDATE — already the strongest training; execute the open REVIEW.md items.
**Effort:** M
**Cross-ref:** `Docker/kickstart/REVIEW.md` §1–4 (already enumerated). **Don't rediscover — just execute.**

---

### 2. `Docker/networking`
**Scope:** 22-file deep-dive: CNM, bridge, overlay, macvlan, routing mesh, troubleshooting.
**Files:** 22 md · in nav as `docker-networking`.
**Rubric:** 2/2/3/3/1/2 = **13**
**Top issues:**
- [Un] Heavy overlap with `Docker/kickstart/chapters/networking-basics.md` and `bridge-network.md`, and with `Docker/security/networking/`
- [Fr] References to archived `docker/labs` and legacy Docker engine versions scattered in concepts/
- [Pe] No unified index — reader lands in A1 through A4 without clear prerequisites vs. the `concepts/` subtree
**Decision:** UPDATE — refresh external links; decide single authority on bridge/overlay (here) and de-link duplicated content from kickstart.
**Effort:** M

---

### 3. `Docker/security`
**Scope:** 13-topic security deep-dive: AppArmor, seccomp, capabilities, cgroups, userns, scanning, secrets, trust, swarm.
**Files:** 17 md · in nav as `docker-security`.
**Rubric:** 1/2/3/3/3/2 = **14**
**Top issues:**
- [TA] `Docker/security/apparmor/wordpress/Dockerfile:1` — `FROM php:5.6-apache` (EOL since 2019)
- [TA] `Docker/security/secrets-ddc/README.md:89,97` — `mysql:5.7` (EOL 2023-10)
- [Fr] `Docker/security/trust-basics/README.md:71–161` — walkthrough assumes **Docker Cloud** (retired 2018) and "Docker Store"; needs rewrite against Docker Hub
- [Fr] `Docker/security/trust/README.md:36` — "Docker Store" reference
**Decision:** UPDATE — the topics are uniquely valuable (security is a gap in most other trainings), but trust-basics and secrets-ddc are materially broken.
**Effort:** M

---

### 4. `Docker/12factor`
**Scope:** 12 chapters mapping 12-factor-app principles onto Docker.
**Files:** 14 md · in nav as `docker-12factor`.
**Rubric:** 1/1/2/3/2/2 = **11**
**Top issues:**
- [TA] `05_build_release_run.md:59`, `06_processes.md:41`, `07_port_binding.md:20` — `image: mongo:3.2` (EOL 2018-09)
- [Fr] `09_disposability.md:21`, `10_dev_prod_parity.md:10` — link to **Docker Store** (retired)
- [Fr] `05_build_release_run.md:80` — "Docker Cloud" orchestrator reference (retired)
- [Pe] Principles are timeless but examples are frozen in ~2017 tooling
**Decision:** UPDATE — bump mongo image, strip Docker Store/Cloud, refresh examples to Compose v2.
**Effort:** M

---

### 5. `Docker/swarm-mode`
**Scope:** Beginner swarm tutorial using Docker Machine to simulate multiple hosts.
**Files:** 2 md + shell/PowerShell scripts · in nav as `docker-swarm`.
**Rubric:** 1/0/2/2/1/2 = **8**
**Top issues:**
- [Fr] Entire tutorial built on **Docker Machine**, which was deprecated in 2021 and archived; the README openly says "preserved for legacy reasons"
- [Un] `Docker/kickstart/chapters/votingapp-swarm.md` already covers swarm end-to-end with a real app
- [TA] Setup scripts (`swarm-node-vbox-setup.sh`, Hyper-V .ps1) rely on deprecated tooling
- [Fr] Points users to external `training.play-with-docker.com/swarm-mode-intro/` as the preferred path
**Decision:** CONSOLIDATE into `Docker/kickstart` (votingapp-swarm is the better flagship). Keep a short "further reading" stub linking to Play-with-Docker and remove the Docker-Machine-based walkthrough.
**Effort:** M

---

### 6. `Docker/registry`
**Scope:** 3-part hands-on for running a private Docker registry.
**Files:** 4 md · in nav as `docker-registry`.
**Rubric:** 1/0/3/3/3/2 = **12**
**Top issues:**
- [Fr] `README.md`, `part-1.md`, `part-3.md` repeatedly say "Docker Store" — the concept is still valid but the product name is retired (now "Docker Hub")
- [Fr] Links to `store.docker.com` return 404 / redirect
- [TA] Registry image/version references should be checked against current `registry:2` guidance
**Decision:** UPDATE — pure find-and-replace for "Docker Store → Docker Hub", plus link fixes. Genuinely unique content; keep it.
**Effort:** S

---

### 7. `Docker/Docker-Orchestration`
**Scope:** Single README — a pointer to Jérôme Petazzoni's external workshop (slides + github.com/docker/orchestration-workshop) with a workshop outline.
**Files:** 1 md · **not** in nav.
**Rubric:** 1/0/1/0/1/1 = **4**
**Top issues:**
- [Co] No actual content — it's a link collection
- [Fr] External `github.com/docker/orchestration-workshop` repo is archived
- [Un] Entirely duplicates the subject of `Docker/kickstart` + `Docker/swarm-mode`
- [Di] Not referenced from any README or the nav
**Decision:** ARCHIVE — add to `_config.yml` `exclude:`. Keep git history. Optionally add a one-line pointer in the main README under "historical materials".
**Effort:** S

---

### 8. `Docker/additional-ressources`
**Scope:** Catch-all: dev-tools (Java/Node debug), Windows container modernization labs, DockerCon US 2017 labs.
**Files:** 67 md · in nav as "Additional Resources".
**Rubric:** 0/0/2/1/1/1 = **5**
**Top issues:**
- [TA] Windows Dockerfiles pinned to `microsoft/aspnet:windowsservercore-10.0.14393.*` and `microsoft/iis:windowsservercore-10.0.14393.693` — 2016 era, no longer on MCR with this tag scheme (7+ files affected)
- [TA] `developer-tools/java-debugging/registration-webserver/Dockerfile:2` — `FROM tomcat:7-jre8` (EOL 2022-08, archive-only)
- [TA] `developer-tools/ruby/README.md:85` — `FROM ubuntu:12.10` (EOL 2014-05)
- [TA] `developer-tools/nodejs/porting/*.md` — `mongo:3.2` throughout; also uses `docker-machine` commands
- [TA] `dockercon-us-2017/securing-apps-docker-enterprise/README.md:109,126,137,311` — `alpine:3.3` → advice to "upgrade" to `alpine:3.5` (both EOL now)
- [Fr] `dockercon-us-2017/` is explicitly event-frozen material
**Decision:** ARCHIVE — add path to `_config.yml` `exclude:` and drop from `main` nav. Preserves git history; the amount of rot across 67 files makes it un-economical to update. Keep as archival reference only.
**Effort:** S (config change) — rewriting the content would be L.

---

### 9. `Kubernetes/`
**Scope:** kickstart (README pointing to kubernetes.io), helm quickstart, minikube quickstart, additional-ressources (kops, links), coscale plugin, download-tools list.
**Files:** 8 md · in nav as `kubernetes`.
**Rubric:** 2/1/1/2/2/1 = **9**
**Top issues:**
- [Co] `Kubernetes/kickstart/README.md` is effectively a redirect to upstream docs — no original hands-on content
- [Fr] `third-party-coscale/` references a product (CoScale) acquired by Cisco in 2018 and discontinued
- [Fr] `additional-ressources/kops-howto.md` is kops-specific and frozen in time
- [Co] No K8s equivalent of `Docker/kickstart` — the flagship gap in the repo
**Decision:** CONSOLIDATE — keep minikube + helm quickstarts; retire CoScale page; either invest in a real K8s kickstart or collapse the section to "Pointers to upstream Kubernetes docs" to set honest expectations.
**Effort:** M (minor) or L (write a real kickstart).

---

### 10. `DockerCon/`
**Scope:** Logging (3 chapters: setup / getting-started / log-drivers) + Monitoring (3 chapters: stats / cAdvisor / Prometheus stack) + resources/links.
**Files:** 8 md · in nav as `dockercon`.
**Rubric:** 2/1/3/3/2/2 = **13**
**Top issues:**
- [Fr] Workshop origin is DockerCon 2017; some commands/images reflect that era
- [TA] `logging/setup.md`, `logging/getting-started.md`, `monitoring/stats.md` use `docker-compose` v1 CLI
- [Un] Topic-overlap with `Docker/additional-ressources/dockercon-us-2017/` and with the root `README.md`'s separate "Monitoring & Logging" link
- [Pe] Workshop content itself (ELK + Prometheus/cAdvisor) still pedagogically useful
**Decision:** UPDATE — the content is genuinely useful and distinct. Migrate to `docker compose` v2 CLI; bump any image tags; de-duplicate with dockercon-us-2017 folder.
**Effort:** S

---

### 11. `DevOpsDays/`
**Scope:** Single README with 4 links to `training.play-with-docker.com`.
**Files:** 1 md · in nav as `DevOpsDays`.
**Rubric:** 2/2/0/2/0/0 = **6**
**Top issues:**
- [Co] No local content at all
- [Un] Every link duplicates material already in `Docker/kickstart` or linked elsewhere
- [Pe] Zero explanation or sequencing — pure external link list
**Decision:** ARCHIVE — add to `_config.yml` `exclude:` and remove from `main` nav. The 4 links can be absorbed into `Docker/additional-ressources`' "Useful Links" if worth preserving.
**Effort:** S

---

### 12. `istio-workshop/`
**Scope:** Numbered tutorial folders (0_cluster … 8_nodes) + Helm charts for Istio 1.0.2 and 1.0.4.
**Files:** 4 md (one is a "Work in progress" README) + scripts/YAML · **already excluded** from Jekyll.
**Rubric:** 0/0/0/0/2/0 = **2**
**Top issues:**
- [TA] Istio 1.0.x is 7+ years behind current (1.20+); Helm chart layout diverges entirely from modern Istio (istioctl / operator)
- [Fr] Last substantive commit 2022-06-16; README says "Work in progress"
- [Co] No narrative markdown — only scripts and YAML; a learner can't follow it
- [Di] Already excluded from site, not linked from any README
**Decision:** RETIRE — `git rm -r istio-workshop/`. The content is unrecoverable short of a rewrite; the public site already treats it as non-existent.
**Effort:** S

---

### 13. `Presentations/`
**Scope:** 2 PDFs (`56K.Cloud Training.pdf` ~33MB, `Monitor_Everything.pdf` ~43MB). Already excluded from Jekyll.
**Files:** 0 md.
**Rubric:** n/a — this is a binary asset folder, not a tutorial.
**Decision:** ARCHIVE (status quo) — keep on disk; leave in `_config.yml` `exclude:`. Consider moving under `assets/archive/` to signal intent, or out of the repo entirely if LFS is ever adopted.
**Effort:** none (already correct).

---

## Cross-Cutting Recommendations

These apply across multiple areas and are cheaper to do once than per-area.

1. **"Docker Store" / "Docker Cloud" → "Docker Hub" global pass.** At least 10 files affected across `Docker/registry`, `Docker/12factor`, `Docker/security/trust*`, `Docker/security/apparmor`, `Docker/additional-ressources/windows/`. Single find-and-replace with eyes-on review. Effort: **S**.
2. **`docker-compose` v1 → `docker compose` v2 migration.** 35 files identified by scan. Most are trivial; the exceptions are `Docker/kickstart/chapters/mongo*.md` (per REVIEW.md §1) and `DockerCon/logging/*`, `DockerCon/monitoring/*`. Effort: **M**.
3. **Compose `version: "2"` / `version: "1"` → modern compose files.** 5 files found (all in `Docker/additional-ressources/windows/` and `developer-tools/java-debugging/`); this falls out when `additional-ressources` is archived. Effort: **S**.
4. **Retire `Docker Machine`-dependent content.** Tool is archived upstream. Affects `Docker/swarm-mode/` (CONSOLIDATE) and pieces of `Docker/additional-ressources/` (ARCHIVE). Effort: covered by the per-area decisions above.
5. **Navigation hygiene.** `Docker/kickstart` has `mongo.md`, `mongo-2.md`, `nodered.md` in `_data/navigation.yml` but not in the kickstart `README.md` index (REVIEW.md §4). Either surface them or drop them from the sidebar. Effort: **S**.
6. **Consider a frontmatter/last-updated convention.** None of the areas use `last_modified_at` frontmatter. Adding it (and showing it in the Jekyll layout) would make staleness self-evident to readers. Effort: **M**.
7. **Kubernetes section strategy call.** Either invest in a real Kubernetes kickstart (there isn't one — surprising for 2026) or collapse `Kubernetes/` into "pointers to upstream docs" so the nav doesn't over-promise. This is a product decision, not a cleanup. Effort: **L** if building, **S** if collapsing.

---

## Suggested Execution Order

1. **Quick wins (S effort, ~1 PR):** global "Docker Store → Docker Hub" replacement; archive `Docker/Docker-Orchestration`, `DevOpsDays`, `Docker/additional-ressources` via `_config.yml`; retire `istio-workshop/`.
2. **Small-area refreshes (S–M):** `Docker/registry`, `DockerCon/`, `Docker/12factor` updates.
3. **Larger refreshes (M):** execute `Docker/kickstart/REVIEW.md` backlog; fix `Docker/security` EOL images and `trust-basics` Docker-Cloud walkthrough.
4. **Consolidations (M):** fold `Docker/swarm-mode` into kickstart; de-duplicate kickstart ↔ `Docker/networking`.
5. **Strategic (L):** decide Kubernetes section direction.

---

## Inputs Used

- `/home/user/training/_config.yml` — Jekyll `exclude:` list
- `/home/user/training/_data/navigation.yml` — authoritative live nav
- `/home/user/training/Docker/kickstart/REVIEW.md` — kickstart open issues (cross-referenced, not re-enumerated)
- `/home/user/training/CONTRIBUTE.md`, `/home/user/training/README.md`, `/home/user/training/CLAUDE.md` — guidance and scope
- Repo-wide scans for: EOL base images, `docker-compose ` v1 CLI, `version: "1"/"2"` compose syntax, retired products (Docker for AWS/Azure, Docker Cloud, Docker Store, Kitematic, Docker Machine)
- `git log` timestamps per area (with the caveat noted above about cosmetic-only recent commits)

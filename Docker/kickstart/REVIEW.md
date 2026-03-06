# Docker Kickstart Training - Review

## Overall Assessment

The Docker Kickstart training is a well-structured, progressive curriculum that takes learners from basic container operations through to orchestration with Docker Swarm. The hands-on, task-based approach is effective. However, there are several issues that should be addressed to improve quality, accuracy, and maintainability.

---

## 1. Structural & Navigation Issues

### 1.1 Inconsistent chapter ordering between readme.md and "Next Steps" links

The `readme.md` index lists chapters in one order, but the "Next Steps" links at the bottom of each chapter follow a different path:

- **readme.md order**: setup -> alpine -> webapps-part1 -> images-and-volumes -> webapps-part2 -> devops -> votingapp-compose -> votingapp-swarm -> networking-basics -> secrets -> prometheus
- **Actual "Next Steps" flow**: setup -> alpine -> **webapps-part1** -> **images-and-volumes** -> webapps-part2 -> devops -> votingapp-compose -> votingapp-swarm -> secrets -> prometheus

This matches for most chapters, but the networking chapters (`networking-basics.md`, `bridge-network.md`) are listed in the readme.md but **never linked in the chapter flow**. A learner following "Next Steps" links will skip networking entirely.

### 1.2 Chapters not listed in readme.md

The following chapters exist but are not referenced in `readme.md`:
- `bridge-network.md` - Deep dive into bridge networking
- `mongo.md` - MongoDB deployment
- `mongo-2.md` - MongoDB with Node-RED
- `nodered.md` - Node-RED flow programming
- `docker-devpops.md` - Duplicate/older version of content

### 1.3 Broken "Next Steps" links

- **`networking-basics.md`** (line 221): Links to `./webapps.md` which does not exist. Should likely link to `./bridge-network.md`.
- **`bridge-network.md`** (line 226): Links to `./bridge-network.md` (links to itself). Should link to the next logical chapter.
- **`docker-devpops.md`** (line 99): Links to `chapters/mongo.md` using a relative path that won't resolve correctly from within the `chapters/` directory.

### 1.4 Duplicate content: `docker-devpops.md` vs `alpine.md`

`docker-devpops.md` is essentially a stripped-down, older version of `alpine.md` that uses legacy `docker run` / `docker ps` / `docker images` commands instead of the newer `docker container run` / `docker container ls` / `docker image ls` syntax. It also references "Docker Store" (now Docker Hub) and links to a non-existent `./webapps.md`. This file should be removed or consolidated.

---

## 2. Outdated Content

### 2.1 Deprecated/EOL base images

| File | Issue |
|------|-------|
| `images-and-volumes.md` line 29 | Uses `ubuntu:15.04` in Dockerfile example (EOL) |
| `images-and-volumes.md` line 71 | Pulls `ubuntu:12.04` (EOL since April 2019) |
| `images-and-volumes.md` lines 98, 146 | Section header says "Pull the Debian:Buster image" but actually pulls `ubuntu:jammy`, then uses `debian:stretch-slim` in step 3 (Stretch is EOL) |
| `flask-app/Dockerfile` line 2 | Uses `alpine:3.5` (EOL) |
| `flask-app/Dockerfile` line 5 | Uses `py2-pip` (Python 2, EOL since Jan 2020) |
| `votingapp-swarm.md` line 90 | Uses `postgres:9.4` (EOL since Feb 2020) |

### 2.2 Outdated Docker commands

`docker-devpops.md` (and `prometheus.md` which is identical) use the legacy command syntax throughout:
- `docker run` instead of `docker container run`
- `docker ps` instead of `docker container ls`
- `docker images` instead of `docker image ls`
- `docker inspect` instead of `docker image inspect` / `docker container inspect`

While the legacy commands still work, the main `alpine.md` chapter correctly uses the modern syntax. This inconsistency is confusing.

### 2.3 Outdated GitHub Actions versions

In `devops.md`, the GitHub Actions workflow uses significantly outdated action versions:
- `actions/checkout@v2.3.4` - current is v4
- `docker/setup-qemu-action@v1.2.0` - current is v3
- `docker/login-action@v1` - current is v3
- `docker/metadata-action@v3.4.1` - current is v5
- `docker/build-push-action@v2.6.1` - current is v6

### 2.4 Docker Store references

`docker-devpops.md` references "Docker Store" (`store.docker.com`) which has been merged into Docker Hub. Should reference Docker Hub throughout.

### 2.5 Flask app uses HTTP image URLs

`flask-app/app.py` contains hardcoded HTTP URLs to `ak-hdl.buzzfed.com` for cat GIF images. These URLs are likely broken/dead and use insecure HTTP.

---

## 3. Technical Accuracy Issues

### 3.1 Inconsistent naming in images-and-volumes.md

This chapter has significant naming confusion:
- Line 98: Header says "Pull the **Debian:Buster** image" but the command pulls `ubuntu:jammy`
- Line 127: Refers to the "Docker pull request for **MySQL**" when the section is about **MariaDB**
- Line 133: References "**MySQl** image" and "**Debian:Jessie** image" when discussing MariaDB and Ubuntu Jammy
- Line 137: Says "the **MySQL** image is based on the **Debian:Jessie** base image" - should say MariaDB is based on Ubuntu Jammy
- Line 139: Says "This will import that layer into the **MySQL** image" - should say MariaDB
- Lines 143-206: The hands-on exercise uses `debian:stretch-slim` for container operations, creating confusion with the `ubuntu:jammy` pulled earlier. The section header promised Debian Buster but neither Buster nor Jammy is used in the actual exercise.

It appears this chapter was partially updated from MySQL to MariaDB, from Debian Jessie to Ubuntu Jammy, but the text was not fully updated to match.

### 3.2 alpine.md line 56: Extra backtick

```
and returned its hostname (`888e89a3b36``).
```
Has a double backtick at the end.

### 3.3 images-and-volumes.md: MySQL vs MariaDB confusion in volumes section

The "Understanding Docker Volumes" section (Task 3) uses **MySQL** containers and the `mysql` image throughout, but the earlier parts of the same chapter used **MariaDB**. This is not technically wrong (they're separate exercises) but is pedagogically confusing within the same chapter.

### 3.4 secrets.md line 294: Typo "Kuberenetes"

```
## Docker & Kuberenetes
```
Should be "Kubernetes".

### 3.5 secrets.md line 288: Typo "haven;t"

```
If you haven;t already done so
```
Should be "haven't".

### 3.6 webapps-part1.md: Inconsistent container naming

Step 8 (line 98) runs:
```
docker container rm -f static-site-2 static-site-3
```
But `static-site-3` was never created in the tutorial. This will produce an error.

### 3.7 webapps-part2.md: Task 3 title is misleading

Task 3 is titled "Create your first image" but the learner already created an image in Task 1. A more accurate title would be "Update and version your image" or "Build a new image version".

### 3.8 images-and-volumes.md line 355: Typo in MySQL prompt

```
myslq> show tables;
```
Should be `mysql>`.

---

## 4. Security Concerns

### 4.1 Hardcoded credentials in examples

Several chapters use hardcoded passwords in commands:
- `alpine.md`: `MARIADB_ROOT_PASSWORD=my-secret-pw`
- `images-and-volumes.md`: `MYSQL_ROOT_PASSWORD=supersecret`, `MYSQL_PASSWORD=mysql`

While these are training examples, it would be beneficial to add a more prominent warning about never using hardcoded credentials in production, or to demonstrate using Docker secrets or environment files as best practice.

### 4.2 Running containers as root

No chapter discusses running containers as non-root users (using `USER` directive in Dockerfiles). This is an important security best practice that should at least be mentioned.

---

## 5. Missing Content / Gaps

### 5.1 No multi-stage build coverage

Multi-stage builds are a fundamental Docker concept for creating smaller, production-ready images. This is not covered anywhere in the training.

### 5.2 No `.dockerignore` coverage

The training doesn't mention `.dockerignore` files, which are important for build performance and security.

### 5.3 No container resource limits

No discussion of `--memory`, `--cpus`, or resource constraint flags.

### 5.4 Docker Compose v2 CLI

The `votingapp-compose.md` correctly uses `docker compose` (v2 CLI), but `mongo.md` and `mongo-2.md` use `docker-compose` (v1 CLI with hyphen). Docker Compose v1 reached end of life in July 2023.

### 5.5 No health check coverage in Dockerfile context

While `votingapp-compose.md` covers health checks in Compose files, the Dockerfile `HEALTHCHECK` instruction is never introduced when teaching Dockerfile syntax.

### 5.6 nextsteps.md mentions "Docker Cloud"

Line 2 references "Docker Cloud" which was discontinued. Should reference Docker Hub.

---

## 6. Formatting & Quality Issues

### 6.1 Inconsistent code block formatting

Some chapters use fenced code blocks with language hints, others don't. Some indent code blocks with spaces instead of using fences.

### 6.2 bridge-network.md: Broken markdown nesting

Lines 129-161: Code blocks and command examples are not properly fenced, causing rendering issues. A `###` comment inside a code block breaks the markdown structure.

### 6.3 Inconsistent use of `$` prompt prefix

Most code examples include the `$` prompt prefix, which is good practice for distinguishing commands from output. However, some examples omit it inconsistently.

### 6.4 Image references

`bridge-network.md` line 203 references `concepts/img/browser.png` which appears to be a path from a different project structure and likely won't resolve.

---

## 7. Example Application Issues

### 7.1 flask-app/Dockerfile

```dockerfile
FROM alpine:3.5          # EOL
RUN apk add --update py2-pip  # Python 2 EOL
RUN pip install --upgrade pip
```

This Dockerfile will fail to build as:
- `alpine:3.5` is extremely old
- `py2-pip` installs Python 2 which is EOL
- The `pip install --upgrade pip` may fail due to Python 2 incompatibility with modern pip

**Recommended fix**: Update to `python:3.x-alpine` base image with modern Python 3.

### 7.2 flask-app/requirements.txt

```
flask>=0.12.3
```

Flask 0.12.3 is very old. The minimum should be updated to a supported version (Flask >= 2.0).

### 7.3 static-site/Dockerfile

The Dockerfile uses `sed` in `CMD` to do environment variable substitution at runtime. While functional, this pattern should note that `envsubst` is a cleaner alternative, or better yet, use a templating approach.

---

## 8. Summary of Priority Fixes

### High Priority (blocking or confusing for learners)
1. Fix the naming confusion in `images-and-volumes.md` (MySQL/MariaDB/Debian/Ubuntu mix-up)
2. Fix broken "Next Steps" links in `networking-basics.md` and `bridge-network.md`
3. Update `flask-app/Dockerfile` to use Python 3 (current version won't build)
4. Remove or update `docker-devpops.md` (duplicate, outdated content)
5. Add missing chapters to `readme.md` index

### Medium Priority (outdated but functional)
6. Update GitHub Actions versions in `devops.md`
7. Update EOL base images in examples (ubuntu:12.04, alpine:3.5, postgres:9.4)
8. Replace `docker-compose` v1 commands with `docker compose` v2
9. Fix typos (Kuberenetes, myslq, haven;t, extra backtick)
10. Fix `static-site-3` reference in `webapps-part1.md`

### Low Priority (enhancements)
11. Add multi-stage build coverage
12. Add `.dockerignore` coverage
13. Add `USER` directive / non-root container discussion
14. Standardize code block formatting across all chapters
15. Update Docker Store references to Docker Hub

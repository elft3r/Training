---
---

# Before You Start

Welcome to the Docker Kickstart course! The course starts with a theoretical introduction to containers and Docker, followed by extensive hands-on exercises reinforced by additional theory throughout.

Before the course begins, work through this page to make sure your system is ready.

> **Course Materials**
>
> The slides used during the course are available here: [Docker Training Presentation](https://elft3r.github.io/presentations/docker-training/)

## Checklist

- [ ] Docker Desktop installed and running
- [ ] Git client installed
- [ ] IDE / text editor installed
- [ ] GitHub account created
- [ ] Docker Hub account created

---

## 1. Docker Desktop

All exercises are built around Docker Desktop. Install it for your OS:

- [Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
- [Linux](https://docs.docker.com/desktop/setup/install/linux-install/)
- [Windows](https://docs.docker.com/desktop/setup/install/windows-install/)

_Commands in this course work in bash and in PowerShell on Windows._

**Verify the installation:**

```console
$ docker container run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## 2. Git Client

You will need Git to clone repositories during the exercises. Install one:

- **[Git CLI](https://git-scm.com/downloads)** — works in any terminal
  > GitHub requires a **Personal Access Token (PAT)** instead of your password for HTTPS git operations. To create one: go to **Settings → Developer settings → Personal access tokens → Tokens (classic)**, generate a token with the **`repo`** scope, and use it in place of your password when git prompts you. To avoid re-entering it on every push, run:
  > ```console
  > $ git config --global credential.helper cache
  > ```
- **[GitHub Desktop](https://desktop.github.com/)** — graphical client; handles GitHub authentication automatically via browser login, no PAT needed

**Verify:**

```console
$ git --version
git version 2.x.x
```

---

## 3. IDE / Text Editor

Any editor works. Recommendations:

- **[Visual Studio Code](https://code.visualstudio.com/)** — free, cross-platform, with an excellent [Docker extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
- Any editor you are already comfortable with

---

## 4. GitHub Account

You will need a [GitHub](https://github.com/) account to clone and push repositories during the exercises.

→ [Create a free account](https://github.com/signup)

---

## 5. Docker Hub Account

You will need a [Docker Hub](https://hub.docker.com/) account to pull and push container images.

→ [Create a free account](https://hub.docker.com/signup)

---

## Next Steps

Once all five items are checked off, you're ready to go.

Head over to [Running your first container](./alpine.md) to begin!

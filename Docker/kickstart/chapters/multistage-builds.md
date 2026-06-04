---
---

# Multi-Stage Builds

Now that you understand Docker images and layers, it's time to learn one of the most important techniques for building production-ready images: **multi-stage builds**.

When you build an application inside a Docker image, you often need compilers, build tools, and development dependencies that are not needed at runtime. A single-stage build includes all of that in the final image, making it unnecessarily large. Multi-stage builds solve this by letting you use multiple `FROM` statements in a single Dockerfile — each one starts a new build stage, and you can selectively copy artifacts from one stage into another.

> **Tasks**:
>
> - [Task 1: Build an app with a single-stage Dockerfile](#task-1-build-an-app-with-a-single-stage-dockerfile)
> - [Task 2: Refactor to a multi-stage Dockerfile](#task-2-refactor-to-a-multi-stage-dockerfile)
> - [Task 3: Use named stages and COPY --from](#task-3-use-named-stages-and-copy---from)
> - [Task 4: Target a specific build stage](#task-4-target-a-specific-build-stage)

## Task 1: Build an app with a single-stage Dockerfile

In this task you will build a small Go web application using a traditional single-stage Dockerfile and observe the resulting image size.

1. Navigate to the example app directory inside the training repository:

   ```console
   $ cd Docker/kickstart/multistage-app
   ```

   > **Note:** If you cloned the repository to a different location, adjust the path accordingly (e.g., `cd ~/Training/Docker/kickstart/multistage-app`).

2. Open `main.go` in the example app directory. It is a simple HTTP server that responds with a greeting, hostname, and platform information:

   ```go
   package main

   import (
       "fmt"
       "net/http"
       "os"
       "runtime"
   )

   func main() {
       port := "8080"
       if p := os.Getenv("PORT"); p != "" {
           port = p
       }

       http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
           hostname, _ := os.Hostname()
           fmt.Fprintf(w, "Hello from Go!\nHostname: %s\nPlatform: %s/%s\n", hostname, runtime.GOOS, runtime.GOARCH)
       })

       fmt.Printf("Listening on :%s\n", port)
       if err := http.ListenAndServe(":"+port, nil); err != nil {
           fmt.Fprintf(os.Stderr, "server failed: %v\n", err)
           os.Exit(1)
       }
   }
   ```

   Go compiles to a single self-contained binary (and with `CGO_ENABLED=0` it's fully statically linked), which makes it ideal for demonstrating multi-stage builds.

3. Open the single-stage `Dockerfile.single`:

   ```dockerfile
   FROM golang:1.23

   WORKDIR /app

   COPY main.go .

   RUN go build -o hello main.go

   EXPOSE 8080

   CMD ["./hello"]
   ```

   This Dockerfile uses the full `golang:1.23` image to both compile and run the application.

4. Build the image using the single-stage Dockerfile:

   ```console
   $ docker image build --tag hello-single:1.0 --file Dockerfile.single .
   Sending build context to Docker daemon  4.096kB
   ...
   Successfully built a1b2c3d4e5f6
   Successfully tagged hello-single:1.0
   ```

5. Check the image size:

   ```console
   $ docker image ls hello-single
   REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
   hello-single   1.0       a1b2c3d4e5f6   10 seconds ago   838MB
   ```

   The image is roughly **800 MB** in this example. The exact size will vary by platform/architecture and over time as the `golang:1.23` base image changes, but it is still much larger than necessary because it contains the entire Go toolchain, compiler, standard library sources, and other build-time tools that are not needed to *run* our small application.

6. Verify the app works:

   ```console
   $ docker container run --detach --publish 8080:8080 --name hello-single hello-single:1.0
   ```

7. Test the app:

   ```console
   $ curl http://localhost:8080
   Hello from Go!
   Hostname: a1b2c3d4e5f6
   Platform: linux/amd64
   ```

8. Clean up:

   ```console
   $ docker container rm --force hello-single
   ```

## Task 2: Refactor to a multi-stage Dockerfile

Now let's refactor the build to use a multi-stage Dockerfile that separates the **build** environment from the **runtime** environment.

1. Open the multi-stage `Dockerfile`:

   ```dockerfile
   # Stage 1: Build the application
   FROM golang:1.23 AS builder

   WORKDIR /app

   COPY main.go .

   RUN CGO_ENABLED=0 go build -o hello main.go

   # Stage 2: Create the minimal production image
   FROM alpine:3.21

   WORKDIR /app

   COPY --from=builder /app/hello .

   EXPOSE 8080

   CMD ["./hello"]
   ```

   Let's break down what's happening:

   - **Stage 1** (`FROM golang:1.23 AS builder`): Uses the full Go image to compile the application. The `AS builder` gives this stage a name we can reference later. `CGO_ENABLED=0` ensures the binary is statically linked and doesn't depend on C libraries.
   - **Stage 2** (`FROM alpine:3.21`): Starts a brand-new image from the minimal Alpine Linux base (~7 MB). The `COPY --from=builder` instruction copies just the compiled binary from the first stage into this clean image.

   The key insight is that the final image only contains Alpine Linux and the compiled binary — all build tools are left behind in the discarded first stage.

2. Build the multi-stage image:

   ```console
   $ docker image build --tag hello-multi:1.0 .
   Sending build context to Docker daemon  4.096kB
   ...
   Successfully built f6e5d4c3b2a1
   Successfully tagged hello-multi:1.0
   ```

3. Compare the image sizes:

   ```console
   $ docker image ls --filter "reference=hello-*"
   REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
   hello-multi    1.0       f6e5d4c3b2a1   5 seconds ago    12.1MB
   hello-single   1.0       a1b2c3d4e5f6   3 minutes ago    838MB
   ```

   The multi-stage image is about **12 MB** compared to **838 MB** — a reduction of over **98%**!

4. Verify the multi-stage image works exactly the same:

   ```console
   $ docker container run --detach --publish 8080:8080 --name hello-multi hello-multi:1.0
   ```

5. Test the app:

   ```console
   $ curl http://localhost:8080
   Hello from Go!
   Hostname: f6e5d4c3b2a1
   Platform: linux/amd64
   ```

   The application behaves identically, but the image is dramatically smaller.

6. Clean up:

   ```console
   $ docker container rm --force hello-multi
   ```

> **Note:** Smaller images are not just a nice-to-have. They mean faster pulls, faster deploys, less storage cost, and a smaller attack surface (fewer packages means fewer potential vulnerabilities).

## Task 3: Use named stages and COPY --from

In the previous task you already saw `AS builder` and `COPY --from=builder`. Let's explore these features in more detail.

### Named stages

Each `FROM` instruction in a Dockerfile starts a new build stage. By default, stages are numbered starting at 0. Giving stages meaningful names with `AS` makes your Dockerfile easier to read and maintain:

```dockerfile
FROM golang:1.23 AS builder
FROM alpine:3.21 AS runtime
```

### COPY --from

The `COPY --from=<stage>` instruction copies files from a previous build stage (or even from an external image) into the current stage. You can reference stages by name or number:

```dockerfile
# By name (preferred)
COPY --from=builder /app/hello .

# By stage number
COPY --from=0 /app/hello .
```

> **Note:** You can even copy files from external images that are not part of your build. For example, `COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf` copies the default NGINX configuration file directly from the official NGINX image.

### Practical example: adding a health check

Let's extend our Dockerfile to add a health-check binary from a third stage. The example app directory already contains a small `healthcheck.go` program that makes an HTTP request to our app and exits with a non-zero status if the request fails — exactly what Docker's `HEALTHCHECK` instruction needs:

```go
package main

import (
    "net/http"
    "os"
    "time"
)

func main() {
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get("http://localhost:" + port + "/")
    if err != nil {
        os.Exit(1)
    }
    defer resp.Body.Close()

    if resp.StatusCode != 200 {
        os.Exit(1)
    }
}
```

Now open `Dockerfile.healthcheck`, the three-stage Dockerfile that combines the main app and the health-check binary:

```dockerfile
# Stage 1: Build the application
FROM golang:1.23 AS builder

WORKDIR /app

COPY main.go .

RUN CGO_ENABLED=0 go build -o hello main.go

# Stage 2: Build a health-check binary
FROM golang:1.23 AS healthchecker

WORKDIR /hc

COPY healthcheck.go .

RUN CGO_ENABLED=0 go build -o healthcheck healthcheck.go

# Stage 3: Create the minimal production image
FROM alpine:3.21

WORKDIR /app

COPY --from=builder /app/hello .
COPY --from=healthchecker /hc/healthcheck /usr/local/bin/healthcheck

EXPOSE 8080

HEALTHCHECK --interval=5s --timeout=5s CMD ["/usr/local/bin/healthcheck"]

CMD ["./hello"]
```

This Dockerfile has three stages:
- `builder` — compiles the main application
- `healthchecker` — compiles the health-check tool
- The final unnamed stage — combines both binaries into a minimal image

Build and test it:

```console
$ docker image build --tag hello-hc:1.0 --file Dockerfile.healthcheck .
```

```console
$ docker container run --detach --publish 8080:8080 --name hello-hc hello-hc:1.0
```

```console
$ docker container ls
CONTAINER ID   IMAGE         COMMAND     STATUS                    PORTS
a1b2c3d4e5f6   hello-hc:1.0  "./hello"  Up 10 seconds (healthy)   0.0.0.0:8080->8080/tcp
```

Notice the `(healthy)` status — Docker is running the health check we copied from the second stage.

Clean up:

```console
$ docker container rm --force hello-hc
```

## Task 4: Target a specific build stage

Sometimes you want to build only up to a certain stage — for example, to run tests or get a development image with debugging tools. The `--target` flag lets you stop the build at a specific named stage.

1. Build only the `builder` stage:

   ```console
   $ docker image build --target builder --tag hello-dev:1.0 .
   ```

   This produces an image from the `builder` stage — it includes the Go toolchain and source code, which is useful for development and debugging.

2. Compare sizes:

   ```console
   $ docker image ls --filter "reference=hello-*"
   REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
   hello-dev      1.0       c3d4e5f6a7b8   5 seconds ago    838MB
   hello-multi    1.0       f6e5d4c3b2a1   5 minutes ago    12.1MB
   hello-single   1.0       a1b2c3d4e5f6   8 minutes ago    838MB
   ```

   The `hello-dev` image is the same size as the single-stage build because it contains the full Go image, but the `hello-multi` production image remains tiny.

3. Clean up all images and containers from this tutorial. If any of these containers or images do not exist, Docker may print an error message. You can ignore those errors.

   ```console
   $ docker container rm --force hello-single hello-multi hello-hc
   $ docker image rm hello-single:1.0 hello-multi:1.0 hello-dev:1.0 hello-hc:1.0
   ```

## Terminology

- **Build stage**: Each `FROM` instruction in a Dockerfile begins a new build stage. Stages are independent and start with a fresh filesystem from their base image.
- **Named stage**: A build stage given an alias using `AS <name>` (e.g., `FROM golang:1.23 AS builder`). Named stages can be referenced by `COPY --from` and `--target`.
- **`COPY --from`**: A variant of the `COPY` instruction that copies files from a previous build stage or an external image, rather than from the build context.
- **`--target`**: A flag for `docker image build` that stops the build at a specific named stage, producing an image from that stage.
- **Builder pattern**: The older approach to multi-stage builds, where two separate Dockerfiles and a shell script were used to first build, then copy artifacts into a runtime image. Multi-stage builds replaced this pattern with a single Dockerfile.

## Next Steps

For the next step in the tutorial, head over to [Docker Volumes](./volumes.md)

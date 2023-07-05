---
title: Dockerizing an R App
date: 2023-03-14 19:00:00 -300
categories: [docker]
tags: [R,docker-compose]
---

### Dockerizing an R App: Making It Available to the World

If you have developed an R application and want to dockerize it to make it available to the world, this guide will walk you through the necessary steps. In this example, we will use the project called "animalcules" available on GitHub.

#### Prerequisites

Before proceeding with the steps, ensure that you have the nginx-proxy configured and working in your environment.

#### Step 1: Clone the Project

Begin by cloning the "animalcules" project from the GitHub repository [animalcules](https://github.com/compbiomed/animalcules/) or follow its Dockerfile if available.

#### Step 2: Create a Docker Compose File

Next, create a `docker-compose.yml` file in the project directory with the following content:

```yaml
version: '3.9'
services:
  animalcules:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      VIRTUAL_HOST: "animalcules.example.com"
      VIRTUAL_PORT: "8787"
      PASSWORD: "animalcules"
    entrypoint: "Rscript --vanilla animalcules_16S.R"
```

This file defines a service called `animalcules` that builds the Docker image using the specified Dockerfile. It also sets environment variables for the virtual host, port, and password. The entrypoint command runs the R script `animalcules_16S.R` using `Rscript` with the `--vanilla` flag.

#### Step 3: Modify the R Script

In the R script `runApp.R`, make the following modifications:

```R
# runApp.R
#' Run the animalcules Shiny app
#'
#' @import assertthat
#' @import covr
#' @import lattice
#' @import DT
#' @importFrom shinyjs addClass
#' @return The Shiny app will open
#'
#' @param dev Run the application in developer mode
#'
#' @examples
#' \dontrun{
#' run_animalcules()
#' }
#' @export
run_animalcules <- function(dev = FALSE) {
  appDir <- system.file("shiny", package = "animalcules")
  if (appDir == "") {
    stop("Could not find myapp. Try re-installing `mypackage`.",
         call. = FALSE)
  }
  if (dev) {
    options(shiny.autoreload = TRUE)
  }
  shiny::runApp(appDir, display.mode = "normal", port = 8787, host = "0.0.0.0")
}
```

The modified script ensures that the application listens on all interfaces (`host = "0.0.0.0"`) and exposes the specific port `8787`. This configuration is crucial to ensure that your container app receives incoming requests.

#### Lesson: Listening on All Interfaces

Remember, it is important to configure your R application to listen on all interfaces (`host = "0.0.0.0"`) to ensure it can receive requests from outside the container.

By following these steps, you can dockerize your R app and make it accessible to the world.

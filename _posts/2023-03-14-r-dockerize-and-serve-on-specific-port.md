---
title: Dockerize an R app
date: 2023-03-14 19:00:00 -300
categories: [docker]
tags: [R,docker-compose]
---

### So you have an R app and want to dockerize and made available to world?

In this example we will follow the steps to do that, the project is this:

Steps:
1. Have the nginx-proxy configured and working
2. Clone this project in R: [animalcules](https://github.com/compbiomed/animalcules/) or follow its Dockerfile
3. Make a docker-compose file like this:
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
4. Modify the R Script to listen on all ifaces and expose a specific port instead a rotating one
```R
# runApp.R
#' Run animalcules shiny app
#'
#' @import assertthat
#' @import covr
#' @import lattice
#' @import DT
#' @importFrom shinyjs addClass
#' @return The shiny app will open
#'
#' @param dev Run the applicaiton in developer mode
#'
#' @examples
#' \dontrun{
#' run_animalcules()
#' }
#' @export
run_animalcules <- function(dev=FALSE) {
    appDir <- system.file("shiny", package="animalcules")
    if (appDir == "") {
        stop("Could not find myapp. Try re-installing `mypackage`.",
            call. = FALSE)
    }
    if (dev) {
        options(shiny.autoreload=TRUE)
    }
    shiny::runApp(appDir, display.mode="normal", port=8787, host="0.0.0.0")
}
```

#### LESSON: It is important that you listen on all ifaces because if not your container app will not receive your request

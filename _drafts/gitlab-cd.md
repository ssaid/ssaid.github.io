---
layout: post
title: gitlab-cd
categories:
- docker
- doodba
- k8s
- ansible
- linux
- python
- bash
- gitlab
- github
- networking
- odoo
tags: []
---

https://docs.gitlab.com/ee/ci/variables/

Use dind to build images
https://hub.docker.com/_/docker/tags

Ensure correct permissions

usermod -aG docker gitlab-runner
sudo service docker restart

---

Mask Variable: check if secret data, but for private keys it cannot be masked (set the private key as type file)

Gitlab CI Variables:
ID_RSA: type file, content private_key for ssh in server
DEV_SERVER_HOST 
DEV_SERVER_USER

Disable StrictHostKeyChecking
ssh -o StrictHostKeyChecking=no

Use linter to detect yaml mistakes: https://www.yamllint.com/

When needs artifacts from a job on the same stage: use dependencies
needs: declares that a job must finish before the actual can start & download artifacts from the first job
dependencies: needs an artifact ()

artifacts -> export and load
envvars -> set the envvar and use later in other jobs

---
cache:
  key: "$CI_COMMIT_REF_NAME"

---
https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
CI_COMMIT_REF_NAME: branch name

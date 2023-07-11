---
layout: post
title: docker-tips
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

view all containers logs at the same time:
docker ps -q | xargs -L 1 -P `docker ps | wc -l` docker logs --since 30s -f

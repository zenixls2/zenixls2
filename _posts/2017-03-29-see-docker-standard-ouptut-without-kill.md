---
layout: post
title: "Attach to Docker STDOUT Without Killing The Instance"
description: "To attach to docker instance, remember to set your argument correct..."
tags: [note]
---
For watching the status of dockers, 2 commands are frequently used by me:
`docker ps -a | grep -v "Exit"` and `docker attach --sig-proxy=false {docker_id}`. The first one is to list all the running instances of docker in detail. The second one is to prevent Ctrl+C signal sent internally to docker instance, otherwise you cannot deattach to a foreground running docker easily, and may kill the internal running process.

If you do not want any command line processing interleaving, you may use CloudWatch in Amazon. Even though it's slow and sometimes broken, it keeps your log time and categorized by tag and date.

---
layout: post
title:  "How Slack Can Help Your Team"
date:   tbd 
categories: slack integrations team automation devops
---

Basically, Slack has support for /commands, if you implement them carefully you 
you can end up with some really neat stuff.

I code alone on the backend, I have a Push Service consisting of 5 docker contai
ners within 2 boxes (one for Sandbox, one for Production) currently behind one
ELB on AWS.

Currently my deploy process is:

```bash
git push
ssh slidemail.sandbox
tmux attach – *I already have all logs tail'd there*
cd to sub-project
git pull
make clean container
```

And this is possible because there is only one box per environment. If I had but
one box, it wouldn't be feasible. Imagine repeating that for 10 boxes, every time
you push something to GitHub.

The proposed scenario is:

* n EC2 boxes behind one ELB per each environment
* using /deploy to do a manual deploy on all instances

The proposed solution is:

* `/deploy <env>` sends a request to Jenkins
* Jenkins tests all the projects
* Jenkins puts the built container images on S3
* Jenkins spawns new EC2 instances preconfigured to read their Environment tag
and pull the appropriate containers and load them up
* ELB terminates older instances as part of it's ScaleDown policy
* 

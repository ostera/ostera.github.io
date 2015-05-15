---
layout: post
title:  "Daemons in Docker"
date:   2015-05-15 09:56:04
categories: docker daemons runit unix pragmatism
---

Nowadays I'm using docker for about everything programming. From personal projects,
to consultancy work, to deploying products. But as with any great emerging technology,
there's no silver bullet for everything.

Part of the Docker philosophy is to encapsulate each logical service in a contai
ner, and run as many containers as logical services you need to on a single VM.

> One concern, one container.

But what if your concern spans more than a single long-running process?

It turns out that it's hard use tools like Ubuntu's `upstart` out of the box,
since Docker has a modified `/sbin/initctl` script.

Some people have made complete images that fix this issue, like the smart fellas
at Phusion and their [base-image](https://github.com/phusion/baseimage-docker) –
which with a stunning 2744 stargazers on GitHub seems to be quite popular – that
sets up a set of services for doing this (including `sshd`)

Some other have done quick fixes, similar to [this one](https://github.com/docker/docker/issues/1024#issuecomment-20018600)
pointed out in the Docker issues page, that allow processes relying on the origi
nal `initctl` binary to keep running as if nothing happened by linking it to the
shell built-in `true` command. Hacky, but works.

Michael Crosby, one of Docker's core contributors and authors, [proposed and merged
a pull request](https://github.com/docker/docker/pull/7414) to allow for `--restart`
policies when running `docker run`. This is definitely most interesting, since it
would allow long-running foreground processes to be restarted on different scenarios
and a limited number of times. But it still wouldn't work for background processes.

I personally am in favor of not delegating more than a single task to a single tool,
which seems to be more UNIXy, and ended up using `runit` to keep my processes
alive. Let's see how that works out with an example.

Let's run an HTTP API and a Dropbox uploading service. Two concerns.

Our API has 4 processes behind a reverse proxy with load balancing. Of course,
it could be just one process that directly maps to the external port, and then have
multiple EC2 boxes running one container each being load balanced by Amazon's
Elastic Load Balancer – this way if a process dies, you can just kill the box and
respawn, but that's far more costly than just respawning a process and out of the
scope of this essay.

> Going for "One process, one container", might not always be the way to go.

After some discussion, we settle for 4 daemons for the API process and haproxy as
a daemon too, exposing the haproxy daemon's port to the host machine.

Our Dropbox Service is composed of multiple processes running in parallel:

* A process that listens to an event through a PubSub mechanism prioritizing files.

* A process that consumes a queue of files to upload, ordered by priority.

While we could run each of this as a separate container, they are clearly different
parts of a single logical service and should be living together.

Using `runit` to run this processes within a box allows us to encapsulate a single
concern within a single container. The API runs on it's own, the Dropbox service
runs on it's own.

But what of the database or the PubSub process?

Let's use Redis for both as a database and PubSub. We can't just include it in both
the API and the Service and have it run connected, or at least it's not very clear
how (ie, we could run a master in one and a slave in the other).

What would be rather easy to do is to run Redis as a separate concern, meaning as
a separate container, and link it appropriately to both the API and the Service.

This way we end up with 3 concerns: API, Service, and Database. Each of which runs
on it's container.

A simple `docker run redis:latest` will get you a runnin Redis instance, but it's
running it as a long-running foreground process!

Some of this officially maintained, public images might not fit exactly in your way
of thinking but they do make a point for being not only usable, but also used. While
of course fine-tunning to your needs will require basing your own Dockerfile from
it (or from scratch), for most use cases it's a worthy trade-off.

Maybe it's time to be _less dogmatic and more pragmatic_.

---
layout: post
title:  "Things I Learned at EUC"
date:   2016-02-15 09:56:04
categories: erlang user conference maxims
---

Last year I had the pleasure to attend the Erlang User Conference in Stockholm's
[Munchenbryggeriet](http://m-b.se/en). Great venue. Great technology. Great community.

This is merely a collection of some memorable quotes and some interesting thoughts that I
could jot down between the talks. It all wasn't entirely obvious nor clear to me, as
I was starting out, but it will definitely be obvious to the experienced erlanger.

Perhaps someone out there in the internet will find this useful :)

### General Maxims

* Never trust your systems, nor yourself.
* Always use a common programming style.
* Let errors go UP.

### Erlangisms

* All race conditions should be considered during the design phase.
* Always trap exits and have very consistent way of raising and catching exceptions.
* Always use links from a supervisor to it's children.
* Don't deal with corrupted data.
* Don't handle errors.
* Don't use Erlang for making GUI applications.
* Escalate errors. The sooner the better.
* From functions, message passing, and function calling we model FSM's in Erlang.
* Global registered processes are erlang's take on Singletons.
* Idiomatic erlang is not defensive.
* It's good practice to specify your timeouts!
* It's nice to ! (bang).
* Log the reason of the process exit that's being trapped.
* State is a property of a process.

### Some about the BEAM

* All of the processes do not actively wait. The VM marks a process (because it
knows where the process is when a message is sent to it) as runnable. The scheduler
looks for processes that are runnable, and runs them.

### Some about OTP

* Etop helps you report processes like UNIXes.
* If your handle call/cast/info returns a timeout, it sets a timer. If it returns no timeout, it unsets a timer.
* gen_event could be used to have declarative control flow.
* OTP behaviors have been implemented for some particular use cases that Ericsson thought of
may years ago and can be reused in many situations but it's also very common practice
to implement your own behaviors, because the OTP provided ones are not always meeting
all your requirements. FSM is one good sample. In many systems it's been used but at
the same time is questioned.

### Distributed Erlang

* There's no election for a primary/master node, all nodes are equal.
* epmd: always open  ports for epmd (4369) and net_kernel (54321).
* The erlang tracer can take you SNAFU really easily.

### Questions

> How do you usually keep a vm/release alive? upstart/runit/sysinit?
You can use the erlang vm feature called heart that can spawn a watchdog in the
operating system that keeps an eye on the erlang vm process to restart it.

> Do atoms evaluate at all and if so do they evaluate to themselves?
They do, to themselves.

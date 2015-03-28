---
layout: post
title:  "Designing an Unobtrusive Development Process"
date:   2014-01-19 12:48:03
categories: jekyll update
---

Most of the software development process should be automated, namely test
running, deploys, all that jazz. But in the fast-paced small startup arena,
most people just don't know or don't care about this enough to take the time,
or don't have that time. Even worse, critical areas, such as communication and
planning, are overlooked and not even included in that cycle. So here's my
proposal:

> Implement a simple, continuous deployment process that takes care of your
> software needs (testing, releasing) as well as your project planning needs
> (measuring, reflecting the state of the project), and last but not least,
> your communication needs (who needs to know what and when).

To do this, you will need a few things:

* A Source Code Management tool –a coder-candy wonderland, where your dev team
  will be happy, and GitHub is the perfect service for it.
* A Team Communication tool – where everyone is and can be reached at any time,
  and Flowdock is amazing for that.
* A Project Planning tool – and Trello Boards works just great.
* A Continuous Integration tool – a place identical to your production
  environment where your project will be tested, like Jenkins, Travis-CI, or
  your own little set of scripts.

That you will wire up in a particular manner:

* Make sure all and every test is run before any code reaches a deployment
  server.
* Make sure the deployment server fails eagerly and loudly. The more verbose,
  the better.
* Make sure people get the information they need. Not everyone needs to know it
  all to function efficiently.

Hoping that we get it working, a week or so in after launch, you start to see
more and more random errors occurring randomly, and there's a load of new cards
on Trello, and sometimes developers don't update their cards, and Flowdock
rooms get filled with unnecessary details of commits and pushes that don't
matter outside the development team. Chaos. Well, not really, but going further
into that direction. Let's see how we can fix this.

# Flashcards

To properly reflect the state of your project, and whether you are using SCRUM
or Kanban, or just managing cards in a board, you will have a set of lists. We
will use 4:

* Icebox, or the less urgent stuff
* Backlog, holding the stuff the team will be working on next
* Working, what everyone is doing right now
* Deployed, tasks that are considered to be finished.

If cards are tagged by feature names, it's easy to keep track of the state of
the project. For instance, if you are building a cashflow tool, some of your
top level features would be: Budgets, or Reports. This is incredibly useful
after you grasp which colors are what feature set, which ends up being pretty
obvious:

[![Screen Shot 2014-01-19 at 1.28.03
PM.png](https://d23f6h5jpj26xu.cloudfront.net/1ejhhgelo9f6dg_small.png)](http://img.svbtle.com/1ejhhgelo9f6dg.png)

*Can you guess what feature the yellow tag is?*

I like to use Red for Urgent. Going forward, this can be split into multiple
boards that focus on UI work, or Frontend, or Backend, or DevOps, or whatever
you might need.

Now hooking this up with Github is easy enough. Both services integrate with
each other quite well by using WebHooks, and it rids your devs from having to
go to Trello, and manually move a card from a list to another. This generates a
notification (as we will see in a minute), but even worse, if there was a
deploying error and the card never really got deployed, moving it back to
Working will generate yet another notification –and one more once it has been
deployed and moved to the proper list.

This can be automated to prevent human error, and save your devs some time
spent on moving cards around on a daily basis.

The next thing is to inform the right people about the right things as it
happen.

# A pattern for notifications

Communication is foundational. But that doesn't mean that everyone should know
everything regarding the development process.

> The development team should be aware of *everything* regarding the
> development stack. That means:

* A commit gets pushed.
* A branch gets merged.
* A test fails, and why it failed, and who broke it.
* A deployment begins, and who triggered, and when.
* A deployment ends, and who triggered, and when.
* A deployment fails, and why it failed, and who triggered it, and when.
* An error occurred on the production or staging server, and a full trace of
  the error.
* A new server spawned.
* A server was killed.
* KERNEL PANIC, or any other issues that will arise.

> While your mileage may vary, the rest of the people in the company might just
> need to know the following:

* The staging server is under maintenance.
* The staging server is up.
* The staging server is down.
* A new feature has been deployed.

This way, you keep your team updated on the stuff that matters to them. If at
one point you separate your development from your DevOps team, then most likely
an infrastructure error involving a virtual machine not being able to spawn is
not of interest to the development team in the same way a failing test for a ui
component is of no interest to the DevOps team.

But now, how do we automate such a process in a way that it can pipe
information –and not just any, but the *right* information– to the right
people?

If we had a continuous deployment system, it would be dead easy to hook up
calls to Flowdock's API so that it would announce something to any particular
groups (like your Main chat, or your Dev chat).

# A Continuous Deployment Process

Continuous Deployment is by no means something new. Everyone knows about it or
at least heard the term once or twice before. In a nutshell:

* the dev team pushes
* the cd server tests
* the cd server deploys

And that happens *all the time*. There are lots of services out there, but
today we will abstract it out. You can use Jenkins and set it up yourself, or
try Travis-CI.

But how does this answer our prior question? Not so obvious at first, so I'll
use an example:

> A new card gets added to your Trello Board, in the Backlog list. It reads
> "Sign up form should be a modal instead of a page", and it was the card
> `#231`, tagged with yellow (meaning "Users").

At this point, there has been a notification in the Development room in
Flowdock announcing that a new card was added. This is possible thanks to a
Trello-Flowdock integration. Look it up, it's quite easy to setup too.

> One of the engineers finishes off his last task and seeing that he has
> nothing to do and it's still 2 till 5, assigns himself the task and moves it
> over to the Working list.

And yes, notifications in Flowdock inform the team that Johnny, who had been
working on yellow-tagged cards all week, is working on that card now.

> Time passes and Johnny, as the good engineer he is, wrote a couple tests and
> the code to make it all work. That was a few commits on git, the last one
> reading "Closes #231". He merges his feature branch into master, the tests
> pass, and he pushes the updated master branch to Github.

Now the dev team sees that he pushed some code. They can pull in his changes
and rebase their branches with the new master branch. This is possible thanks
to Github-Flowdock integration. They are aware that any second now, the
Continuous Deployment server will start munching bytes. 

> John's code reaches Github, which in turn lets the CD server know that
> there's new code, and it pulls it, sets it all up, and runs the tests.
> Everything seems to be fine as the almost 200 tests are about to finish
> running when, unexpectedly, the last two fail.

`Deployment interrupted: Test #199 failed.` – Reads the headline of the
notification in the Flowdock room for development. They can also see a stack
trace and a test report stating what failed, why, what commit broke it, and
who's commit that was. Why who? He who broke it is most likely the guy to fix
it.

This is possible thanks to the Flowdock API, to which the CD server has access
to, and can literally push notifications to chat rooms that need to know at any
time, of about anything really. Be it some critical data or an inside joke.

> So John sees that it wasn't a typo, for the whole test suite ran ok just
> before the code left his machine to travel through the internet of things. It
> seemed to be an issue with his development setup being different from the one
> used everywhere else. 'Ha.' – he thought. And started digging.

And that's a story for another post. But now let's imagine that the test ran
ok, and that the CD server ran the whole test battery just fine.

> John's code reaches GitHub, which in turn lets the CD server know that there
> is new code, and it pulls it. It sets it all up, and runs the tests.
> Everything seems to be fine as the almost 200 tests battery is about to
> finish. And it finishes alright.

Again thanks to the Flowdock API, a notification will be sent to the
development team showing that the deployment was a success. The staging
environment will be updated (and everyone in Flowdock will see the notification
in the Main room). Ultimately the Card is set to Deployed by the CD server once
it finishes the deploy.

Now that's a happy ending.

# Caveats and Implementation

Of course, this story is less a cautionary tale and maybe more a
recommendation. This is what I consider to be an unobtrusive development
process that helps the organization as a whole, specially when
multidisciplinary teams are formed.

Using the services described, it's easy enough to get a similar setup. Just
make sure that:

1. Github commits can move Trello cards.
* Github commits and pushes show only in the Flowdock rooms that matter.
* Trello card updates only show in Flowdock to the users involved.
* The CD server informs people over Flowdock when things happen.
* The CD server can move Trello cards.

In the next post I will implement this unobtrusive process using the
aforementioned services and AWS *–SQS for deploy queues, hooking up SNS/SES for
notifications, EC2/S3 for the actual deploy system, and maybe another fancy
little things like complete disk snapshots to replicate errors or whatnot but
I'm gonna stop right here or I'll spoil everything*.

# Summary

These are my thoughts on this matter. They are not final, as I'm constantly
looking for flaws and improvements, so feel free to reach out on twitter and
let me know what you think would improve this draft. Hopefully this got you
thinking about your development process as well.

Cheers!

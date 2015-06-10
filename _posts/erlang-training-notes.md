State is a property of a processs.

Don't deal with corrupted data.

Log the reason of the process exit that's being trapped.

Always trap exits and have very consistent way of raising and catching exceptions.

Never trust your systems, nor yourself.

Idiomatic erlang is not defensive.

Don't handle errors.

Always use a common programming style.

Golden Rules:
1. Your function should be bigger than 5 lines of code.
2. Lighweight processes.
3. 

Event Handlers register for a Stream <-
  each behavior has a set of callback functions
  that allow us to handle requests

For gen_fsm each state has a one to one relationship with a function. So you
have a function that relates to a state. And it heavily relies on their names.

Dialyzer -> type annotations help you typecheck your functions.

Let errors go up!

It's nice to ! (bang).

A proper list always ends with an empty list.
An inproper list ends with anything but an empty list.

-compile(export_all). is actually useful when writing tests.

Links are bidirectional.

Always use links from a supervisor to it's childs.

Escalate errors. The sooner the better.

Timeouts are another thing that makes erlang software robust.

after clauses require a Number, however that's evaluated, or the infinity atom.

It's good practice to specify your timeouts!

If your handle call/cast/info returns a timeout, it sets a timer.
If your handle call/cast/info returns no timeout, it unsets a timer.

Sys is your friend. Sys:log will help you log OTP events and messages that OTP compliant
processes can understand.

For tracing you'dl ike to use the erlang tracer that comes with the vm.

Erlang uses reductions to determine how they'it'll run processes.
Etop helps you report processes like UNIXes.

All of the processes do not actively wait. The VM marks a process (because it 
knows where the process is when a message is sent to it) as runnable. The scheduler
looks for processes that are runnable, and runs them.

A property list is a list of tuples with an atom and a value:
  [{key, value}, ...]

Global registered processes are erlang's take on Singletons.

There's no election for a primary/master node, all nodes are equal.

OTP behaviors have been implemented for some particular use cases that Ericsson thought of
may years ago and can be reused in many situations but it's also very common practice
to implemenet your own behaviors, because the OTP provided ones are not always meeting
all your requirements. FSM is one good sample. In many systems it's been used but at
the same time is questioned.

From functions, message passing, and function calling we model FSM's in Erlang.

All race conditions should be considered during the design phase.

erlang tracer can take you SNAFU.

epmd: always close ports from epmd (4369) and net_kernel (54321).

Questions:
- How do you usually keep a vm/release alive? upstart/runit/sysinit?
  You can use the erlang vm feature called heart that can spawn a watchdog in the
  operating systme that keeps an eey ein the erlang vm process to restart it.

- Do atoms evaluate at all and if so do they evaluate to themselves?
  They do, to themselves.

Supervisor Specification: Nested compound data type.

Don't use Erlang for making GUI applications.

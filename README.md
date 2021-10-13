# Lab 3 and Lab 4: Raft
To get started go to [https://classroom.github.com/a/Wsj-WCCp](https://classroom.github.com/a/Wsj-WCCp)
to create a repository and get the template code.

## Introduction
In this lab you will implement a replicated state machine using Raft as the
underlying consensus protocol. The RSM you are implementing models a queue
supporting enqueue and dequeue operations. The RSM also provides a no-op
operation that is used for testing and forcing commits.

**NOTE:** This project is significantly more complex than Lab 1 and 2. Despite
being split across 2 labs, we expect that you will need to spend significant time
on completing this. You should start early, and really spend some time thinking
about how to best implement this code. The stencil code itself is very long, and
reading through it will probably take a while.

**NOTE 2** Figure 2 in the [Raft paper](https://cs.nyu.edu/~apanda/classes/fa20/papers/ongaro14in.pdf)
is exactly what your implementation should do. If in doubt, you should figure out
how and why you implementation differs from what is described in Figure 2.

You do not need to implement:
* Snapshotting or anything else to reduce log sizes.
* Reconfiguration or mechanisms to change membership.

For this project you can safely assume that the network does not lose packets and
that messages between pairs of processes are delivered in order.

## Lab 3: Get the RSM Working without Leader Election (AppendEntries)

For Lab 3 you must write all of the code necessary to launch a cluster with
a pre-appointed leader and allow clients to commit changes. In particular this
requires

* Implementing AppendEntries so the leader can replicate log entries across the
  cluster.
* Making sure followers respond to AppendEntries appropriately.
* Making sure the leader can decide when an entry has been committed. The leader
  **should only** respond to the client once the appropriate operation has been
  applied to the queue, which of course requires that the corresponding log entry
  be committed. To help with this LogEntry's store information about the client
  that invoked a request.
* Making sure that followers apply committed log entries eventually.

While you do not need to implement Heartbeats as a part of this portion, we strongly
recommend doing so since the logic closely mirrors bits you will be implementing.

Also, you can start with the assumption that all processes are either leaders or
followers, and hence not worry about candidates in this part of the lab.

### Testing Lab 3
We have test cases for Lab 3 in [`test/lab3_test.exs`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/test/lab3_test.exs)
which can be run using:

```
> mix test test/lab3_test.exs
```

You are definitely encouraged to add additional tests.

## Handing in Lab 3

**WARN WARN WARN** PLEASE READ THESE INSTRUCTIONS CAREFULLY. YOU MAY **RECEIVE
A 0 (ZERO) IF YOU DO NOT**, EVEN IF YOU COMPLETE EVERYTHING THUS FAR.


To handin this assignment:

* First make sure `mix test` shows that you pass all tests. If not be aware
  that you will loose points.
* Second, make sure you have updated this `README.md` file. This requires
  providing line numbers for the test you added in Part 2, potentially adding
  implementation notes to Part 3, and filling out the information below.
* Commit and push all your changes.
* Use `git rev-parse --short HEAD` to get a commit hash for your changes.
* Fill out the [submission form](https://forms.gle/GmjyEesJSpkQoaLd7) with
  all of the information requested.

We will be using information in the submission form to grade your lab, determine
late days, etc. It is therefore crucial that you fill this out correctly.

Github username: (e.g., apanda)

NYU NetID: (e.g., ap191)

NYU N#:

Name: 

### Citations

## Lab 3 Grading
We will be grading Lab 3 and Lab 4 at the same time. Half (50%) of your Lab 3
grade will be determined by how the final Lab 4 handing works with the lab 3
tests, while the rest will be determined the handin for Lab 3. Lab 3 largely
exists to act as a forcing function fo your to being working on this lab.

## Lab 4: Get Leader Election Working
In this second part you will complete your implementation so that leader election
works correctly. This requires implementing everything required to handle 
`RequestVote` calls correctly, and for getting logs to match.


### Testing Lab 4
We have test cases for Lab 4 in [`test/lab4test.exs`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/test/lab4_test.exs)
which can be run using:

```
> mix test test/lab4_test.exs
```

You are definitely encouraged to add additional tests.

## Handing in Lab 4

**WARN WARN WARN** PLEASE READ THESE INSTRUCTIONS CAREFULLY. YOU MAY **RECEIVE
A 0 (ZERO) IF YOU DO NOT**, EVEN IF YOU COMPLETE EVERYTHING THUS FAR.


To handin this assignment:

* First make sure `mix test` shows that you pass all tests. If not be aware
  that you will loose points.
* Second, make sure you have updated this `README.md` file. This requires
  providing line numbers for the test you added in Part 2, potentially adding
  implementation notes to Part 3, and filling out the information below.
* Commit and push all your changes.
* Use `git rev-parse --short HEAD` to get a commit hash for your changes.
* Fill out the [submission form](https://forms.gle/pY9KSPsK1RyTTK7DA) with
  all of the information requested.

We will be using information in the submission form to grade your lab, determine
late days, etc. It is therefore crucial that you fill this out correctly.

Github username: (e.g., apanda)

NYU NetID: (e.g., ap191)

NYU N#:

Name: 

### Citations

## Hints and Code Structure

The stencil code for this project is quite extensive. We recommend reading and
understanding it, particularly the helper functions before beginning work. 
Below we provide a few notes that might help you as you look through the code:

* We represent RPC calls and their returns using Elixir structs. These structs
  are defined in [lib/messages.ex](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/messages.ex)
  and you can see examples of their use in [raft.ex](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L596).
  
  Utility functions in [raft.ex](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex)
  including `get_last_log_index`, `get_last_log_term`, `save_election_timer`, etc.
  demonstrate how you can manipulate these structs.

* Per-process state is defined in [raft.ex](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L22)
  which also provides a [description](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L37)
  of how the log is stored. Please make sure you read and understand that comment
  since it is important that you either (a) follow that log structure or (b)
  appropriately change the utility functions.
  
* Log entries can be committed by calling the [`commit_log_entry`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L128)
  or [`commit_log_index`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L157)
  function.

* You should use the [`save_heartbeat_timer`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L331)
  and [`save_election_timer`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L331) functions
  to store handles to the appropriate timers in process state. These handles are
  of course useful when cancelling timer.
  
* You are **required** to implement the [`reset_election_timer`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L331)
 and [`reset_heartbeat_time`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L331).
 Each should cancel any existing timer, and set a new one. You **must** use 
 `state.heartbeat_timeout` as the heartbeat timeout, and call 
 [`get_election_time`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L316)
 to get the **election** timeout.
 
* The code is structured so that each processes code appears as a state machine:
  * A process can be in one of three states: [`follower`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L394),
    [`leader`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L525) and
    [`candidate`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L686).
    Each has a function that you should fill out with appropriate information.
    
    Of course some of the logic is shared across the three, and so you should make
    use of additional private functions to avoid code duplication.
    
    Each of these functions take both process state (`state`) and an additional 
    `extra_state` parameter that you can use to track anything specific to a particular
    type of process. For example, a candidate can use the `extra_state` parameter to
    track the number of votes it has received.
    
  * We provide three functions [`become_leader`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L509),
    [`become_follower`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L378),
    and [`become_candidate`](https://github.com/nyu-distributed-systems/fa20-lab3-code/blob/master/apps/lab3/lib/raft.ex#L670)
    that should be called when transitioning from one process state to another. You
    should used these functions to implement any one-time processing required,
    for instance you might want to add logic in `become_leader` to (a) start a
    heartbeat timer, and (b) assert leadership using an empty AppendEntry message.
    
    You should make sure your transitions call these `become_*` functions. For
    example when implementing the election timeout on a follower your code should
    roughly do the following:
    
    `follower` -(timer)-> `become_candidate` -----> `candidate`
    
    You might want to send out `RequestVote` requests in `become_candidate`.

* Finally, start early, read through the code you are given, and write plan things
  out on paper before you start writing code.

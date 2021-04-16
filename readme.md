# Node Immortal

Node Immortal is a Node library for making your Node application immortal. You
use it just by including its ES module and calling `await immortal()`:

```javascript
import immortal from 'https://github.com/tomashubelbauer/node-immortal';

await immortal();
```

With this setup, you run your application normally, using `node .` and enjoy the
features Node Immortal provides:

## Features

### Keeps your application single-instance without failing on new runs

Node Immortal keeps your program single-instance. If no instance is running yet,
the one your start becomes the sole one. If there is one instance running, its
standard I/O streams will be propagated through your current invocation so that
you can interact with the existing application.

### Detaches your Node process from the terminal it was started in

When you close the terminal, your process will continue to live on. To view your
program's logs, just open a new terminal and restart the application.

## Retaches your Node process to the terminal you restart it from

You can start your application from the terminal, close the terminal without
worrying about program closure, restart the application another time and all
throughout, your process will stay alive.

## Restarts itself on source code changes (source code reloading)

This feature improves the development experience. The edit-compile-test cycle
gets reduced to an edit/test loop.

## Restarts itself on crash to keep the process running without interruption

This feature also improves the development experience. Crashes should be noticed
to not go ignored, but during development, there will be many, especially with
the source code reloading leading to syntax errors / inconsistent code as the
files are being saved. To bridge over these intermitted inconsistencies, Node
Immortal will keep the process alive across crashes.

## Lineage

Node Immortal is culmination of proof of concept experiments I've done in:

- [`node-forever`](https://github.com/TomasHubelbauer/node-forever)
- [`node-phoenix`](https://github.com/TomasHubelbauer/node-phoenix)

## Status

This project is in draft state. I have no written any code here yet, but what
ultimately ends up here will be a combination of the above two projects.

## Method

To implement the features of Node Immortal, the following techniques are used:

Two processes are used: an overseer responsible for managing the worker and the
worker process itself.

When starting up the program the user uses Node Immortal in, through the
`await immortal()` call, Node Immortal starts up. The first course of action is
to determine whether there is already a process running.

If there is no process running yet, this process will become the overseet and it
will not return from `immortal`, so the code following the `await immortal()`
call never runs. That code is user code which is meant to run in the worker.
The overseer will fork off a worker process and act as a conduit between the
worker process' standard I/O streams and the terminal the overseer process was
started from.

The worker will tell it is a worker by checking `process.send` on startup. It
will return from the `await immortal()` call without doing much of anything to
let the user code run.

From this point on, the overseer will monitor the worker and restart it in case
it crashes. The crash will be logged into the standard output of the overseer,
so the event is visible to the user. If there is no attached terminal, no report
of the crash will be recorded, so it is up to the user to do crash logging (any
sort of logging, actually).

The overseer will also monitor the directory of the process (the current working
directory assuming the process has been started from where its source is) and on
changes, will restart the worker so the worker always has up to date source code
whenever the source code changes.

At this point, the source code and crash monitoring features are taken care of.
The user can edit code and use the application without worrying. Since both the
overseer and the worker processes are detached from the terminal, they will both
keep running if the terminal is closed, taking care of the terminal detachment
feature as well.

If the overseer process did find an existing overseer process while starting up, 
it will kill it so that the application remains single instance. The worker
process is also killed and started anew. This ensures that the currently running
oversees process shows the correct I/O in the terminal and as a consequence, the
terminal re-attachment feature is taken care of, too.

## To-Do

### Implement the proof of concept

Use the projects mentioned in the [Lineage](#lineage) section to put together
the code for this project.

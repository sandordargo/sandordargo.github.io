---
layout: post
title: "The Many Ways to End a C++ Program"
date: 2026-11-18
category: dev
tags: [cpp, exit, abort, terminate]
excerpt_separator: <!--more-->
---
Reading through [Patrice Roy's *C++ Memory Management* book](https://amzn.to/4l8fGzq), I didn't only learn about memory management, but other topics as well. Some topics he couldn't go into in detail, as it would have been a detour, but they sparked my curiosity to explore further.

In one paragraph, he mentions that *"a C++ program can conclude in many different ways"* and quickly enumerates those options.

In this article, I want to go a bit deeper and look into each of them with some examples.

<!--more-->

## `return` from `main`

The most obvious way to conclude a program is to `return` from `main()` with an exit code. In most examples you see this:

```cpp
int main() {
	// your code
	return 0;
}
```

This implies that `0` means success and anything else means failure. In the `<cstdlib>` header, you'll even find some constants defined for this: `EXIT_SUCCESS` (typically `0`) and `EXIT_FAILURE` (typically `1`). Anything else is implementation-defined.

There are a couple of things worth noting:
- If you omit the `return` from `main`, it's treated as `return 0`.
- On POSIX, the exit status is an 8-bit unsigned value, so `256` becomes `0` - which means success. There are also plenty of conventions around exit codes. The safest advice: don't go beyond `125` if you want to express your own failure codes, as higher values are often reserved by shells and signal reporting conventions.

But what happens when you return from `main`?

First, what you'd expect:
- Local objects in `main` are destroyed.
- Then the return value is used as if `std::exit` were called with it. For the details of that, read on.

## `std::exit(int)`

Let's continue naturally and walk through what `std::exit` does.

First, `std::exit` destroys thread-local objects associated with the current thread. Thread-local objects belonging to other threads are not destroyed. In a multithreaded program this is a real concern: other threads may be mid-execution when `std::exit` is called, and their thread-local destructors simply never run.

Then it destroys static objects interleaved with executing `atexit` handlers in reverse order of construction / registration. So if you construct `static A`, register `atexit_fn`, then construct `static B`, the sequence is: `~B → atexit_fn → ~A`.

> You can register any function pointer to be executed during `std::exit` with `atexit(func)`. If you register more than one, the most recently registered one will be executed first.

It flushes and closes C `FILE*` streams, and C++ standard streams are flushed as part of static object destruction.

It removes temporary files created by `tmpfile()`.

And finally, it terminates the program and returns the status to the host environment.

This means that if you call `std::exit` from deep inside a function, local variables are not destroyed. The memory will eventually be reclaimed by the OS, but destructors are not executed.

It seems obvious, but it's worth noting that any code after `std::exit(int)` is unreachable.

Also, calling `std::exit` from a static destructor or an `atexit` handler is undefined behaviour. You cannot reenter the shutdown sequence that is already in progress.

So when would you use `std::exit`?

You might use it at the very beginning of `main`, before setting up any local objects. For example, validating the arguments passed:

```cpp
int main(int argc, char** argv) {
    if (argc < 2) {
        std::cerr << "usage: " << argv[0] << " whatever help message...\n";
        std::exit(1);
    }
    // ... real work
}
```

You might also use it in a parsing helper that prints some help text and exits.

You might write your own custom assertion - call it `fatal_error` - for situations where you want no recovery. In such places, you might prefer `std::abort` instead (more on that below):

```cpp
[[noreturn]] void fatal_error(const char* msg) {
    std::cerr << "fatal: " << msg << "\n";
    std::exit(1);
}
```

My goal is not to enumerate all possible uses, just a few major ones. One important rule to highlight: in a library, you should never call `exit` - unless you explicitly declare that you own `main`.

## `std::quick_exit(int)`

It was added in C++11 to solve a specific problem: static destruction can deadlock or crash, and sometimes you need to terminate without going through it.

Here is an example: a static destructor accesses another static object that has already been destroyed - the static destruction order fiasco. You crash during cleanup, which is frustrating because the program had successfully done its real work and was just trying to leave.

Another interesting example is when you simply need a fast exit without closing and flushing a potentially huge number of files.

So what does it do?

It calls handlers registered with `std::at_quick_exit` in last-in-first-out order, then calls `std::_Exit` to terminate. It doesn't destroy local objects, thread-local objects, or static objects. It doesn't call `atexit` handlers. It doesn't flush or close standard streams, and it doesn't remove temporary files.

It only executes its own registered handlers and returns the status to the host environment.

## `std::abort()`

This is the "stop everything right now, something is wrong" termination. In a way, it's the most violent of standard exits. Its main characteristic is that it leaves forensic evidence.

It raises the `SIGABRT` signal. Even if there is a handler for that signal and it returns normally, `abort` continues anyway and terminates the program. The one edge case worth knowing: a signal handler that uses `longjmp` to escape rather than returning can technically side-step abort. This is non-portable, interacts badly with C++ destructors, and is never the right approach.

You typically use it for failed assertions - states that should be impossible - so you want to stop any risk of doing further harm. You halt and let the developer see what happened. Besides asserting logical invariants, you might call `abort` if you detect undefined behaviour or memory corruption.

`abort` doesn't run any destructors, doesn't call the different exit handlers, isn't guaranteed to flush or close streams, and leaves temp files untouched.

It simply causes an abnormal program termination, something the OS differentiates from a normal exit.

It typically produces a core dump and breaks into debuggers if one is attached.

## `std::terminate()`

It gets called by the runtime in specific failure modes. It's saying *I don't know what to do, I'm giving up*.

By default, it calls `std::abort`, but with `std::set_terminate` you can set any callable that will be invoked when `std::terminate` is called. There are not many constraints - it must not return or let any exceptions escape. Otherwise, `std::abort` will be called anyway.

The standard lists quite a few cases when `std::terminate` is called. Here are a few of them:
- An exception is thrown but no handler catches it.
- An exception escapes a `noexcept` function.
- [An exception escapes a destructor during stack unwinding](https://www.sandordargo.com/blog/2026/04/01/when-a-destructor-throws).
- An exception is thrown during the construction or destruction of a static or thread-local object.
- An `atexit` or `at_quick_exit` handler throws an exception that escapes the handler.

## `std::_Exit(int)`

There is a more stripped-down way to end a program than `std::abort`, and that is `std::_Exit(int)`, declared in `<cstdlib>`. Both bypass destructors and atexit handlers. The difference in cleanup: `abort`'s handling of open streams is implementation-defined - the standard makes no guarantees either way. `_Exit` is stricter: it is guaranteed to perform no cleanup whatsoever, no flushing, no removal of temporary files.

Another difference between `abort` and `_Exit` is that the former raises `SIGABRT` and typically produces a core dump. `_Exit` just returns a normal exit status with a code passed by the call site. Choose `_Exit` over `abort` when you specifically want a clean exit status - so the parent process doesn't think you crashed - but you still can't afford the cleanup that `exit` or `quick_exit` would do.

## Conclusion

A C++ program can end in many different ways, and the differences matter.

`return` from `main` is the normal case: local objects are destroyed, and the standard `std::exit` sequence runs. `std::exit` handles static and thread-local destruction, `atexit` handlers, and stream flushing - but skips local objects if called from deep inside a function. `std::quick_exit` skips all of that and only runs its own registered handlers, useful when you want to avoid the risks of static destruction. `std::abort` is the emergency stop: it raises `SIGABRT`, produces a core dump, and leaves everything as-is for post-mortem debugging. `std::terminate` is the runtime's last resort and by default delegates to `std::abort`. And `std::_Exit` is the most bare-bones option: it exits with a given status and does nothing else.

Each has a specific purpose. Choosing the wrong one can mean missing cleanup, confusing the parent process, or losing valuable crash information. Knowing the difference is the kind of thing that separates a developer who just uses C++ from one who truly understands it.

{% include connect-deeper.html %}

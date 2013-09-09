# Chapter 1: The Process Model

The Android Operating-System has, at its heart, a heavily modified Linux kernel, designed to securely and efficiently run multiple virtual machines on devices with relatively small memory and CPU power.

Android applications are typically programmed using the Java language, but the virtual machines in the Android stack are not instances of the Java Virtual Machine (JVM).

Instead the Java code is compiled to Java byte-code then translated into a Dalvik executable for execution on a Dalvik virtual machine (DVM).

## Why Dalvik?

The Dalvik Virtual Machine was created specifically for Android, and as such was designed to operate in memory, processor, and power constrained environments such as mobile devices.

Satisfying these design constraints resulted in a very different virtual machine than the typical JVM's we know from desktop and server environments.

### Stacks vs Registers

The most fundamental difference between the two VM architectures is that the JVM is a stack-based machine, whereas the DVM is register-based.

A stack-based virtual machine must transfer data (variable values) from registers to the operand stack before manipulating them.

In contrast a register-based VM operates directly using registers, reducing the number of instructions that must be executed to achieve the same result, at the cost of larger instructions (2 bytes in Dalvik, vs. 1 byte in the JVM).

The cost of these larger instructions is largely mitigated by reading both bytes together, so the net result of this architectural difference is that the DVM is, on average, around 30% more memory and CPU efficient than the JVM.

### Memory Sharing and the Zygote

Another huge efficiency of the Dalvik VM is brought about by the way in which a new VM instance is created and managed.

When the Android system boots, a special process called the Zygote is launched. 

The Zygote starts up a virtual-machine, pre-loads the core libraries and initialises various shared structures, then waits for instructions by listening on a socket.

When a new Android application is launched the Zygote receives a command to launch a VM in which to run that application, which it does by "forking" its pre-warmed VM process.

The forked child process benefits in two ways: first, the virtual machine and core libraries are already pre-loaded into memory and thus the startup overhead is minimised; second, access to the memory containing the core libraries and common structures is shared by the Zygote with all other applications, resulting in large memory savings when running multiple apps.

## The Process/Thread Model

When an Android application is installed a new unix user is created specifically for it, and the application and its data are then owned by that new user. 

In this way, each application's files are protected from interference by other applications at the level of the Operating-System.

When the Zygote forks a new process, ownership of the forked process is transferred to the user that owns the application, which effectively sand-boxes each running application.

Each such process runs independently of other processes in the system, and is scheduled frequent small amounts of CPU-time by the Operating-System.

Within a process there may be many "threads", sometimes known as light-weight processes. These threads are also managed and scheduled by the Operating-System. 

Any threads running within a single process can communicate and share data with other threads running within the same process.

### The Main Thread

Within the DVM process there is a single main thread of execution, known as the "main" or "UI" thread. By default all code which you write in your application will be executed by the main thread.

For example, when we write code in an `Activity`'s onCreate method, it will be executed on the main thread. 

Likewise when we attach listeners to user-interface components to handle taps and other gestures, the listener callback executes on the main thread.

This raises an interesting question: *What happens if we perform long-running operations on the main thread?*

### The ANR Dialog

As you can imagine, if the main thread is busy reading data from a network socket it cannot immediately respond to user-input.

If it does not respond to user-interaction within a few hundred milliseconds the app will feel "laggy". Taken to the extreme, if it takes a long time to respond - several seconds, say - it will feel as though the app has hung.

This is such a pernicious problem that the Android platform protects users from applications which do "too much" on the main thread.

*If an app does not respond to user-input within 5 seconds, the user will see the "Application Not Responding" dialog or ANR, and be offered the option to quit the app.*

The ANR is to be avoided at all costs, as the poor user-experience translates into bad reviews and an unpopular application. Thus, a simple rule to live by when building Android applications is: 

    *do not block the main thread!*

[TIP] For developers, Android provides a helpful setting in the "Developer Options" on each device: "strict mode". In this mode, the screen will flash when applications perform long-running operations on the main thread.

## Concurrency in Android

The solution to ANR is, of course, to do as much work as possible *off the main thread*.

Since Android applications are written in Java, and the Android thread-model is essentially the Java thread-model, the low-level constructs of concurrency in Android are those provided by the Java language: java.lang.Thread, java.lang.Runnable and the synchronised and volatile keywords are all available and behave as expected.

Likewise, higher-level concurrency constructs provided by the java.lang.concurrent package are available, and use of them where appropriate is highly recommended.

You can start threads in your Android application just as you would in any other Java application, and the Operating-System will schedule those threads some time on the CPU. 

Clearly then, Android applications can benefit from concurrency just like any other Java application, and they also face the same concurrency issues, which fall into the two broad categories of correctness and liveness.

There are, however, two problems facing developers of concurrent Android applications which are specific to Android:

1. Long-running operations vs. the Activity Life-cycle.
2. You can only manipulate the user-interface from the main thread.

### The Activity Lifecycle

Android applications are typically composed of one or more subclasses of `android.app.Activity`. An Activity has a very well defined lifecycle, which the system manages through the execution of lifecycle method callbacks, all of which are executed on the main thread.

An Activity which is no longer the focus of the running application should be eligible for Garbage Collection, but it is very easy to unintentionally create memory leaks when off-loading work to background threads.

### Manipulating the User-Interface

The second Android specific problem lies not in what you *can* do off the UI thread, but in what you can *not* do.

*You can not manipulate the user-interface from any thread other than the main thread.*

This is because the user-interface toolkit is not thread-safe, and the system protects itself from potential problems by actively denying access to user-interface components from threads other than the one that originally created the components.

The challenge, then, lies in communicating the results of work done on a background thread back to the main thread, in order to update the user-interface.

### Android-specific Concurrency Constructs

The rest of this book deals with the Android specific constructs for doing background work and re-integrating the result with the main thread. They are:

_AsyncTask_ - ideal for short-lived background operations that can provide progress updates and re-integrate easily with the main thread.

_Handler, HandlerThread and Looper_ - related and fundamental concurrency concepts in the Android platform for submitting work to be carried out by specific threads.

_Loaders_ - a special construct for populating certain types of view with data.

_IntentService_ - a mechanism for instigating and performing background work outside of the scope of a single Activity's lifecycle.

_Service_ - a powerful construct for performing long-running background operations.






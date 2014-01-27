### Matlab is blocked
processManager relies on the ability to call Java functions from within Matlab. In particular, it uses the Java [Runtime object](http://docs.oracle.com/javase/6/docs/api/java/lang/Runtime.html) to create [subprocesses](http://docs.oracle.com/javase/6/docs/api/java/lang/Process.html). The docs note that,

>Because some native platforms only provide limited buffer size for standard input and output streams, failure to promptly write the input stream or read the output stream of the subprocess may cause the subprocess to block, and even deadlock.

processManager uses a timer to periodically drain the io streams. The interval of this timer is controlled by the property `pollInterval`. If you find your Matlab process blocking indefinitely, it may be that your process is particularly verbose or your buffers are particularly small, and you might try setting `pollInterval` to a lower value (default = 0.5 sec).

### Process fails to start
In rare instances, processes can fail to start with the following error:
```
Java exception occurred: 
java.io.IOException: Cannot run program "program" (in directory "dir"): error=24, Too many open files
	at java.lang.ProcessBuilder.processException(ProcessBuilder.java:478)
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:457)
	at java.lang.Runtime.exec(Runtime.java:593)
	at java.lang.Runtime.exec(Runtime.java:431)
Caused by: java.io.IOException: error=24, Too many open files
	at java.lang.UNIXProcess.forkAndExec(Native Method)
	at java.lang.UNIXProcess.<init>(UNIXProcess.java:53)
	at java.lang.ProcessImpl.start(ProcessImpl.java:91)
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:452)
	... 2 more
```
This results from the JVM exceeding the maximum number of file descriptors (for each process, one is opened for each stream STDIN, STDOUT and STDERR). Although streams are explicitly closed, it seems to take some time before resources are really released. Errors can occur in cases where many processManager instances are created and running simultaneously. I only ever see this when unit-testing and running lots of processes back-to-back, and it seems to occur on older releases (eg,2011b) and not newer releases (eg, 2012b). A simple solution is to throttle back the processes so that garbage collection can process the stream closures that are backing up. If there is really a need for many processes, the more useful solution is to increase the maximum files open allowed per process.
```
2011b
Java 1.6.0_65-b14-462-11M4609 with Apple Inc. Java HotSpot(TM) 64-Bit Server VM mixed mode
$ lsof -p 84020 | wc -l
     820

2012b
Java 1.6.0_65-b14-462-11M4609 with Apple Inc. Java HotSpot(TM) 64-Bit Server VM mixed mode
$ lsof -p 91747 | wc -l
     860

2013a
Java 1.6.0_65-b14-462-11M4609 with Apple Inc. Java HotSpot(TM) 64-Bit Server VM mixed mode
$ lsof -p 1508 | wc -l
     832
```
On OSX, you can see the number of allowable open files per process
```
$ launchctl limit
	maxfiles    256            unlimited

$ ulimit -a
open files                      (-n) 256
```

### Failure to cleanup timers
#### Timer still running when projectManager goes out of scope
_This issue is mostly fixed as of version 0.4.0. However, it is still possible for objects referencing processManager objects to go out of scope and leave timers, and the method below for clearing them is still valid. This is an issue with how Matlab determines when to [delete handle objects with timer objects](http://www.mathworks.fr/matlabcentral/answers/39858-clearing-handle-subclasses-with-timer-objects
)._

Because the [timers](http://www.mathworks.com/help/matlab/ref/timerclass.html) used to drain the io streams are [user-managed](http://blogs.mathworks.com/loren/2008/07/29/understanding-object-cleanup/), it is possible to get into a situation where they are not properly cleaned up. If you find yourself in this situation (usually heralded by Matlab refusing when you attempt `clear classes`), find the problem timers:
```
>> timers = timerfindall
```
and delete those that with the name including `processManager-pollTimer`. Eg.,
```
>> delete(timers)
```

To avoid orphaned timers, make sure that objects referencing processManager objects clean up properly (ie., they call the `stop()` method on processManager instances.

Useful info

[http://stackoverflow.com/questions/10489996/matlab-objects-not-clearing-when-timers-are-involved](http://stackoverflow.com/questions/10489996/matlab-objects-not-clearing-when-timers-are-involved)
[http://stackoverflow.com/questions/9559849/matlab-object-destructor-not-running-when-listeners-are-involved](http://stackoverflow.com/questions/9559849/matlab-object-destructor-not-running-when-listeners-are-involved)
[http://stackoverflow.com/questions/7236649/matlab-run-object-destructor-when-using-clear](http://stackoverflow.com/questions/7236649/matlab-run-object-destructor-when-using-clear)
[http://www.mathworks.com/matlabcentral/answers/39858-clearing-handle-subclasses-with-timer-objects](http://www.mathworks.com/matlabcentral/answers/39858-clearing-handle-subclasses-with-timer-objects)
[http://www.mathworks.com/matlabcentral/newsreader/view_thread/306641](http://www.mathworks.com/matlabcentral/newsreader/view_thread/306641)

#### Orphaned timers with handle arrays
You may eventually want to run many instances of the same process in parallel. An obvious approach would be,
```
>> p(1:10) = processManager('command','ping www.google.com');
```
**Do not do this**, it will put you into circular handle hell since all elements of `p` are references to one processManager instance. Instead, you need to call the constructor for each process,
```
>> for i = 1:10
>>     p(i) = processManager('command','ping www.google.com','autoStart',false);
>> end
>> p.start();
```

### Can't assign a property to object array
If you have an array of processManager objects and try to assign a single property to all of the objects, 
```
>> p(1) = processManager('id','google','command','ping www.google.com');
>> p(2) = processManager('id','yahoo','command','ping www.yahoo.com');
>> p.printStdout = false;
```
you will get the following error
```
Incorrect number of right hand side elements in dot name assignment.  Missing [] around left hand side is a likely cause.
```
Unfortunately, object property setters don't work with object arrays, so you need to do this:
```
>> [p.printStdout] = deal(false);
```
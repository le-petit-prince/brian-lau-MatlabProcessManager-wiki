## Matlab is blocked
processManager relies on the ability to call Java functions from within Matlab. In particular, it uses the Java [Runtime object](http://docs.oracle.com/javase/6/docs/api/java/lang/Runtime.html) to create [subprocesses](http://docs.oracle.com/javase/6/docs/api/java/lang/Process.html). The docs note that,

>Because some native platforms only provide limited buffer size for standard input and output streams, failure to promptly write the input stream or read the output stream of the subprocess may cause the subprocess to block, and even deadlock.

processManager uses a timer to periodically drain the io streams. The interval of this timer is controlled by the property `pollInterval`. If you find your Matlab process blocking indefinitely, it may be that your process is particularly verbose or your buffers are particularly small, and you might try setting `pollInterval` to a lower value (default = 0.5 sec).

## Failure to cleanup timers
### Timer still running when projectManager goes out of scope
Because the [timers](http://www.mathworks.com/help/matlab/ref/timerclass.html) used to drain the io streams are [user-managed](http://blogs.mathworks.com/loren/2008/07/29/understanding-object-cleanup/), it is possible to get into a situation where they are not properly cleaned up. This is due to the fact that the each timer holds a reference to the processManager object itself, which prevents the processManager destructor from getting called (a seemingly logical place to clean up the timers). For example, if a processManager object goes out of scope (say you clear it from the workspace while a process is still running), its timer(s) will continue to run even though you can no longer get to the processManager object. If you find yourself in this situation, find the problem timers:
```
>> timers = timerfindall
```
and delete those that with the name including `processManager-pollTimer`. Eg.,
```
>> delete(timers)
```

To avoid orphaned timers, always call `stop()` method before deleting a running process.

Useful info

[http://stackoverflow.com/questions/10489996/matlab-objects-not-clearing-when-timers-are-involved](http://stackoverflow.com/questions/10489996/matlab-objects-not-clearing-when-timers-are-involved)
[http://stackoverflow.com/questions/9559849/matlab-object-destructor-not-running-when-listeners-are-involved](http://stackoverflow.com/questions/9559849/matlab-object-destructor-not-running-when-listeners-are-involved)
[http://stackoverflow.com/questions/7236649/matlab-run-object-destructor-when-using-clear](http://stackoverflow.com/questions/7236649/matlab-run-object-destructor-when-using-clear)
[http://www.mathworks.com/matlabcentral/answers/39858-clearing-handle-subclasses-with-timer-objects](http://www.mathworks.com/matlabcentral/answers/39858-clearing-handle-subclasses-with-timer-objects)
[http://www.mathworks.com/matlabcentral/newsreader/view_thread/306641](http://www.mathworks.com/matlabcentral/newsreader/view_thread/306641)

### Orphaned timers with handle arrays
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

## Can't assign a property to object array
If you have an array of processManager objects and you try to assign a single property to all of the objects, 
```
>> p(1) = processManager('id','google','command','ping www.google.com');
>> p(2) = processManager('id','yahoo','command','ping www.yahoo.com');
>> p.printStdout = false;
```
will produce the following error
```
Incorrect number of right hand side elements in dot name assignment.  Missing [] around left hand side is a likely cause.
```
Unfortunately, object property setters don't work with object arrays, so you need to do this:
```
>> [p.printStdout] = deal(false);
```
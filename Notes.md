[timer callbacks with outputs using nested functions](http://compgroups.net/comp.soft-sys.matlab/what-happens-to-output-from-timer-callba/846063)

[fifo cell array for stdout and stderr?](http://www.weizmann.ac.il/home/eofek/matlab/General/q_fifo.m)

#Creating a more general Java class that handles threading for the streams, using runtime.exec
Polling is a terrible hack. Worse, it seems that when the process exits, the io streams are closed. Since polling is relatively infrequent, this means that io can be lost if the process exits in between polls. Really seems like I need to wrap up a Java class with threading to deal with the streams

[multithreading matlab with java](http://www.osmanoglu.org/computing/1-matlabjavamultitreading)
[processbuilder](http://thilosdevblog.wordpress.com/2011/11/21/proper-handling-of-the-processbuilder/)
[race conditions](http://www.coderanch.com/t/605311/threads/java/race-condition-Runtime-exec)
[nearly complete class](http://alvinalexander.com/java/java-exec-processbuilder-process-3)
[a streamgobbler](http://viralpatel.net/blogs/how-to-execute-command-prompt-command-view-output-java/)
[another almost complete class](http://stackoverflow.com/questions/2463475/design-for-a-wrapper-around-command-line-utilities)

[general runtime.exec notes](http://www.javaworld.com/article/2071275/core-java/when-runtime-exec---won-t.html)

# other libraries
[Plexus](http://ted-gao.blogspot.fr/2012/03/executing-external-programs-via-java.html)

# java in matlab
[creating classes](http://stackoverflow.com/questions/9520503/calling-java-from-matlab)
[envp](http://stackoverflow.com/questions/8607249/how-to-set-an-environment-variable-in-java-using-exec/8607281#8607281)
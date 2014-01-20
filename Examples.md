##Notification when a process finishes
processManager objects issue a notification when their processes exit. This is useful, for example, when something should be done when a process finishes. Below is a minimal example of listening for the exit notification (download [exampleListener.m](https://github.com/brian-lau/MatlabProcessManager/blob/master/Tests/exampleListener.m)):
```
if ispc
   cmd = 'ping -n5 www.google.com';
else
   cmd = 'ping -c5 www.google.com';
end

% Create an object and attach the listener
p = processManager('id','ping','command',cmd);
addlistener(p.state,'exit',@exitHandler);
```
where the listener callback is defined as:
```
function exitHandler(src,data)
   fprintf('\n');
   fprintf('Listener notified!\n');
   fprintf('Process %s exited with exitValue = %g\n',src.id,src.exitValue);
   fprintf('Event name %s\n',data.EventName);
   fprintf('\n');
end
```
Events are handled through the processState object, which each processManager instance instantiates.
More information can be found in the [Matlab documentation](http://www.mathworks.com/help/matlab/matlab_oop/learning-to-use-events-and-listeners.html).
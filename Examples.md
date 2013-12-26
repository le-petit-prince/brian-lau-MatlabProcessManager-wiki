##Notification when a process finishes
processManager objects issue a notification when their processes exit. This is useful, for example, when something should be done when a process finishes. Below is a minimal example of listening for the exit notification (download [exampleListener.m](https://github.com/brian-lau/MatlabProcessManager/blob/master/Tests/exampleListener.m)):
```
function exampleListener
   if ispc
      cmd = 'ping -n5 www.google.com';
   else
      cmd = 'ping -c5 www.google.com';
   end
   % Create an object and attach the listener
   p = processManager('id','ping','command',cmd);
   addlistener(p,'exit',@exitHandler);

   % Define the listener callback function
   function exitHandler(src,data)
      fprintf('\n');
      fprintf('Listener notified!\n');
      fprintf('Process %s exited with exitValue = %g\n',src.id,src.exitValue);
      fprintf('\n');
   end
end
```
More information can be found in the [Matlab documentation](http://www.mathworks.com/help/matlab/matlab_oop/learning-to-use-events-and-listeners.html).

```
% --- Executes on button press in pushbutton1.
function pushbutton1_Callback(hObject, eventdata, handles)
% hObject    handle to pushbutton1 (see GCBO)
% eventdata  reserved - to be defined in a future version of MATLAB
% handles    structure with handles and user data (see GUIDATA)
p = processManager('command','ping -c15 127.0.0.1','verbose',true);
handles.p = p;
guidata(handles.output,handles);
addlistener(p.state,'exit',@exitHandler);

% --- Executes on button press in pushbutton2.
function pushbutton2_Callback(hObject, eventdata, handles)
% hObject    handle to pushbutton2 (see GCBO)
% eventdata  reserved - to be defined in a future version of MATLAB
% handles    structure with handles and user data (see GUIDATA)
%temp=setappdata(handles.pushbutton1,'p',p);
handles.p.stop();
```
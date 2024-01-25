```json
{

"version": "0.2.0",

"configurations": [

{

"name": "gdb Debug",

"type": "cppdbg",

"request": "launch",

"program": "${workspaceFolder}/install/Debug/bin/onnxRuntimeExampleApp",

"args": [],

"stopAtEntry": false,

"cwd": "${workspaceFolder}",

"environment": [],

"externalConsole": false,

"MIMode": "gdb",

"setupCommands": [

{

"description": "Enable pretty-printing for gdb",

"text": "-enable-pretty-printing",

"ignoreFailures": true

}

],

"miDebuggerPath": "/usr/bin/gdb",

}

]

}
```

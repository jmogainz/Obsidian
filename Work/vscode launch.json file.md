```json
{

"version": "0.2.0",

"configurations": [

{

"name": "gdb Debug",

"type": "cppdbg",

"request": "launch",

"program": "${workspaceFolder}/deps/reinforcementlearningsimulator/install/bin/ServiceAllocatorExample",

"args": [],

"stopAtEntry": false,

"cwd": "${workspaceFolder}/deps/reinforcementlearningsimulator/install/bin/",

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

},

{

"name": "Python: Current File (Integrated Terminal)",

"type": "python",

"request": "launch",

"program": "${file}",

"console": "integratedTerminal",

"justMyCode": false

},

{

"name": "C++ Attach to Python Linux",

"type": "cppdbg",

"request": "attach",

"program": "/home/jmoore2/i2dev/remote/reinforcementlearning/venv/bin/python",

"processId": "${command:pickProcess}",

"MIMode": "gdb",

"miDebuggerPath": "/usr/bin/gdb",

"setupCommands": [

{

"description": "Enable pretty-printing for gdb",

"text": "-enable-pretty-printing",

"ignoreFailures": true

}

],

"additionalSOLibSearchPath": "/home/jmoore2/i2dev/remote/reinforcementlearning/install/lib/",

"logging": {

"moduleLoad": true,

"engineLogging": true

}

},

{

"name": "C++ Launch Python Windows",

"type": "cppvsdbg",

"request": "attach",

"processId": "${command:pickProcess}",

"additionalSOLibSearchPath": "${workspaceFolder}", // Adjust as needed for .dll location

"logging": {

"moduleLoad": false,

"engineLogging": false

}

}

]

}
```
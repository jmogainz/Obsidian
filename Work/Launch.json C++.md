{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": false
        },
        {
            "name": "Python: Current File with Arguments",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": false,
            "args": "${command:pickArgs}"
        },
        {
        "name": "gdb Debug",
        
        "type": "cppdbg",
        
        "request": "launch",
        
        "program": "/home/jmoore2/i2dev/remote/ai-reinforcementlearning/deps/rlsim/build/deps/UtilCpp/src/libs/datarecorder/tests/datarecorder_tests",
        
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
        
        },
        {
            "name": "Linux C++ Attach to Python",
            "type": "cppdbg",
            "request": "attach",
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "program": "/home/jmoore2/i2dev/remote/ai-reinforcementlearning/.venv/bin/python",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
              {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
              },
            ],
            "additionalSOLibSearchPath": "${workspaceFolder}/install/lib/",
        },
        {
            "name": "Windows C++ Attach to Python",
            "type": "cppvsdbg",
            "request": "attach",
            "processId": "${command:pickProcess}",
            "additionalSOLibSearchPath": "${workspaceFolder}/install/bin/",
            "logging": {
                "moduleLoad": false,
                "engineLogging": false
            }
        }
    ]
}
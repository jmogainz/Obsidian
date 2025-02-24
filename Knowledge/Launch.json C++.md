{
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
            "name": "Python: pytest",
            "type": "debugpy",
            "request": "launch",
            "module": "pytest",
            "args": [
                "--maxfail=3",
                "--disable-warnings",
                "--verbose",
                "${workspaceFolder}/"
            ],
            "justMyCode": false,
        },
        {
            "name": "gdb Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "Uninitialized",
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
            "program": "Uninitialized",
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
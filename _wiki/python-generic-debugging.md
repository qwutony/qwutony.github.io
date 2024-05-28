---
layout: wiki
title: Remote Debugging
cate1: Debugging
cate2: Python
description: Remote Debugging - Python
keywords: Debugging
---

# Remote Debugging for Python

## Prerequisites
1. **Install Visual Studio Code (VSCode)**
    - [Download VSCode](https://code.visualstudio.com/docs/?dv=linux64_deb)
    - Install VSCode:
        ```bash
        sudo apt install ~/Downloads/code_1.45.1-1589445302_amd64.deb
        ```
    - Install the "Python" extension in VSCode
    - Install Python debugging package:
        ```bash
        pip install ptvsd
        ```

## Configure Application for Debugging

1. **Edit the Main Application File**
    - Example configuration for `app.py`:
        ```python
        import ptvsd
        # By default, remote debugging starts on port 5678
        ptvsd.enable_attach(address=('0.0.0.0', 5678), redirect_output=True)
        print("Now ready for the IDE to connect to the debugger")
        ptvsd.wait_for_attach()
        ```

## Load the Code into Visual Studio Code

1. **Transfer Code to Local Machine**
    - Use Rsync via SSH:
        ```bash
        rsync -azP user@remote_host:/path/to/remote/code /path/to/local/code
        ```

## Configure Visual Studio Code to Connect to the Remote Debugger

1. **Start the Application with Debugging**
    - Ensure your application starts with the debugging configuration:
        ```bash
        python app.py
        ```

2. **Create `launch.json` in VSCode**
    - Open the VSCode debug view and click on the gear icon to create a `launch.json` file.
    - Select "Remote Attach".
    - Update the `launch.json` configuration:
        ```json
        {
            "version": "0.2.0",
            "configurations": [
                {
                    "name": "Python: Remote Attach",
                    "type": "python",
                    "request": "attach",
                    "connect": {
                        "host": "remote_host",
                        "port": 5678
                    },
                    "pathMappings": [
                        {
                            "localRoot": "${workspaceFolder}",
                            "remoteRoot": "/path/to/remote/code"
                        }
                    ]
                }
            ]
        }
        ```

## Generic Steps to Configure Remote Debugging for Any Python Application

1. **Install Necessary Tools and Extensions**
    - Ensure VSCode and the Python extension are installed.
    - Install `ptvsd` or `debugpy` in your Python environment.

2. **Modify Application Code for Debugging**
    - Add debugging configuration in your main application file (e.g., `main.py` or `app.py`):
        ```python
        import debugpy
        debugpy.listen(("0.0.0.0", 5678))
        print("Waiting for debugger attach")
        debugpy.wait_for_client()
        ```

3. **Transfer Code to Local Machine**
    - Use tools like Rsync, SFTP, or any file synchronization tool to transfer code from the remote server to the local machine.

4. **Configure VSCode for Remote Debugging**
    - Create or update `launch.json` with appropriate settings:
        ```json
        {
            "version": "0.2.0",
            "configurations": [
                {
                    "name": "Python: Remote Attach",
                    "type": "python",
                    "request": "attach",
                    "connect": {
                        "host": "remote_host",
                        "port": 5678
                    },
                    "pathMappings": [
                        {
                            "localRoot": "${workspaceFolder}",
                            "remoteRoot": "/path/to/remote/code"
                        }
                    ]
                }
            ]
        }
        ```

5. **Start Application with Debugging Enabled**
    - Run your Python application on the remote server with the debugging configuration:
        ```bash
        python app.py
        ```

6. **Attach Debugger in VSCode**
    - Open the debug view in VSCode and start the "Python: Remote Attach" configuration.

This guide provides a generic way to set up remote debugging for Python applications using VSCode.

# ddeploy

ddeploy is a deployment and provisioning automation tool targeting GNU/Linux systems designed to automate the deployment of complex testing environments.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [Step configuration](#step-configuration)
- [Modules](#modules)
- [Custom Configuration and modules](#custom-configuration-and-modules)
- [Examples](#examples)

## Installation
To install `ddeploy`, clone this repository to your desired location:

```sh
git clone https://github.com/dennisman219/ddeploy.git ~/.local/ddeploy
```

Then, add the `ddeploy` directory to your `PATH` environment variable:

```sh
export PATH=$PATH:~/.local/ddeploy
```

You can add the above line to your shell's startup file (e.g., `.bashrc`) to make the change permanent:

```sh
echo 'export PATH=$PATH:~/.local/ddeploy' >> ~/.bashrc
```

## Usage

To run the deployment script, use the following command:

```sh
deploy
```

by default the tool looks for a `deploy_config.yaml` configuration file in the current working directory, or in an optional `.deploy` directory in the current working directory.

### Command Line Options
- `-q`, `--quiet`: Quiet mode, suppresses all output.
- `-d`, `--debug`: Debug mode, enables verbose output.
- `-v`, `--verbose`: Verbose mode, enables verbose output.
- `-c <config_file>`, `--config <config_file>`: Specifies a custom configuration file.
- `-s <step_name>`, `--step <step_name>`: Runs a specific step exclusively.

## Step configuration

Each step to be executed is defined as a yaml item following the following format:
```yaml
<name>:
    module: <exampleModule>
    <param1>: <value>
    <param2>: <value>
    [result]: <value>
    [skip]: <value>
```

Each step in the deployment process can be configured with the following parameters:
- `name`: Name of the step to recognize it during deployment.
- `exampleModule`: The module to execute in this step.
- `param1`, `param2`, etc.: Parameters specific to the module being used.
- `result` (optional): Expected return code from the module execution.
- `skip` (optional): Boolean to indicate if the step should be skipped.

## Modules
Modules are scripts or executables that can be invoked by the deployment tool to perform specific tasks during the deployment process. Each module expects certain environment variables to be set, which are to be defined in the configuration file.

### Available Modules
- `copyToTarget`: Copies a file to a target location using scp.
    - expecting the following environment parameters:
    - `target`: the ssh hostname of target host
    - `file`: the file to be copied to the target
    - `target_location`: the location on the target machine to copy the file to
- `localCommand`: Executes a local shell command.
    - expecting the following environment parameters:
    - `command`: the shell command to execute locally
- `remoteScript`: Copies a bash script to a remote target and executes it.
    - expecting the following environment parameters:
    - `target`: the ssh hostname of target host
    - `script`: the local path to the script to execute 
- `sshCommand`: Executes a shell command on a remote target using ssh.
    - expecting the following environment parameters:
    - `target`: the ssh hostname of target host
    - `command`: the shell command the execute on the remote host
- `subdeploy`: Changes to a specified directory and runs the deploy script.
    - expecting the following environment parameters:
    - `working_directory`: the working directory to execute deploy in 
    - `deploy_config` (optional): the name of the configuration file to pass to deploy tool
- `waitForUserInput`: Waits for user input with a timeout.
    - expecting the following environment parameters:
    - `timeout`: how long to wait without user input before continuing 

## Custom Configuration and Modules

Users can create a `.deploy` folder in the project root directory where the deploy tool will also look for deployment configuration files. Users can place custom modules in the `.deploy/modules` folder. These custom modules can then be invoked by the deployment tool just like the built-in modules.

An example of how you can structure your project directory to include the `.deploy` folder and custom modules:

```
my-project/
├── .deploy/
│   ├── deploy_config.json
│   └── modules/
│       ├── customModule1.sh
│       └── customModule2.py
├── src/
│   ├── ...
└── ...

```

## Examples

To run the deployment with a custom configuration file:

```sh
deploy -c custom_config.yaml
```

To run a specific step exclusively:

```sh
deploy -s "local command"
```

Example `deploy_config.yaml`:

```yaml
wait:
    module: waitForUserInput
    timeout: 5
    result: 142

copy_config:
    module: copyToTarget
    target: "user@hostname"
    file: "path/to/file"
    result: -1

"command on local machine":                     # Key can be a string
    module: localCommand
    command: "echo 'Hello, world!'"
    skip: true

remote_script:
    module: remoteScript
    target: "user@hostname"
    script: "path/to/script.sh"

remote_command:
    module: sshCommand
    target: "user@hostname"
    command: "echo 'Hello, world!'"

run subdeploy:                                  # Key can contain spaces
    module: subdeploy
    working_directory: "path/to/subproject"
```
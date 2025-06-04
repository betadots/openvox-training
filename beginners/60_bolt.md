# Bolt

Bolt is an open-source task automation tool that allows you to remotely execute commands, scripts, and tasks on remote systems.

## Run Commands and Scripts

```
bolt help command run
bolt command run 'pwd' --targets 192.168.64.179 --native-ssh --user cloud
```

```
bolt script run ./script.sh --targets 192.168.64.179 --native-ssh --user cloud
```

## Starting a Bolt Project

```
./
|-- bolt-project.yaml
|-- inventory.yaml
|-- plans
|   |-- myplan.yaml
|   `-- myplan_vox.pp
|
`-- tasks
    |-- mytask.json
    `-- mytask.sh
```

`bolt-project.yaml` and `inventory.yaml` can be created by:

```console
bolt project init [project name]
```

* `bolt-poject.yaml`
 * project name
 * optional the required modules (for your plans maybe).
* `inventory.yaml`
  * target definition
  * target grouping
  * transport parameter

```yaml
---
groups:
  - name: training
    targets:
      - 192.168.56.100
      - name: student1
        uri: 192.168.64.179

config:
  transport: ssh
  ssh:
    native-ssh: true
    user: cloud
    host-key-check: false
    run-as: root
    run-as-command:
      - sudo
      - '-nksSEu'
```

**Note**: Overrides the options from the command line

```
bolt command run 'pwd' --targets student1
```

## Run Tasks

* similar to scripts, but they are kept in modules and can have metadata

```json
{
  "description": "some summary description",
  "parameters": {
    "version": {
      "description": "description of the parameter",
      "type": "Optional[String]"
    },
    ...
  },
  "implementations": [
    {
      "name": "mytask_shell.sh",
      "requirements": ["shell"],
      },
    {
      "name": "mytask_powershell.ps1",
      "requirements": ["powershell"]
    }
  ]
}
```

* any programming language the targets run

```console
ls ./tasks
mytask.json mytask_powershell.json mytask_powershell.sh 
mytask_shell.json mytask_shell.sh
```

* located in `./tasks` of the project or a module directory

```console
bolt task show
```

* show build-in  and additional installed tasks

```console
bolt task show
```

## Run Plans

* show build-in  and additional installed tasks

```console
bolt plan show
```

* run plan from built-in module fact

```console
bolt plan run facts::info --targets training
```

* install modules (from forge) into `./.modules/`

  * new project

  ```console
  bolt project init myproject --modules module1[,Module2[,...]]
  ```

  * existing project

  ```console
  bolt module add module1[,Module2[,...]]
  ```

  * using Puppetfile

  ```console
  bolt module install [--no-resolve]
  ```

* write plans in
  * `yaml`
  * Puppet DSL
  * or use for wrapping scripts

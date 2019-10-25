## Install and Run

To install, call:

```shell
npm install -g ton-dev-cli   
```

To run, call:

```shell
tondev command ...args
```

## Key Сommands

- `help`: displays the complete list of available commands;
- `setup`: installs all required TON Labs software and start services;
- `start`: starts Node SE and Compiler containers;
- `sol files` *[* -l js*]*: build the contract .tvc and .abi.json files from Solidity files 

> Optionally you can generate a JavaScript file with contract ABI and TVC encoded with base64

```shell
tondev setup 
tondev sol filename 
tondev sol filename -l js   
```

- `clean`: stops and removes all containers and images related to Node SE and its components;
- `info` gets the current Node SE state. The command shows:

1. the list of images and docker containers related to Node SE. 
2. current container state
3. list of versions available at docker hub. 
4. container settings
5. the version in use

- `use <version>`: allows switching between containers, e.g.: `tondev use 0.11.0`. By default` :latest` is used.
- `restart`restarts the containers;
- `recreate` used to recreate containers;
- When called without parameters, `tondev` is similar to the` info` command.

​      

<iframe class="no-border max-full-width full-height flex-grow" src="https://www.youtube.com/embed/Jsix7U0oZHI?autohide=1&amp;showinfo=0&amp;rel=0&amp;fs=0" style="max-width: 100%; height: 346px; -webkit-box-flex: 1; flex-grow: 1; border: none;"></iframe>

## Reference List

```html
Usage: tondev [options] [command]

TON Labs development tools

Options:
  -V, --version               output the version number
  -a, --available             show available versions
  -h, --help                  output usage information

Commands:
  info [options]              Show summary about dev environment
  setup [options]             Setup dev environment
  start [options]             Start dev containers
  stop [options]              Stop dev containers
  restart [options]           Restart dev containers
  recreate [options]          Recreate dev containers
  clean [options]             Remove docker containers and images related to TON Dev
  use [options] <version>     Use specified version for containers
  set [options] [network...]  Set network[s] options
  add [network...]            Add network[s]
  remove|rm [network...]      Remove network[s]
  sol [options] [files...]    Build solidity contract[s]

-------------- 
Commands help:
--------------


Command: info

 Usage: tondev info [options]

Show summary about dev environment

Options:
```



> Visit [TON Dev](https://ton.dev/) for additional product, company & community info.


  
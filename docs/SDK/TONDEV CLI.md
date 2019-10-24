# TONDEV CLI

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
  -a, --available  show available versions
  -h, --help       output usage information



Command: setup

 Usage: tondev setup [options]

Setup dev environment

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: start

 Usage: tondev start [options]

Start dev containers

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: stop

 Usage: tondev stop [options]

Stop dev containers

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: restart

 Usage: tondev restart [options]

Restart dev containers

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: recreate

 Usage: tondev recreate [options]

Recreate dev containers

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: clean

 Usage: tondev clean [options]

Remove docker containers and images related to TON Dev

Options:
  -n, --networks   clean compilers docker containers and images
  -m, --compilers  clean local node docker containers and images
  -h, --help       output usage information



Command: use

 Usage: tondev use [options] <version>

Use specified version for containers

Options:
  -n, --networks [names]  apply command to specified network[s] (names must be separated with comma)
  -m, --compilers         apply command to the compilers container
  -h, --help              output usage information



Command: set

 Usage: tondev set [options] [network...]

Set network[s] options

Options:
  -p, --port <port>        host port to bound local node
  -d, --db-port <binding>  host port to bound local nodes Arango DB ("bind" to use default Arango DB port, "unbind" to unbind Arango DB port)
  -n, --new-name <name>    set new name for network
  -h, --help               output usage information



Command: add

 Usage: tondev add [options] [network...]

Add network[s]

Options:
  -h, --help  output usage information



Command: remove

 Usage: tondev remove|rm [options] [network...]

Remove network[s]

Options:
  -h, --help  output usage information



Command: sol

 Usage: tondev sol [options] [files...]

Build solidity contract[s]

Options:
  -l, --client-languages <languages>  generate client code for languages: "js", "rs" (multiple languages must be separated with comma)
  -L, --client-level <client-level>   client code level: "run" to run only, "deploy" to run and deploy (includes an imageBase64 of binary contract)
  -h, --help                          output usage information
```



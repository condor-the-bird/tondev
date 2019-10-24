# Installation

## Preparation

### Prerequisites

- Install the latest version of [Docker](https://docs.docker.com/install/). See installation tips in the screenshots below:

![img](https://tonlabs.zeroheight.com/uploads/LXwZDEvN59LmO5AEg9PXjg.png)

![img](https://tonlabs.zeroheight.com/uploads/HlTD8VkiUFQIvrd7ucCeHw.png)

- For Linux users, make sure that you are able to run docker as non-root user (see <https://docs.docker.com/install/linux/linux-postinstall/>);

> **Tip**: check this page for more Docker installation options <https://phoenixnap.com/kb/how-to-install-docker-on-ubuntu-18-04>.

- Install [Node.js 10.x](https://www.digitalocean.com/community/tutorial_collections/38) or newer.

> Make sure that Docker daemon is running on your computer. To check its status, call `docker ps`. 

> Note that it is recommended to have at least 2Gb of RAM to use Node SE efficiently.

## Basic Installation

This is the easiest version. You can only call three commands to install the solution.

First, install TON Labs CLI by running:

```shell
npm install -g ton-dev-cli
```

>  **Important**: If you get errors related to permissions when trying to install packages globally, you can try to fix them using the following options: 

<https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally>. 

> Or call:

```shell
 sudo chown -R $(whoami) $(npm root -g)
```

> If you fail, run the command under `sudo`.

Then setup your machine. The step is optional, because `tondev start` and `tondev sol` perform the step on demand (see below).

```shell
tondev setup
```

### Run the Local Node

Call the following command to run the local node instance (Node SE):

```shell
tondev start
```

## Manual Installation

If you have troubles with `tondev` CLI utility you can install all required components yourself with the following commands:

> Note that Windows and Linux use different slashes (**/** vs. **\**) for system paths. Make sure to change them as needed in the paths to avoid errors. 

```shell
docker pull tonlabs/local-node
docker pull tonlabs/compilers

docker create -e USER_AGREEMENT=yes --name tonlabs-local-node -i -p80:80 tonlabs/local-node
mkdir -p <user home directory>/.tonlabs/compilers/projects
docker create -e USER_AGREEMENT=yes --name tonlabs-compilers -it --mount type=bind,dst=/projects,src=<user home directory>/.tonlabs/compilers/projects tonlabs/compilers

docker start tonlabs-local-node
docker start tonlabs-compilers

```

Home directory may have one of the following formats:

- Windows:` C:\Users\User1`
- MacOS:` '/Users/johnDough'`
- Linux: `/home/johnDough`

​      

<iframe class="no-border max-full-width full-height flex-grow" src="https://www.youtube.com/embed/S-WxdIL3vIA?autohide=1&amp;showinfo=0&amp;rel=0&amp;fs=0" style="max-width: 100%; height: 346px; -webkit-box-flex: 1; flex-grow: 1; border: none;"></iframe>

## Install Client Libraries

> **Tip**: go to the Getting Started section, to create your own environment and a test project from scratch according to detailed guidelines.

### Rust

1. Add a dependency into your cargo manifest:

```shell
[dependencies]
ton-client-rs = "0.11.1"
```

2. Call the following command:

```shell
cargo update
```

### Node.js

Call the following command to install Node.js client library (the recommended version comes first):

```shell
"dependencies": { "ton-client-node-js": "^0.12.1"}
```

Then execute (in the project folder):

```shell
~/ton-dev/hello$ npm install   
```

Or instead of steps 1 and 2 call:

```shell
npm install ton-client-node-js
```

### Web

Call the following command to install client library for web browsers:

```shell
npm install ton-client-web-js
```

### React Native

Call the following command to install client library for React Native:

```shell
npm install ton-client-react-native-js 
```

<iframe class="no-border max-full-width full-height flex-grow" src="https://www.youtube.com/embed/FMLTyQ2bYvE?autohide=1&amp;showinfo=0&amp;rel=0&amp;fs=0" style="max-width: 100%; height: 346px; -webkit-box-flex: 1; flex-grow: 1; border: none;"></iframe>

> Visit [TON Dev](https://ton.dev/) for additional product, company & community info.



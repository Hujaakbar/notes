# Distrobox

[Distrobox](https://github.com/89luca89/distrobox) is open-source software, used for running various linux distros as a guest distro within a host distro.
It is similar to VirtualBox that runs virtual machines, however with two key differences:

1. distrobox uses containers
1. distrobox does NOT prioritize the container's (guest distro's) isolation and sandboxing. Instead, it does the opposite, it tightly integrates the guest distro with the host distro.

> Simply put [distrobox] is a fancy wrapper around podman, docker, or lilipod to create and start containers which are highly integrated with the hosts.

Distrobox itself is not a container manager and relies on Podman or Docker to create containers.

**Distrobox is well-suited for software development and troubleshooting the host operating system, without having to install software on the host.**

Note: Distrobox is relatively new software; the fist public release was in 2021.

## Appeal of the Distrobox

The main appeal of the distrobox is two-fold:

1. It makes working with containers very easy
1. it makes the host's filesystem, network and other components available to the container. Configuring this set-up manually with docker/podman takes time and would probably be error-prone. Distrobox uses podman/docker under the hood and abstracts away these configurations.

## Use-Cases for Distrobox

1. Keeping the host distro clean by installing apps into the containers. When containers are deleted, all the installed apps will be removed too.

    - files created at home directory and its subdirectories are NOT removed.

    - Distrobox makes it very easy to enter the container's shell environment with single `distrobox enter` command.
    The apps installed inside the container is not accessible by the host.

        Example:

        Inside the container `rg` (ripgrep) program is installed.

        ```bash
        📦[user1@ubuntu ~]$ which rg
        /usr/bin/rg
        ```

        However, it is not accessible from the host.

        The host:

        ```bash
        user1@fedora:~$ which rg
        /usr/bin/which: no rg in (/home/user1/.local/bin:/home/user1/bin:/usr/local/bin:/usr/bin)
        ```

    - It is possible to make the apps installed inside the containers available to the host. So that, those apps can be used by the host.
    This process is called _exporting_.

2. Providing a mutable environment on an immutable OS, like ChromeOS, Fedora Atomic Desktops (e.g. Silverblue), etc

3. Providing a locally privileged environment for `sudo`less setups (e.g. company-provided laptops, security reasons, etc...)

4. Enabling running development-friendly distros like Arch linux within other distros such as Ubuntu LTS

## Security

> Isolation and sandboxing are not the main aims of the distrobox project, on the contrary it aims to tightly integrate the container with the host. The container will have complete access to your home, pen drive, and so on, so do not expect it to be highly sandboxed like a plain docker/podman container or a Flatpak.

## Tips

- for distrobox docs, use GitHub repo [docs](https://github.com/89luca89/distrobox/tree/main/docs/usage). The docs website is outdated
- for container management, use `podman` because it runs in rootless mode
- use Arch linux for development because it provides (latest versions of) vast library of software and libraries
- instead of using long commands, use [`distrobox assemble` command](#7-assemble) with manifest file
- to upgrade the containers' packages, use [`distrobox upgrade` command](#8-upgrade)
- Distrobox uses container managers like podman, docker and lilipod. Using `--additional-flags/-a` options, it is possible to pass options/commands to these container managers.
- create easily discard-able and reproducible containers
- use custom images that contains necessary libraries/tools

Note: NixOS is not a supported container distro, and there are currently no plans to bring support to it.

## Commands

Distrobox has 12 commands. I will explain most commonly used commands below.

### 1. create

`distrobox create` command create a container, ([docs](https://github.com/89luca89/distrobox/blob/main/docs/usage/distrobox-create.md)).

```bash
distrobox create --name <container-name> --image ubuntu-latest
```

Some of the options:

|option|description|
|---|---|
|`--name`/`-n`|name for the distrobox (default is my-distrobox)|
|`--image`/`-i`|image to use for the container (default is fedora workstation latest)|
|`--hostname`|hostname for the distrobox (default: /</container-name/>)|
|`--home`/`-H`|select a custom HOME directory for the container. Useful to avoid host's home littering with temp files.|
|`--clone`/`-c`|name of the distrobox container to use as base for a new container this will be useful to either rename an existing distrobox or have multiple copies of the same environment.|
|`--verbose`/`-v`|show more verbosity|

Cloning a container:

```bash
distrobox create --clone fedora-44 --name fedora-44-clone
```

### 2. list

`distrobox list` command lists the distrobox containers. `ls` is alias for `list`.

Note: it only lists distrobox created containers.

```bash
distrobox list
```

shorter version:

```bash
distrobox ls
```

### 3. enter

`distrobox enter` command enters the container making it available at the terminal.
If the container was not running, this command starts the container and enters it.

```bash
distrobox enter <container-name>
```

### 4. stop

`distrobox stop` command stops the running container.

stopping one container:

```bash
distrobox stop <container-name>
```

stopping multiple containers:

```bash
distrobox stop container-name1 container-name2
```

Some of the options:

```txt
--all/-a:        stop all distroboxes
```

stopping all containers:

```bash
distrobox stop --all
```

### 5. rm

`distrobox rm` command removes the container and all of its apps, and binaries.

```bash
distrobox rm <container-name>
```

Some of the options:

```txt
--all/-a:        delete all distroboxes

--force/-f:      force deletion

--rm-home:       remove the mounted home if it differs from the host user's one
```

### 6. ephemeral

`distrobox ephemeral` command creates and enters a temporary container that is destroyed once the container is exited.

```bash
distrobox ephemeral --image arch:latest
```

Above command is equivalent to running below commands individually:

1. creating a container with auto generated names like `distrobox-AbN5g6QJvE`:

    ```txt
    distrobox create --image arch:latest --name <auto-generated-name>
    ```

1. entering the container:

    ```txt
    distrobox enter <auto-generated-name>
    ```

Once the container is exited, distrobox removes the container along with its exported binaries.

Note: a container is not just stopped, but removed as well.

It is possible to provide `distrobox ephemeral` command an already created container. Even in this case, when container is exited, the container is removed.

### 7. assemble

`distrobox assemble` command allows creating and destroying containers as defined in a manifest file. Distrobox, by default, expects the manifest file to be named `distrobox.ini` and be in the current directory, but it can be specified using the `--file` flag as well.

To get the whole list of the available attributes for the manifest file, refer to the [official docs](https://github.com/89luca89/distrobox/blob/main/docs/usage/distrobox-assemble.md).

`distrobox.ini`:

```ini
[ubuntu]
image=ubuntu:latest
hostname="ubuntu-box"
entry=false # Generate an entry/link for the container in the host's app list
replace=false # replace already existing distroboxes with matching names
additional_packages="git bat tldr code"

[arch]
image=arch:latest
hostname="arch-box"
entry=false
replace=false
additional_packages="git bat tldr"
```

Some of the options available for manifest file.

- `include`: if containers share a lot of the configuration, instead of duplicating the configs, use `include` property and overwrite differing attributes.

    ```ini
    [ubuntu]
    image=ubuntu:latest
    additional_packages="git vim tmux nodejs"
    additional_packages="htop iftop iotop"
    additional_packages="zsh fish"
    nvidia=false

    [ubuntu-nvidia]
    include=ubuntu
    nvidia=true
    ```

- `additional_packages`: Additional packages to install inside the container.
   Note below points:

  - it runs every time the container starts.
  - it uses guest OS's default repository and some packages may not be available. The prime example is VS Code.

- `init_hooks`, `pre_init_hooks` and other options that runs commands/scripts are executed every time container starts.
- all the options that take boolean values have `false` as a default value

`distrobox assemble` command is used with either `create` or `rm` subcommands.

Some of the options:

```txt
--file:              path or URL to the distrobox manifest/ini file
--name/-n:           run against a single entry in the manifest/ini file
--replace/-R:        replace already existing distroboxes with matching names
--dry-run/-d:        only print the container manager command generated
--verbose/-v:        show more verbosity
```

creating all the containers defined in the manifest file:

```bash
distrobox assemble create
```

creating a specific container

```bash
distrobox assemble create arch
```

using url for manifest file path:

```bash
distrobox assemble create --file https://some-url.com/files/distrobox.ini
```

### 8. upgrade

`distrobox upgrade` command will enter the specified list of containers and will perform an upgrade using the container's package manager.

Some of the options:

```txt
--all/-a:            perform for all distroboxes
--running:           perform only for running distroboxes (requires --all)
--verbose/-v:        show more verbosity
```

Upgrade a container:

```bash
distrobox upgrade container-name
```

Upgrade all containers:

```bash
distrobox upgrade --all
```

Upgrade all running containers:

```bash
distrobox upgrade --all --running
```

## Config file for Distrobox

It is possible to configure default values for certain properties of the containers and dictate the behavior of the distrobox using config file.

`${HOME}/.config/distrobox/distrobox.conf`:

```ini
container_always_pull="1"
container_generate_entry=0
container_manager="podman"
container_image_default="registry.opensuse.org/opensuse/toolbox:latest"
container_name_default="my-container"
container_user_custom_home="$HOME/"
container_init_hook="~/.local/distrobox/a_custom_default_init_hook.sh"
container_pre_init_hook="~/a_custom_default_pre_init_hook.sh"
container_manager_additional_flags="--env-file /path/to/file --custom-flag"
container_additional_volumes="/example:/example1 /example2:/example3:ro"
non_interactive="1"
skip_workdir="0"
PATH="$PATH:/path/to/custom/podman"
```

## Version 2 of Distrobox

Currently Distrobox is being rewritten in go language. This new version will be v2.
It is not stable yet. Keep an eye on it.

## Alternatives

[toolbx](https://containertoolbx.org/) by fedora also allows running containers just like distrobox. Actually, distrobox gets its inspiration from toolbx. However, toolbox currently supports only four distros:

1. Arch Linux
1. Fedora
1. Red Hat Enterprise Linux >= 8.5
1. Ubuntu

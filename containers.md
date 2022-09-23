# Containers

***Concepts:***

- Control groups (abbreviated as cgroups): splits processes into groups to set limits on their consumption of compute resources—CPU, memory, disk, and network I/O.
- Namespaces restrict the ability of process groups from seeing or accessing system resources—PIDs, network interfaces, mount points, hostname, etc. Thus instituting a layer of isolation between process groups and the rest of the system.
- Seccomp and SELinux: The container technology employs these characteristics to run processes isolated in a highly secure environment with full control over what they can or cannot do
- Root vs. Rootless: Privileged container can perform admin functions and open ports below 1024. Rootless containers allow an unprivileged user to launch containers and interact with them.
- RH follows OCI (Open Container Initiative) to build images
- In RHEL 8 CRI-o is the container engine, and Red Hat offers three main tools to manage the containers: podman, skopeo, buildah

***Installation:***
`dnf module install containers-tools`

## Tools

### Podman

#### Image management

- `images`:  List local images
- `inspect` Examines an image and displays its details
- `login/logout` Logs in/out to/from a container registry. A login may required to access private and protected registries
- `pull` Downloads an image to local storage from a registry
- `rmi` Removes an image from local storage

#### Container management

- `attach` Attaches to a running container
- `exec` Runs a process in a running container
- `generate` Generates a systemd unit configuration file that caused to control the operational state of a container
- `info` Reveals system information, including the defined registries
- `inspect` Exhibits the configuration of a container
- `ps` Lists running containers (includes stopped contain the -a option)
- `rm` Removes a container
- `run` Launches a new container from an image -d (detached), -i (interactive), and -t (term)

### Skopeo

For interacting with local and remotes images and registries.

#### Configuration

- System-wide configuration file for image registries is `/etc/containers/registries.conf`
- Users configuration, if requierered, `~/.config/containers` (preference over system)

Sections:

- [registries.search](http://registries.search) lists the registries that are searched, first one more priority
- registries.insecure lists the registries that do not have valid SSL/TLS certificates
- registries.block lists registries that must not be used.

```bash
[registries.search]
registries = [‘registry.private.myorg.io’, ‘registry.redhat.io’, ‘quay.io’, ‘docker.io’]
```

#### Commands

- `inspect` inspect image local or remote
- `copy` copy an image
- `login/logout` to a container registry

## Exercises

1. Download and inspect an image from ([quay.io](http://quay.io/))

```bash
podman search quay.io/rhel8/mysql
quay.io/centos7/mysql-80-centos7
skopeo inspect docker://quay.io/centos7/mysql-80-centos7
podman pull docker://quay.io/centos7/mysql-80-centos7
podman images
podman inspect quay.io/centos7/mysql-80-centos7
```

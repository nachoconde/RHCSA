# Containers

***Concepts***

- Control groups (abbreviated as cgroups): split processes into groups to set limits on their consumption of compute resources—CPU, memory, disk, and network I/O.
- Namespaces restrict the ability of process groups from seeing or accessing system resources—PIDs, network interfaces, mount points, hostname, etc. Thus instituting a layer of isolation between process groups and the rest of the system.
- Seccomp and SELinux: The container technology employs these characteristics to run processes isolated in a highly secure environment with full control over what they can or cannot do
- Root vs. Rootless: Privileged container can perform admin functions and open ports below 1024. Rootless containers allow an unprivileged user to launch containers and interact with them.
- RH follows OCI (Open Container Initiative) to build images
- In RHEL 8, Red Hat has replaced Docker with a new solution: CRI-o is the container engine, and Red Hat offers three main tools to manage the containers: podman, skopeo buildah

***Installation***

`dnf module install containers-tools`

## Tools

### Podman

**Image management**

- `images` Lists downloaded images from local storage
- `inspect` Examines an image and displays its details
- `login/logout` Logs in/out to/from a container registry. A login may required to access private and protected registries
- `pull` Downloads an image to local storage from a registry
- `rmi` Removes an image from local storage
- `search` We can use filters: -filter option. Use podman search --filter is-official=true alpine. UBI stands for Universal Base Image, and it’s the image that is used as the foundation for all of the Red Hat products

**Container management**

- `attach` Attaches to a running container
- `exec` Runs a process in a running container. Example `podman exec -it web2 /bin/bash`
- `generate` Generates a systemd unit configuration file that caused to control the operational state of a container
- `info` Reveals system information, including the defined registries
- `inspect` Exhibits the configuration of a container
- `ps` Lists running containers (includes stopped contain the -a option)
- `rm` Removes a container
- `run` Launches a new container from an image -d (detached), -i (interactive), and -t (term)

**Podman information**

- `podman info`
- `podman version`

### Skopeo

- System-wide configuration file for image registries is `/etc/containers/registries.conf`
- Users configuration, if required, `~/.config/containers` (preference over system)

Conf file sections

- registries.search lists the registries that are searched, first one more priority
- registries.insecure lists the registries that do not have valid SSL/TLS certificates
- registries.block lists registries that must not be used.

```python
[registries.search]
registries = [‘registry.private.myorg.io’, ‘registry.redhat.io’, ‘quay.io’, ‘docker.io’]
```

****Commands****

- `inspect` inspect image local or remote
- `copy` copy an image
- `login/logout` to a container registry

## Containers Ports

- Rootless containers in podman run without a network address because a rootless container has insufficient privileges to allocate a network address → To make the service running in the container accessible from the outside, you need to configure port forwarding. This container can just have nonprivileged ports (not between 1–1024)
- To run a container that has an IP address and can bind to a privileged port, you need to run a root containing

    `podman run --name nginxport -d -p 8080:80`

    `sudo firewall-cmd --add-port 8080/tcp --permanent; sudo firewall-cmd -reload`

## Environment Variables

Passing host environment variables and setting new environment variables is done at the time of launching a container.

- `-e SECRET="secret134" -e HITSIZE`

## Persistent storage

To add persistent storage to Podman containers, you bind-mount a directory on the host operating system into the container. A bind-mount is a specific type of mount, where a directory is mounted instead of a block device.

To check:

- Permissions
- SELinux context: The file inherits the SELinux type from the parent directory.

`podman run -d --name mydbase -v /opt/dbfiles:/var/lib/mysql:Z -e MYSQL_USER=user -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=mydbase [registry.redhat.io/rhel8/mariadb103](http://registry.redhat.io/rhel8/mariadb103).`

Add :Z as shown to ensure the SELinux type container_file_t is automatically set on the directory and files within.

## Containers as Systemd Services

To enable a system service from a container:

- For root container
  - `sudo podman generate systemd  --new --name mycontainer | sudo tee /etc/systemd/system/root-container.service`
  - it will generate a service unit file called root-container.service under /etc/systemd/system
- For rootless container
  - create the directory ~/.config/systemd/user
  - `podman generate systemd  --new --name mycontainer >~/.config/systemd/user/container.service`
  - Update systemd to bring the new service to its control`systemctl —user daemon-reload`
  - Enable and start `systemctl --user enable --now container.service`
  - When systemctl --user is used, services can be automatically started only when a user session is started.  To define an exception:
    - **loginctl** session manager, which is part of the systemd solution to enable “linger” for a specific user account.
    - `loginctl enable-linger myuser`. When linger is enabled, systemd services that are enabled for that specific user will be started on system start, not only when the user is logging in.

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

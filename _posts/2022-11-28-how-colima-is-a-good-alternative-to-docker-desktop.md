Being a software engineer for some years, I have often used Docker Desktop in my career to manage containerized
applications. As Docker [updated their service agreements](https://www.docker.com/blog/updating-product-subscriptions)
August 31st, 2021, I needed to find an open-source or a free alternative to Docker Desktop, leading to Colima.

## What is Colima?

[Colima](https://github.com/abiosoft/colima) is a VM based on [Lima](https://github.com/lima-vm/lima) providing
container runtimes in MacOS/Linux.

Some of Colima's features are:

- a simple CLI;
- multi-profiles management;
- multi-architectures (Intel, M1);
- Docker and containerd runtimes;
- Kubernetes integration.

## Colima vs Docker Desktop

Time to get our hands dirty! Let's run some scripts to compare Colima and Docker Desktop performance.

For the purpose of the tests, we gonna compare I/O using volumes and (^) calculation inside the container.
Both VMs have the same configuration (2Gb memory, 2 CPUs, 60Gb disk).
Files management and CPU stress load will be triggered using the official `alpine` Docker image.

Please find the testing scripts on [Github](https://github.com/ths83/colima-blog).

### I/O

#### Using a [bind volume](https://docs.docker.com/storage/bind-mounts) (host)

![5c663858814f3faa0a246e1a0ce7a5ec.png](/assets/how-colima-is-a-good-alternative-to-docker-desktop/5c663858814f3faa0a246e1a0ce7a5ec.png)

| VM  | Create 200Mb file (MB/s) | Read 200Mb file (MB/s) | Copy 200Mb file (MB/s) | Create 10000 4Kb files (MB/s) | Delete folder (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Colima | 165.29 | 250 | 126.58 | 24.27 | 61.54 |
| Docker Desktop | 176.99 | 133.33 | 112.42 | 20.96 | 31.75 |

#### Using a [Docker volume](https://docs.docker.com/storage/volumes)

![cf88dfdcbf0ceff753f0660434c12b52.png](/assets/how-colima-is-a-good-alternative-to-docker-desktop/f5105030bff34509a5c5d3a9b5640e62.png)

| VM  | Create 200Mb file (MB/s) | Read 200Mb file (MB/s) | Copy 200Mb file (MB/s) | Create 10000 4Kb files (MB/s) | Delete folder (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Colima | 2222.22 | 6666.67 | 4000 | 2000 | 2500 |
| Docker Desktop | 748.31 | 2000 | 1538.46 | 1333.33 | 1052.63 |

#### Using the container

![3ea63e37beaa9e31b3f79343ab7cd893.png](/assets/how-colima-is-a-good-alternative-to-docker-desktop/3ea63e37beaa9e31b3f79343ab7cd893.png)

| VM  | Create 200Mb file (MB/s) | Read 200Mb file (MB/s) | Copy 200Mb file (MB/s) | Create 10000 4Kb files (MB/s) | Delete folder (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Colima | 2000 | 6666.67 | 4000 | 1538.46 | 2000 |
| Docker Desktop | 800 | 2000 | 1666.67 | 952.38 | 952.38 |

### CPU load

![79bf094c5816e3e55ad8ab8f86f2e2fb.png](/assets/how-colima-is-a-good-alternative-to-docker-desktop/79bf094c5816e3e55ad8ab8f86f2e2fb.png)

## How to install Colima?

Using Homebrew as package manager, run `brew install colima` to install it.

> Note: Other installation options are available [here](https://github.com/abiosoft/colima#installation).

For a smoother experience, add Colima [autocompletion](https://github.com/abiosoft/colima/blob/main/cmd/completion.go)
to your terminal.
Finally, run `colima start` to start a default instance.

```shell
$ colima start
INFO[0000] starting colima
INFO[0000] runtime: docker
INFO[0000] preparing network ...                         context=vm
INFO[0000] starting ...                                  context=vm
INFO[0022] provisioning ...                              context=docker
INFO[0022] starting ...                                  context=docker
INFO[0027] done
```

### Runtimes

Two container runtimes are supported by Colima: Docker (default) and containerd.

#### Docker

Docker is Colima's default runtime, meaning features from Docker are available out of the box.
`docker` and `docker-compose` CLIs are required to use Docker with Colima. `brew install docker docker-compose`

#### containerd

containerd is available by running `colima start -r containerd`.

> NOTE: containerd is not covered in this article. More details are
> available [here](https://github.com/abiosoft/colima/blob/main/README.md#containerd).

### Kubernetes

Colima includes a standalone [K3s](https://k3s.io/) server, so you can manage a Kubernetes cluster
using `colima start -k`.

There are other options available with the Kubernetes cluster:

- picking a specific runtime with the `-r` flag;
- deploying an ingress controller [Traefik](https://github.com/traefik/traefik) with the `--kubernetes-ingress` flag;
- selecting a Kubernetes version with the `--kubernetes-version` flag.

Last but not the least, `kubectl` is required to use Kubernetes by running `brew install kubectl`.

> Note: A future post will cover Kubernetes management with Colima. Stay tuned!

### Customizing the VM

The default `colima start` will set up an instance with 2 GB memory, 2 CPUs, 60 GB disk and a Docker runtime.

You can easily customize your instance by using flags.
As an example, `colima start -c 5 -m 4 -d 100` will start a 4 GB memory with 5 CPUs, an 100 GB disk and a Docker runtime
instance.
To get the full list of available options, run `colima start -h` .

```shell
$ colima start -h
Start Colima with the specified container runtime and optional kubernetes.

Colima can also be configured with a YAML file.
Run 'colima template' to set the default configurations or 'colima start --edit' to customize before startup.

Usage:
  colima start [profile] [flags]

Examples:
  colima start
  colima start --edit
  colima start --runtime containerd
  colima start --kubernetes
  colima start --runtime containerd --kubernetes
  colima start --cpu 4 --memory 8 --disk 100
  colima start --arch aarch64
  colima start --dns 1.1.1.1 --dns 8.8.8.8

Flags:
      --activate                    set as active Docker/Kubernetes context on startup (default true)
  -a, --arch string                 architecture (aarch64, x86_64) (default "aarch64")
  -c, --cpu int                     number of CPUs (default 2)
      --cpu-type string             the CPU type, options can be checked with 'qemu-system-aarch64 -cpu help'
  -d, --disk int                    disk size in GiB (default 60)
  -n, --dns ipSlice                 DNS servers for the VM (default [])
  -e, --edit                        edit the configuration file before starting
      --editor string               editor to use for edit e.g. vim, nano, code (default "$EDITOR" env var)
      --env stringToString          environment variables for the VM (default [])
  -h, --help                        help for start
  -k, --kubernetes                  start with Kubernetes
      --kubernetes-ingress          enable Traefik ingress controller
      --kubernetes-version string   must match a k3s version https://github.com/k3s-io/k3s/releases (default "v1.25.0+k3s1")
  -l, --layer                       enable Ubuntu container layer
  -m, --memory int                  memory in GiB (default 2)
  -V, --mount strings               directories to mount, suffix ':w' for writable
      --mount-type string           volume driver for the mount (sshfs, 9p) (default "sshfs")
      --network-address             assign reachable IP address to the VM
      --network-driver string       network driver to use (slirp, gvproxy) (default "gvproxy")
  -r, --runtime string              container runtime (docker, containerd) (default "docker")
  -s, --ssh-agent                   forward SSH agent to the VM

Global Flags:
  -p, --profile string   profile name, for multiple instances (default "default")
  -v, --verbose          enable verbose log
      --very-verbose     enable more verbose log
```

To stop your running instance, run `colima stop`.
To delete it, run `colima delete`. All associated items will be deleted (images, containers, volumes...).

#### Multi-profiles management

One of the most exciting Colima features is instances management. To do so, Colima is using profiles.

To create an instance with a profile name, add the `-p` flag to the command line.
For example, `colima start -p funny_profile` will create a default instance with `funny_profile` as profile name.
`colima start -p intel -a x86_64 -c 1` will start an intel instance with 1 CPU using the `intel` name.

You can retrieve your current profiles by running `colima list`.

```shell
PROFILE          STATUS     ARCH       CPUS    MEMORY    DISK     RUNTIME    ADDRESS
default          Stopped    aarch64    2       2GiB      60GiB
funny_profile    Running    aarch64    2       2GiB      60GiB    docker
intel            Running    x86_64     1       8GiB      60GiB    docker
```

> Note: Profile name must be unique. If the name is not given when starting an instance, `default` will be the default
> name.

To stop your instance, run `colima stop` with `-p <PROFILE>` *(&lt;PROFILE&gt; is your instance name).*
To delete a profile, run `colima delete` with `-p <PROFILE>`.

### Editing an instance

It is possible to customize an instance before starting it using flags, or by running the `start` command with the `-e`
flag.
Your terminal editor will open the instance configuration file to be edited. After modification, the instance will
start.

> Note: Once created, each configuration property is editable except the disk size.

#### Multi-architecture management

The second great feature of Colima is the CPU architecture emulation (i.e. aarch64, x86_64), available with the `-a`
flag.
Let's say you got an M1 laptop and the desired docker image does not have an aarch64 version.
By switching or creating a specific amd64 instance, you will be able to run the image.

```shell
# Default with Intel architecture
colima start -a x86_64

# Default with arm architecture
colima start -a aarch64
```

## Conclusion

Colima's performance is way better than *Docker Desktop***, especially when performing I/O operations and CPU load.
Moreover, features such as profile management and container runtime management are also a great addition to this
project.

**If you are looking for a Docker Desktop alternative on Mac, Colima seems to be the best player for container
management.**

#### Container applications alternatives

- [Rancher Desktop](https://rancherdesktop.io/) using the Lima VM and a GUI.
- [Finch](https://aws.amazon.com/blogs/opensource/introducing-finch-an-open-source-client-for-container-development/)

*Tested on an M1 Mac Book Pro with Colima v0.4.6, Docker Desktop v4.14.1.*

***Docker Desktop beta features can improve I/O operations by enabling VirtioFS and the new Virtualization framework,
but Colima still have the best I/O performance.*

> Post also available on [Kumojin's website](https://kumojin.com/en/colima-alternative-docker-desktop)
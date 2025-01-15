`dockerd` can be thought of as the "backend" for Docker. It receives API requests from the users via commands  in the terminal (or RESTful requests) and processes them to the `containerd` runtime. `dockerd` communicate to `containerd` via the gRPC call. 

The `containerd` runtime is the process that actually creates the containers and manages their lifecycle. It's been decoupled from the `dockerd` server to allow programs to use them exclusively if needed. This is what Kubernetes does. `runc` is the default lower-level runtime by `containerd`. `dockerd` communicates important information to `containerd` like the image layers, entry point, cpu limits and more, so that it can create the environment. 

The **OCI Runtime Specification** is one of the objects sent to `containerd` which holds a bunch of metadata for how the container should be set up. It's a standard that `runc` is compliant to. The `config.json` is a file that holds a lot of that info.  

`runc` is the process that actually sets up the linux namespaces (mount, pid, network, cgroups, etc) based on the specs it receives.  

>![[Pasted image 20241027120415.png]]

### Container Networking
*Bridge Mode* is the default network behavior. All containers within the docker environment can communicate via their _internal IP_. This IP does not exist from the outside, so you wouldn't be able to ping it from your machine. You can ping/communicate with the container depending on what **port** you assigned to it from outside in. One container per host port! They can be the same port on the inside though.

**Example**:

```bash
docker run -p 8080:80 foo-bar:latest
```

This means that `8080` is talking to `foo-bar` container, so I can't use that same port for a different container.

The virtual bridge interface called `docker0` acts as a proxy / middle-man for traffic in and out of the Docker environment. Traffic going out is NAT'ed to the hosts IP. This means that when a container communicates with an external API or the internet, it uses the hosts IP. 
Traffic coming in is forwarded to the correct container based on port mappings. 

### Internals Exploration
Using the following command, you can exit out of the default mounted filesystem and explore behind the veil.

```bash
> docker run --rm -it --privileged --pid=host debian \
	nsenter -t 1 -m -u -n -i sh

> cd /var/lib/docker/overlay2/ 
```

This directory contains all of the filesystem layers that make up a container.

```bash
cd /var/lib/docker/containers/<container-id>/
```

In here you will find all the configurations for the docker container.

```bash
docker inspect <container-id>
```

Will give you configuration information, including the `GraphDriver` which shows you how the directories are "stacked". Notice how the `LowerDir` contains multiple filesystems separated by a colon. The leftmost one is the bottom layer (with suffix `-init`), so layers get stacked from left to right.

`/l` contains shortened layer ids that are symbolic links to the actual image layers.

The lowest layer contains a `link` file which contains the shortened id and a directory called `/diff` which has it's contents.

The second-lowest layer and going up contain more than that:
- `lower`: file that points to its parent.
- `/diff`: contains the contents of that layer.
- `/merged`: contains the unified contents of its parent layer and itself.
- `/work`: used internally by OverlayFS

```bash
find . -type d -name "merged" # Find subdirs call /merged
```

Most of these dirs are empty because we haven't actually mounted them yet!

```bash
/var/lib/docker/overlay2 # mount | grep overlay

overlay on /containers/services/01-docker/rootfs type overlay (rw,relatime,lowerdir=/containers/services/01-docker/lower,upperdir=/containers/services/01-docker/tmp/upper,workdir=/containers/services/01-docker/tmp/work,uuid=on)
```

The `mount` command shows us the mounted overlay filesystems in use.


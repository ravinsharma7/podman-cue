# podman-cue
Cue wrapper around podman functionality. Its a streamlined cue wrapper that works as a sane glue that connects container services in a unifed way. Minimal learning and concept to use it quickly.

# Why
Currently to work with podman containers you can do the following:  
1. running podman command and piece them up using bash scripts.
2. use `podman kube play`
3. use quadlet command to generate systemd service units to integrate with systemd
4. use ansible + "ansible podman collection"
5. use podman's docker compatibility and use docker compatibile tools.
 
### `podman` command + bash script
- The OG way of GTD.
- Has all the typical bash scripting issues, no consistency in terms of scripting for different services.
- Can be error prone and messy when loading data for working with services in container.
- no build-in linting and validation layer. Unknown error will cause you to loose your mind.

### `podman kube play`
- A subset of k8s yaml manifest.
- Doesn't support all k8s options.
- Assumes you have k8s manifest knowledge.
- Docs can be confusing and convoluted because it requires you to refer k8s docs + `podman kube` docs.
- You need to manually check the parity between `podman kube play` and `kubectl` way of reading the manifest files and test whether they behave somewhat same.
- no build-in linting and validation layer, you can only know when running the `podman kube play` itself or when accessing the services. There are some github project that do k8s manifest validation, but is not `podman kube` aware.
  
### `podman quadlet` command
- Very easy to generate service units for systemd that works with podman containers
- Sometimes can be confusing because the docs doesn't show word by word diff between a systemd service unit file and a quadlet service unit.
- no build-in linting and validation layer. `systemd-analyze` doesn't work with podman specific fields and value. There is a `dry-run` flag for podman if not mistaken, but I have not used much of it to know if there is parity issues.

### `ansible` + "ansible podman collection"
- you can see it here: https://docs.ansible.com/ansible/latest/collections/containers/podman/index.html#plugins-in-containers-podman
- it is pretty extensive coverage of podman command
- I and not sure of the actual parity between collection with podman version. I'm not sure if the command and forward compatibile or backward compatible.
- ansible error will get mixed with podman container error adding a bit more confusion.
- no build-in linting and validation layer. You could use Molecule to do dry-run but I don't know how reliable is those tests will be across version and production environment. Plus Molecule is opiniated and has its own ways of doing things.

### docker compatibility
- There is nothing much to say on this.
- Docker will work well with tools that specifically support it. But podman's compatibility with docker can a "hit and miss".
- If everything works then lucky you. If something breaks because of parity difference you will get into a world of pain.

# Direction of the project
- A unified way to load services and turn them on, down them, and do specific deployment strategy like rolling updates and pre/post checks and healthcheck.
- Easily express depedencies between individual containers and groups of containers.
- Integration with podman and systemd, and seemless migration towards k8s for specific containers. Cue has k8s module and can export toml, json, json scheme, yaml, etc, so this is most likely possible.
- Using the same unified way to do horizontal scaling for specific services, or multiple vps, by integrating with tools like ansible and opentofu.
- Use it without being tied to k8s for small to medium scale services or work with specific k8s hosted services without migrating everything.
- Simple way to use yaml, json, etc. without worrying about compatibility between the formats.
- Buildin validation, verification, pre/post checks of commands. Feature aware verification; port checking, active network, dns availability, file/folder/volume check, etc.
- Accurately assist to pin point errors that are host related, vps related, or related to container or podman.
- UI Panel to do things without command line.

### Example creating multiple services with podman containers
- syntax using cue, but only need to know very minimal to use it.

```cue
package service

import (
  os "tool/os"
  ctr "ravinsharma.com/podman/lib/container"
)

env: os.Getenv & {
  "HOME":    *""   | string  // empty string fallback
  "BASEDIR": *"./" | string  // default “./” if not set
  "PATH":    *""   | string
}

containers: {
  caddy: ctr.#ContainerSpec & {
    image: "localhost/runtime/caddy"
    tag:   "2.8.4"
    name: "qs_caddy"
    description: """
    The http server for application with the frontend
    """
    ports: [
      "80:8080",
      "8080:8080",
    ]
    volumes: [
      ["\(env.BASEDIR)/services/caddy/conf/Caddyfile", "/etc/caddy/Caddyfile", "Z"],
    ],
    env: {
      name: "prod"
      vars: {
        CADDY_ENV: "production"
      }
    }
  },
  {
    postgres: ctr.#ContainerSpec & {
      image: "docker.io/library/postgres"
      tag:   "16-alpine"
      name: "qs_postgres"
      ports: ["5432:5432/tcp"]
      volumes: [
        ["pgdata", "/var/lib/postgresql/data", "Z"]
      ]
      flags: ["--health-cmd=pg_isready", "--health-interval=10s"]
      env: {
        POSTGRES_USER:     "pg_user",
        POSTGRES_PASSWORD: "secret",
      }
    }
  },
//  "golang_app": ctr.#ContainerSpec & {
//     ...
//	}
}
```

# Why not just docker + docker compose + docker swarm

I'm invested in podman tooling and am balls deep into it. Docker is cool as always, but lacks some flexibility I get from podman, like when using with systemd and k8s.

# Goal and Non-goals

This project does not intent to be a replacement of `docker-compose` or `podman-compose`, but only integrates with the process.  
Integration with existing tooling is always a goal of the project.  
It will be much closer as portainer alternative but with much better flexibility and more extensibility.  
Also it should be usable without knowing a whole bunch of concept to use it, everything you know about podman container should be enough to get started.  

# How to use

TBD.   
Still very early stage, things will break and I'm trying to flesh out some idea and workflow based on my own usage. Still figuring out cuelang itself. One problem I face in cuelang right now are error messages, customizing it is not the easiest, I can use cue as a library in go if I really need to customize further.  
If you want to learn more about cuelang, here is the site:  https://cuelang.org/  

Two downstream projects are created to support the development of this project.
[cuerun](https://github.com/ravinsharma7/cuerun)
[cuesplit](https://github.com/ravinsharma7/cuesplit/tree/main)

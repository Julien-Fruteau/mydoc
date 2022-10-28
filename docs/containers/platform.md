# Platform

## Host Architectures

- Docker containers aims to isolate an os running instance from your host os.
- It is widely used to develop applications that will be deployed on cloud infrastructure on linux os.
- One may think that developing on your localhost personal computer shall reproduce consistent behaviour on the image being deployed on the cloud. *There is a one important thing to consider* :

> *The architecture of the host running the container*

If the host architecture [arch] is `amd64` (amd, intel processor), or `arm64` (raspberrypi, mac m1/m2) the underlying docker image used will be dependent of the host architecture you pull the image from.

> **Consequences** : you may experience docker image that cannot be built from a host, but possibly from another one for reasons such as `linux package and dependencies` being available for one arch, and not for the other.

### How to mitigate

- Consider building your image from a CICD runner with the same architecture of the end goal target architecture
- Develop directly on a cloud dev environment

## Image Manifest & Architecture

- On which **platform (architecture + os)** an image from a docker registry is available on :

```bash
# example from docker hub registry:
docker manifest inspect nginx:latest
```

- returns :

```json linenums="1" hl_lines="9-11 18-21 28-31 38-41 48-50 57-59 66-68 75-77"
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:06aa2038b42f1502b59b3a862b1f5980d3478063028d8e968f0810b9b0502380",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:215352c0d3c37d4b7dd4e54b333755a6c7f85cde0e6286632c0daa5f92dc3f14",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v5"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:ee28c54380d836869a4c6bbe1a825e649174fedc4c2bc281a6d3384bbd0fac4f",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:5b35ed9fcfadf3750fb0e2b6a88913c5a7f84d8d1804c7c6fd6b79e35c071e8b",
         "platform": {
            "architecture": "arm64",
            "os": "linux",
            "variant": "v8"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:f63318b7ccc12e1fce241fdbf65c16ccee4c2e5bb48ac6ed468222c3590de6cf",
         "platform": {
            "architecture": "386",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:36b5efd39313d987f700a1996267d47fea871f872b3cea13e4cac9a4dc02d017",
         "platform": {
            "architecture": "mips64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:82b830dbef0567a4a5f5ba2e47ee8eab34485af6921310060aad72c0a34dd1f0",
         "platform": {
            "architecture": "ppc64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1570,
         "digest": "sha256:e13e070adae8f92e90fc9975aec3946f4a8bff3ee048c108b9fcce18720c9d84",
         "platform": {
            "architecture": "s390x",
            "os": "linux"
         }
      }
   ]
}
```

## Pull image & Architecture

> When pulling an image, docker will pull the image corresponding to the platform the command is run on

- example on an intel/amd host architecture (amd64) :

```bash
docker pull nginx:latest
docker inspect nginx:latest
```

> **Line 87** indicates the image pulled is for architecture **"Architecture": "amd64"**

```json linenums="1" hl_lines="87"
[
    {
        "Id": "sha256:76c69feac34e85768b284f84416c3546b240e8cb4f68acbbe5ad261a8b36f39f",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "",
        "Created": "2022-10-25T10:23:08.612298023Z",
        "Container": "783c85ff89f13f3b8df2f85ff9a8c0967600272c4da75f18023c75713d64785f",
        "ContainerConfig": {
            "Hostname": "783c85ff89f1",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.23.2",
                "NJS_VERSION=0.7.7",
                "PKG_RELEASE=1~bullseye"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
            ],
            "Image": "sha256:81d85638579047abdda2c2274af5961cb0e3c96c575171d0ff698f68f13e7b00",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "DockerVersion": "20.10.12",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.23.2",
                "NJS_VERSION=0.7.7",
                "PKG_RELEASE=1~bullseye"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "sha256:81d85638579047abdda2c2274af5961cb0e3c96c575171d0ff698f68f13e7b00",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 141778201,
        "VirtualSize": 141778201,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/87557c711a0f2ccc048d343d830538047af6ba74b3bd1affbb803f4073766759/diff:/var/lib/docker/overlay2/5b060b75091d0f2c75b06ebe6a8579653ae252b2298d330a42032cf8e793feaa/diff:/var/lib/docker/overlay2/903ddff588bee8f67832558c597edc5c68c190cf0cab596b79b2560bb67dd36e/diff:/var/lib/docker/overlay2/889db9695e0b6e5b96d70b6d6af5d1b592a4fcc6070f5c5254b4815be5d087ca/diff:/var/lib/docker/overlay2/9af163b7a8f4150f2ecb7b862464d6d68ac8e8a3259c1a4b065f55fbfa2a5b3b/diff",
                "MergedDir": "/var/lib/docker/overlay2/02aab637ed5c8fd748dea5b256bb68c224f1029e663516576391d80779f11055/merged",
                "UpperDir": "/var/lib/docker/overlay2/02aab637ed5c8fd748dea5b256bb68c224f1029e663516576391d80779f11055/diff",
                "WorkDir": "/var/lib/docker/overlay2/02aab637ed5c8fd748dea5b256bb68c224f1029e663516576391d80779f11055/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:a12586ed027fafddcddcc63b31671f406c25e43342479fc92a330e7e30d65f2e",
                "sha256:e74d0d8d2defd5fff2f34af104d18e2512941fd9a6abb0581a6abcc95d7e90ee",
                "sha256:2280b348f4d6af723032eec5a0c05f07222d6d10eafb5687eb2e86ca69de04fd",
                "sha256:9e7119c28877f445e5893da11829e0aaa4e5b8112bf1521aebc4fd40219ddbae",
                "sha256:4091cd312f19d65a309dd0962d374daf40f3f14b8c9e11538a6f250819c72801",
                "sha256:a2e59a79fae0d350555b7143026eb0a6a55e31b0de877f6b202d5bde77b1e863"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

> **NB** : it is possible to pull an image for another platorm but experience so far indicates that even it you can build the image, you may face unexpected behaviour running you application, and this not advised.

- Example of command to pull image for a specific platorm :

```bash
docker pull --platform=linux/arm64 nginx:latest
```

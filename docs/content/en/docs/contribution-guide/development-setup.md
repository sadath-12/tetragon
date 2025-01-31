---
title: "Development setup"
weight: 1
description: "This will help you getting started with your development setup to build Tetragon"
---

## Building and running Tetragon

For local development, you will likely want to build and run bare-metal Tetragon.

### Requirements

- A Go toolchain with the [version specified in the main `go.mod`](https://github.com/cilium/tetragon/blob/main/go.mod#L4);
- GNU make;
- A running Docker service (you can use Podman as well);
- For building tests, `libcap` and `libelf` (in Debian systems, e.g., install
  `libelf-dev` and `libcap-dev`).

### Build everything

You can build most Tetragon targets as follows (this can take time as it builds
all the targets needed for testing, see [minimal build](#minimal-build)):

```shell
make
```

If you want to use `podman` instead of `docker`, you can do the following (assuming you
need to use `sudo` with `podman`):

```shell
CONTAINER_ENGINE='sudo podman' make
```
You can ignore `/bin/sh: docker: command not found` in the output.

To build using the local clang, you can use:
```shell
CONTAINER_ENGINE='sudo podman' LOCAL_CLANG=1 LOCAL_CLANG_FORMAT=1 make
```

See [Dockerfile.clang](https://github.com/cilium/tetragon/blob/main/Dockerfile.clang)
for the minimal required version of `clang`.

### Minimal build

To build the `tetragon` binary, the BPF programs and the `tetra` CLI binary you
can use:
```shell
make tetragon tetragon-bpf tetra
```

### Run Tetragon

You should now have a `./tetragon` binary, which can be run as follows:

```shell
sudo ./tetragon --bpf-lib bpf/objs
```

Notes:

1. The `--bpf-lib` flag tells Tetragon where to look for its compiled BPF
   programs (which were built in the `make` step above).

2. If Tetragon fails with an error `"BTF discovery: candidate btf file does not
   exist"`, then make sure that your kernel support [BTF](#btf-requirement),
   otherwise place a BTF file where Tetragon can read it and specify its path
   with the `--btf` flag. See more about that
   [in the FAQ]({{< ref "/docs/faq/#tetragon-failed-to-start-complaining-about-a-missing-btf-file" >}}).

## Running code generation

Tetragon uses code generation based on protoc to generate large amounts of
boilerplate code based on our protobuf API. We similarly use automatic
generation to maintain our k8s CRDs. Whenever you make changes to these files,
you will be required to re-run code generation before your PR can be accepted.

To run codegen from protoc, run the following command from the root of the
repository:
```shell
make codegen
```

And to run k8s CRD generation, run the following command from the root of the repository:
```shell
make generate
```

Finally, should you wish to modify any of the resulting codegen files (ending
in` .pb.go`), do not modify them directly. Instead, you can edit the files in
`cmd/protoc-gen-go-tetragon` and then re-run `make codegen`.

## Running vendor

Tetragon uses multiple modules to separate the main module, from `api` from
`pkg/k8s`. Depending on your changes you might need to vendor those changes,
you can use:

```shell
make vendor
```

Note that the `make codegen` and `make generate` commands already vendor
changes automatically.

## Building and running a Docker image

The base kernel should support [BTF](https://github.com/cilium/tetragon#btf-requirement)
or a BTF file should be bind mounted on top of `/var/lib/tetragon/btf` inside
container.

To build Tetragon image:
```shell
make image
```

To run the image:
```shell
docker run --name tetragon \
   --rm -it -d --pid=host \
   --cgroupns=host --privileged \
   -v /sys/kernel/btf/vmlinux:/var/lib/tetragon/btf \
   cilium/tetragon:latest \
   bash -c "/usr/bin/tetragon"
```

Run the `tetra` binary to get Tetragon events:
```shell
docker exec -it tetragon \
   bash -c "/usr/bin/tetra getevents -o compact"
```

## Building and running as a systemd service

To build Tetragon tarball:
```shell
make tarball
```

## Running Tetragon in kind

This command will setup tetragon, kind cluster and install tetragon in it. Ensure docker, kind, kubectl, and helm are installed.

```shell
# Setup tetragon on kind
make kind-setup
```

Verify that Tetragon is installed by running:
```shell
kubectl get pods -n kube-system
```

## Local Development in Vagrant Box

If you are on an intel Mac, use Vagrant to create a dev VM:

```shell
vagrant up
vagrant ssh
make
```

If you are getting an error, you can try to run `sudo launchctl load
/Library/LaunchDaemons/org.virtualbox.startup.plist` (from [a Stackoverflow
answer](https://stackoverflow.com/questions/18149546/macos-vagrant-up-failed-dev-vboxnetctl-no-such-file-or-directory)).

## What's next

- See how to [make your first changes](/docs/contribution-guide/making-changes).


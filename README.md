# concourse and gpgpu

As part of the [spearow/juice](https://github.com/spearow/juice) efforts, it became necessary to have
cuda™/cudnn access and eventually also rocm and OpenCL™ support from within the container without granting
excessive privileges that would allow to remount the device tree.

All instructions here are for [`Fedora 32` / `Fedora 33`](https://getfedora.org). For Ubuntu-specific instructions, see ubuntu/README.md

Assumes concourse is unpacked under `/usr/local`, such that `/usr/local/concourse/bin/{gdn,concourse}` exist.

Note that unlike kubernetes / nomad (iirc), concourse has no means of scheduling something on GPUs.

Make sure to set the `serial: true` in your resources and apply a shared serial group.

All filesystems are formatted as `btrfs` with `compress=zstd:3`.

There is a test pipeline defined in `councourse-test.yml`.

## Dead Ends

The first attempt was to use `garden`/`gdn` directly to use the `nvidia-container` runtime, and also
manually specify the additional rules for `runc` (which is all that `nvidia-container` does anyways), but
garden does not care about the hooks and does not pass them on to the runc container launch (it's a bit more complex, but roughly like this, from memory).

**Solution**: Use `containerd` as another intermediate layer step, use the `nvidia-container-hook` with a custom hook for the default `runc`.

## nvidia

### Installation

Copy the whole tree under `/etc` to your os, make sure the permissions are sane.

```sh
# note that it is intentional to use rhel / centos 8
# the fedora repos sometimes lag behind a couple or releases(!)
dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.reposudo
dnf clean all
# do not install the nvidia driver from here!
# Use the kernel module from https://negativo17.org/nvidia-driver/
# which describes the compat with the upstream cuda https://github.com/negativo17/compat-nvidia-repo/issues/1
# eventually one can use all packages from negativio17 for convenience, but right now there are multiple issues
dnf config-manager --add-repo=https://negativo17.org/repos/fedora-nvidia.repo
dnf install nvidia
dnf -y install cuda-11-1

dnf install -y \
    nvidia-container-runtime \
    nvidia-container-runtime-hooks \
    containerd
```

The provided `nvidia-tools.json` is not great, it allows access for all
containers, such that given you use it for a CI that validates your PRs,
could cause a DoS on your GPU resources.
See [man oci-hooks 5](https://github.com/containers/podman/blob/master/pkg/hooks/docs/oci-hooks.5.md) for more details
on how to specify proper `annotations`.

### Caveats

The container and nvidia libraries in the container must match up, or you will get errors regarding failure to
communicate with the driver.

This might be aleviated by mapping the host nvidia libs into the container, to date I did not find a solution
to do this. Please file a pull request if you figured this out!

## AMD

WIP

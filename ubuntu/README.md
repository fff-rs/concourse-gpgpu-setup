# Ubuntu 20.04

The following describes a variation to the Fedora-style configuration described at ../README.md, and explains how I setup Concourse worker with GPU acceleration using Ubuntu 20.04

## nvidia drivers

Install the latest nvidia drivers recommended by `sudo ubuntu-drivers devices`, or simply run `sudo ubuntu-drivers autoinstall` if you're feeling lucky

## nvidia runtime

Per the [docs](https://nvidia.github.io/nvidia-container-runtime/), install the [nvidia-runtime](https://github.com/NVIDIA/nvidia-container-runtime):

```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
sudo apt-get install nvidia-container-runtime
```

## config

Copy the contents of the `etc` folder to `/etc`, and customize to suit. The primary distinction to the fedora instructions is that containerd running on Ubuntu seems to ignore the `/etc/containers` directory, so it's necessary to specify the nvidia runtime in `/etc/containerd/config.toml`.
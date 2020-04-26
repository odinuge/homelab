# homelab

> Odin's homelab setup

![homelab](/media/homelab_1.jpg)

## Goals

The primary goals of this project are...

- to have a highly-available cluster for home autmation & testing
- to have no SPOFs (Single Point of Failure)
- to have fun (most important)

## Hardware

- 3 x [Raspberry Pi 4 B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
  - 4GiB RAM
  - 32GiB mmcblk + 128GiB ssd storage (for os)
  - Ubuntu 20.04 'Focal Fossa' (aarch64)
- 3 x [Tinker Board S](https://www.asus.com/no/Single-Board-Computer/Tinker-Board-S/)
  - 2GiB RAM
  - 16GiB eMMC + 32GiB mmcblk storage
  - Armbian GNU/Linux 10 buster (arm32)
- 1 x [Raspberry Pi 4 B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
  - 4GiB RAM
  - 32GiB mmcblk + 128GiB ssd storage
  - Raspbian GNU/Linux 10 buster (arm32) (soon Ubuntu 20.04)
- 1 x [Raspberry Pi 3 B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/)
  - 1GiB RAM
  - 32GiB mmcblk storage - dead :/
  - Raspbian GNU/Linux 10 buster (arm32)
- 1 x Custom desktop w/ [Intel 4670K @ 4.2GHz](https://ark.intel.com/content/www/us/en/ark/products/75048/intel-core-i5-4670k-processor-6m-cache-up-to-3-80-ghz.html)
  - 24GiB RAM
  - 256GiB ssd storage + 1TiB HDD storage
  - Arch Linux (x86_64)
  - kvm host running [CONTAINER-OPTIMIZED OS](https://cloud.google.com/container-optimized-os/) VMs

## Networking

- Local lan is `10.12.0.0/22` (Why did i choose `10.12.0.0`?; no idea)
- metallb use range `10.12.1.10-10.12.1.40` for load balancing services
- Basic switch connecting the nodes together.
- Wireguard for connection to the outside world, using `10.10.0.0/24`
- Gateway is `10.12.1.1`
  - Three nodes run as gateways with access to WAN (still behind NAT tho.), eliminating gw SPOF

## TODO
- Look into running ceph
- Switch to [Cilium](https://cilium.io/) for networking (instead of flannel).
  - eBPF is awesome, and so is hubble
  - Ish. official support for aarch64, but missing images. Need to verify arm32
  - kernel on Armbian (for tinkerboard) has to be self compiled to enable eBPF :/
  - Add [Cluster Mesh](https://cilium.io/blog/2019/03/12/clustermesh/) to connect multiple clusters
- Switch to cgroup v2 only
  - Still no support in  most container runtimes like cri-o, docker, containerd. Partial support in runc and k8s, but works in podman and crun (yay)
- Try rook and ceph
  - Since rpi4 run aarch64, it should be possible to run ceph for block storage
  - Also, gluster has been runnning (maybe a bit too) stable for 1,5 years now, but since it runs on mmcblk-storage, it may blow up at any time... Block storage from SSDs would be awesome!!
  - There are still some stuff that has to be done for both aarch64 and arm32, but _should_ be possible
  - kernel on Armbian (for tinkerboard) has to be self compiled to enable ceph/rbd client support :/
- New image with the new hardware

## Software

- [k8s](https://k8s.io) does its things in a high availability manner. Running 3 nodes as masters, ensuring that stuff works even if one node dies. Using [metallb](https://metallb.universe.tf/) as a Layer 2 load-balancer for services with type `LoadBalancer`
- [gluster](https://www.gluster.org/) for distributed storage. Dead simple block storage that can be used inside the cluster via persistent volumes in k8s.
  - Looking into geo replication for off site backups.
  - [ceph](https://ceph.io/) is awesome too, but doesn't support `arm32` (and it _eats_ ram), but may be an alternative at a later stage.
  - Currently runs on 3 x 32GiB mmcblk storage devices, aka. SD cards. Looking into using USB 3 SSDs
- [keepalived](https://www.keepalived.org/) for configuring fallback routing for the three gateways, making the connection work even tho a gateway dies.

## Monitoring

- [Prometheus](https://prometheus.io/) with a set of custom exporters (including data from Home Assistant)
- [slack](https://slack.com) for alerting

## Home Automation

- [Home Assistant](https://www.home-assistant.io/) running inside Kubernetes. Floats beetween all nodes. Does things like turning on and off lights, and controlling the temperature based of various inputs.
- [emqx](https://github.com/emqx/emqx) as mqtt broker (mqtt is awesome btw.). Currently running 3 instances clustered together.
- [Node Red](https://nodered.org/) for creating automations. Connected to Home Assistant.
- [ESPHome](https://esphome.io) for creating simple & cheap sensors with `esp32/esp8266`s

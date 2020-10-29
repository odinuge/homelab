# homelab

> Odin's homelab setup

![homelab](/media/homelab_1.jpg)

## Goals

The primary goals of this project are...

- to have a highly-available cluster for home autmation & testing
- to have no SPOFs (Single Point of Failure)
- to have fun (most important)

## Hardware

- 1 x [Raspberry Pi 4B @ 8GiB](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
  - 8GiB RAM
  - 32GiB mmcblk (boot) + 128GiB ssd storage (64GiB for os & 64GiB for gluster)
  - Ubuntu 20.04 'Focal Fossa' (aarch64)
- 4 x [Raspberry Pi 4B @ 4GiB](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
  - 4GiB RAM
  - 32GiB mmcblk (boot) + 128GiB ssd storage (64GiB for os & 64GiB for gluster)
  - Ubuntu 20.04 'Focal Fossa' (aarch64)
- 3 x [Tinker Board S](https://www.asus.com/no/Single-Board-Computer/Tinker-Board-S/)
  - 2GiB RAM
  - 16GiB eMMC
  - Armbian GNU/Linux 10 buster (arm32)
- 1 x Custom desktop w/ [Intel 4670K @ 4.2GHz](https://ark.intel.com/content/www/us/en/ark/products/75048/intel-core-i5-4670k-processor-6m-cache-up-to-3-80-ghz.html)
  - 24GiB RAM
  - 256GiB ssd storage + 1TiB HDD storage
  - Arch Linux (x86_64)
  - kvm host running [CONTAINER-OPTIMIZED OS](https://cloud.google.com/container-optimized-os/) VMs

## Networking

- Local lan is `10.12.0.0/22` (Why did i choose `10.12.0.0`?; no idea)
- metallb use range `192.168.2.1-192.168.2.255` for high availability and load balancing services
- Basic switch connecting the nodes together.
- Wireguard for connection to the outside world, using `10.10.0.0/24`
- Gateway is `10.12.1.1`
  - Three nodes run as gateways with access to WAN (still behind NAT tho.), eliminating gw SPOF

## TODO

- Switch to cgroup v2 only
  - Waiting for new containerd builds for Ubuntu
- New image with the new hardware

## Software

- [k8s](https://k8s.io) does its things in a high availability manner. Running 3 nodes as masters, ensuring that stuff works even if one node dies.
  - [metallb](https://metallb.universe.tf/) as a Layer 2 load-balancer for services with type `LoadBalancer`
  - 3xRpi4 as master nodes
- [Longhorn](https://longhorn.io/) for distributed block storage. Pretty simple block storage for using PVCs in k8s, with simple backup systems and a nice shiny ui.
  - Currently runs on 4 x 64GiB USB3 SSD drives (64GiB for longhorn and 64GiB for system).
- [keepalived](https://www.keepalived.org/) for configuring fallback routing for the three gateways, making the connection work even tho a gateway dies.

## Monitoring

- [Prometheus](https://prometheus.io/) with a set of custom exporters (including data from Home Assistant)
  - [VictoriaMetrics](https://victoriametrics.com/) for long term storage
- [slack](https://slack.com) for alerting

## Home Automation

- [Home Assistant](https://www.home-assistant.io/) running inside Kubernetes. Floats beetween all nodes. Does things like turning on and off lights, and controlling the temperature based of various inputs.
- [emqx](https://github.com/emqx/emqx) as mqtt broker (mqtt is awesome btw.). Currently running 3 instances clustered together.
- [Node Red](https://nodered.org/) for creating automations. Connected to Home Assistant.
- [ESPHome](https://esphome.io) for creating simple & cheap sensors with `esp32/esp8266`s

# Old TODOS

- Looking into geo replication for off site backups.
  - Longhorn support backups to s3, and just works!
- Look into running longhorn
  - Longhorn looks awesome, but has no support for aarch64. _EDIT_: Supports aarch64 now!
- Look into running ceph
  - Small pain with images only supporting x86_64. Some support arm64 tho., but none works on arm32.
  - Deploying simple ceph cluster with rook used about 4.5GiB memory when idle, aka. eating a bit too much on rpi4
  - gluster works well, and is stabel as hell, and only use about ~200MiB per node, so ill stick with that.
- Switch to [Cilium](https://cilium.io/) for networking (instead of flannel).
  - eBPF is awesome, and so is hubble - But way too much work compared to the gain.
  - Has nice support for arm64 now, so thinking about dropping the arm32 devices and running cilium.
  - Flannel works just fine

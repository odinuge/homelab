# homelab

## Goals

The primary goals of this project are...

- to have a highly-available cluster for home autmation & testing
- to have no SPOFs (Single Point of Failure)
- to have fun (most important)

![homelab](/media/homelab_1.jpg)

## Hardware

- 3 x [Tinker Board S](https://www.asus.com/no/Single-Board-Computer/Tinker-Board-S/)
  - 2GiB RAM
  - 16GiB eMMC + 32GiB mmcblk storage
  - Armbian GNU/Linux 10 buster (arm32)
- 1 x [Raspberry Pi 4 B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
  - 4GiB RAM
  - 32GiB mmcblk + 128GiB ssd storage
  - Raspbian GNU/Linux 10 buster (arm32)
- 1 x [Raspberry Pi 3 B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/)
  - 2GiB RAM
  - 32GiB mmcblk storage
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

## Software

- [k8s](https://k8s.io) does its things in a high availability manner. Running 3 nodes as masters, ensuring that stuff works even if one node dies. Using [metallb](https://metallb.universe.tf/) as a Layer 2 load-balancer for services with type `LoadBalancer`
- [gluster](https://www.gluster.org/) for distributed storage. Dead simple block storage that can be used inside the cluster. Looking into geo replication for off site backups. [ceph](https://ceph.io/) is awesome too, but doesn't support `arm32` (and it _eats_ ram), but may be an alternative at a later stage.
- [keepalived](https://www.keepalived.org/) for configuring fallback routing for the three gateways, making the connection work even tho a gateway dies.

## Monitoring

- [Prometheus](https://prometheus.io/) with a set of custom exporters (including data from Home Assistant)
- [slack](https://slack.com) for alerting

## Home Automation

- [Home Assistant](https://www.home-assistant.io/) running inside Kubernetes. Floats beetween all nodes. Does things like turning on and off lights, and controlling the temperature based of various inputs.
- [emqx](https://github.com/emqx/emqx) as mqtt broker (mqtt is awesome btw.). Currently running 3 instances clustered together.
- [Node Red](https://nodered.org/) for creating automations. Connected to Home Assistant.
- [ESPHome](https://esphome.io) for creating simple & cheap sensors with `es32/esp8266`s

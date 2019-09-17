# homelab

< insert img here >

## Goals

The primary goals of this project are...

- to have a highly-available cluster for home autmation & testing
- to have no SPOFs (Single Point of Failure)
- to have fun

## Hardware

- 3 x Tinker Board S
  - 2GiB RAM
  - 16GiB internal storage
  - 32GiB mmcblk storage
  - arm32
- 1 x Raspberry Pi 4 B
  - 4GiB RAM
  - 32GiB mmcblk storage
  - arm32 (aarch64 soon)
- 1 x Raspberry Pi 3 B+
  - 2GiB RAM
  - 32GiB mmcblk storage
  - 128GiB ssd storage
  - arm32 (aarch64 soon)
- 1 x Define R4
  - 24GiB RAM
  - 256GiB ssd storage
  - Intel 4670K @ 4.2GHz

## Networking

- Local lan is `10.12.0.0/22` (Why did i choose `10.12.0.0`?; no idea)
- Wireguard for connection to the outside world, using `10.10.0.0/24`
- Gateway is `10.12.1.1`
  - Three nodes run as gateways with access to WAN (still behind NAT tho.), eliminating gw SPOF

## Software

- [k8s](https://k8s.io) does its things in a high availability manner
- [gluster](https://www.gluster.org/) for distributed storage. [ceph](https://ceph.io/) doesn't support `arm32`, but may be an alternative for

## Monitoring

- [Prometheus](https://prometheus.io/) with a set of custom exporters (including)
- [slack](https://slack.com) for alerting

## Home Automation

- [Home Assistant](https://www.home-assistant.io/) running inside Kubernetes. Floats beetween all nodes
- [emqx](https://github.com/emqx/emqx) as mqtt broker (currently 3 instances)
- [Node Red](https://nodered.org/) for creating automations. Connected to Home Assistant.
- [ESPHome](https://esphome.io) for creating simple sensors with `es32/esp8266`s

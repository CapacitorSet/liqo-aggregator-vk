# liqo-aggregator-vk

This repository is a fork of Liqo meant to be used with [liqo-broker](https://github.com/CapacitorSet/liqo-broker). It patches the virtual kubelet to support an aggregator broker.

## Patches

The virtual kubelet is patched to support a second, "external" IPAM. When a remote IP is translated into a local one, the virtual kubelet will check if the IP is in the pod CIDR for the external IPAM, and if so it will pick that IPAM for translating the IP (rather than the default, local IPAM).

The end result is that in a daisy chain of NamespaceOffloadings, networking is configured correctly so that the consumer can connect to the twice-offloaded pods.

## Building

```sh
docker build --build-arg COMPONENT=virtual-kubelet -t capacitorset/liqo-aggregator-vk -f build/common/Dockerfile .
```

[![Go Report Card](https://goreportcard.com/badge/github.com/costela/hcloud-ip-floater)](https://goreportcard.com/report/github.com/costela/hcloud-ip-floater)
[![Docker Automated build](https://img.shields.io/docker/cloud/automated/costela/hcloud-ip-floater.svg)](https://hub.docker.com/r/costela/hcloud-ip-floater)
[![Docker Build Status](https://img.shields.io/docker/cloud/build/costela/hcloud-ip-floater.svg)](https://hub.docker.com/r/costela/hcloud-ip-floater/builds)
[![Image Info](https://images.microbadger.com/badges/image/costela/hcloud-ip-floater.svg)](https://hub.docker.com/r/costela/hcloud-ip-floater/tags)


# Hetzner Cloud™ IP Floater

This small [kubernetes](https://kubernetes.io/) controller manages the attachment of
[hetzner cloud](https://hetzner.cloud) ("hcloud") floating IPs to kubernetes nodes.

It watches for changes to kubernetes `LoadBalancer` services, chooses one of the nodes where its pods are scheduled and
attaches its assigned floating IP to the selected node.

The service IP assignment is left to a separate component, like [MetalLB](https://metallb.universe.tf/).

## Installation

The controller can be installed to a cluster using e.g. [kustomize](https://kustomize.io/). Simply `kubectl apply -k` the
following `kustomization.yaml`:

```yaml
namespace: hcloud-ip-floater
bases:
  - github.com/costela/hcloud-ip-floater/deploy?ref=v0.1.4
secretGenerator:
  - name: hcloud-ip-floater-secret-env
    literals:
      - HCLOUD_IP_FLOATER_HCLOUD_TOKEN=<YOUR HCLOUD API TOKEN HERE>
```

The provided deployment manifest expects a secret named `hcloud-ip-floater-secret-env` to exist, which is the
recommended location for storing the hcloud API token.

It's also possible to provide a `configMapGenerator` called `hcloud-ip-floater-config-env` with the non-secret options
listed in the [configuration options](#configuration-options) section below.

⚠ in order for the controller to attach IPs to the hcloud nodes, the k8s nodes **must** use the same names as in
hcloud.

## Configuration options

Either as command line arguments or environment variables.

### `--hcloud-token` or `HCLOUD_IP_FLOATER_HCLOUD_TOKEN` **(required)**

API token for hetzner cloud access.

### `--service-label-selector` or `HCLOUD_IP_FLOATER_SERVICE_LABEL_SELECTOR`

Service label selector to use when watching for kubernetes services. Any services that do not match this selector will be ignored by the controller.

**Default**: `hcloud-ip-floater.cstl.dev/ignore!=true`

### `--manual-assignment-label` or `HCLOUD_IP_FLOATER_MANUAL_ASSIGNMENT_LABEL`

This is experimental and hasn't seen a lot of production use!

Label name used to manually assign floating IPs on a Pod.

This can be useful when other means of routing the traffic to a pod than a load balancer are used. E.g. you could be using the [`ipvlan` CNI plugin](https://www.cni.dev/plugins/current/main/ipvlan/) with [Multus](https://github.com/k8snetworkplumbingwg/multus-cni/).
The label accepts a comma-seperated list of floating IP addresses to assign to the node the pod is on.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    hcloud-ip-floater.cstl.dev/floating-ip: "1.2.3.4,2.3.4.5"
```

This mechanism will be ignored if there is a service with the same IP present.

**Default**: `hcloud-ip-floater.cstl.dev/floating-ip`

### `--floating-label-selector` or `HCLOUD_IP_FLOATER_FLOATING_LABEL_SELECTOR`

Label selector for hcloud floating IPs. Floating IPs that do not match this selector will be ignored by the controller.

**Default**: `hcloud-ip-floater.cstl.dev/ignore!=true`

### `--log-level` or `HCLOUD_IP_FLOATER_LOG_LEVEL`

Log output verbosity (debug/info/warn/error)

**Default**: `warn`

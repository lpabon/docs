# Kubernetes CSI Specification Extensions
The following documentation describes the extensions to the [CSI
Specification](https://github.com/container-storage-interface/spec/blob/master/spec.md)
for Kubernetes features.

# Ephemeral Volume Support

## Status and Releases

* **Status:** v1.16-Beta

## Description

First released in v1.15 of Kubernetes, ephemeral volumes are used as temporary
storage for PODs in Kubernetes. Unlike persistent volumes, these volumes are
created when a POD started and deleted when a POD is stopped.

This document focuses on the requirement on a CSI driver to support this feature
and not on the Kubernetes support. For information on Kubernetes support, please
read [Ephemeral Local
  Volumes](https://kubernetes-csi.github.io/docs/ephemeral-local-volumes.html).

## Specification

### Overview
Due to this requirement these volumes will not be created using the [Controller
Service](https://github.com/container-storage-interface/spec/blob/master/spec.md#controller-service-rpc),
but the [Node
Service](https://github.com/container-storage-interface/spec/blob/master/spec.md#node-service-rpc),
instead. When the
[kubelet](https://github.com/kubernetes/kubernetes/blob/70132b0f130acc0bed193d9ba59dd186f0e634cf/pkg/volume/csi/csi_mounter.go#L329)
calls _NodePublishVolume_, it is the responsibility of the driver to create the
volume during that call, then publish the volume to the specified location. When
the `kubelet` calls _NodeUnpublishVolume_, it is the responsibility of the CSI
driver to delete the volume.

### Extensions

#### [NodePublishVolume](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodepublishvolume)

##### Arguments

* **volume_id**: Volume ID will be created by the Kubernetes and pass to the
  driver by the kubelet.
* **volume_context["csi.storage.k8s.io/ephemeral"]**: This value will be
  available and it will be equal to `"true"`.

##### Workflow

The driver will receive the appropriate arguments as defined above when an
ephemeral volume is requested. The driver will create and publish the volume
to the specified location as noted in the _NodePublishVolume_ request. Size of
the volume will __XXX__TBD.

#### [NodeUnpublishVolume](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodeunpublishvolume)

##### Arguments

No changes

##### Workflow

The driver is responsible of deleting the ephemeral volume once it has
unpublished the volume. It MAY delete the volume before finishing the request,
or after the request to unpublish is returned.

## References

* [CSI Host Path
  driver ephemeral volumes support](https://github.com/kubernetes-csi/csi-driver-host-path/blob/9fdddc2061b9013286e01189b2bf3268276af99b/pkg/hostpath/nodeserver.go#L63-L82)
* [Issue 82507: Drop VolumeLifecycleModes field from CSIDriver API before
  GA](https://github.com/kubernetes/kubernetes/issues/82507)
* [Issue 75222: CSI Inline - Update CSIDriver to indicate driver
  mode](https://github.com/kubernetes/kubernetes/issues/75222)

Spec to the ephemeral
https://github.com/kubernetes/kubernetes/blob/70132b0f130acc0bed193d9ba59dd186f0e634cf/staging/src/k8s.io/api/storage/v1beta1/generated.proto#L96-L97

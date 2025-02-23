---
layout: layout.pug
navigationTitle: Release notes
title: Release notes
menuWeight: 15
excerpt: View release-specific information for Konvoy
enterprise: false
---

## Release Notes

### Version 1.1.0 - Released 12 August 2019


| Kubernetes support | Version |
| ------------------ | ------- |
|**Minimum** | 1.15.2 |
|**Maximum** | 1.15.x |
|**Default** | 1.15.2 |

#### Breaking changes

This version expects a different structure for the `inventory.yaml` file, one change is when using the bastion node feature, and an additional `version:` var.

Previous versions:

```yaml
control-plane:
…
node:
…
all:
  vars:
    bastion_hosts: [ec2-18-237-7-198.us-west-2.compute.amazonaws.com, ec2-34-221-251-83.us-west-2.compute.amazonaws.com]
      bastion_user: "bastion"
      bastion_port: 2222
      ansible_user: "centos"
      ansible_port: 22
```

This version, expects `bastion` to be its own Ansible group:

```yaml
control-plane:
…
node:
…
bastion:
  hosts:
    10.0.131.50:
      ansible_host: ec2-18-237-7-198.us-west-2.compute.amazonaws.com
    10.0.127.36:
      ansible_host: ec2-34-221-251-83.us-west-2.compute.amazonaws.com
  vars:
    ansible_user: "bastion"
    ansible_port: 2222
all:
  vars:
    version: "v1beta1"
    ansible_user: "centos"
    ansible_port: 22
```

#### Improvements

- Lock the Kubernetes RPM packages (kubelet, kubeadm, kubectl, containerd.io), preventing them from being upgraded when running yum upgrade.
- [AWS] Force detach EBS volumes when deleting them, preventing possible errors when running `konvoy down`.
- Fix a well known [kmem issue](https://github.com/kubernetes/kubernetes/issues/61937) by using a custom built Kubelet with kmem accounting disabled.

#### Addons improvements

- Enabled the service monitor for Velero.
- Added dashboards for Calico and Velero.

#### Bug fixes

- [AWS] Fix a bug where the installer would fail provisioning when using bastion nodes in a multi AZ cluster.
- Support cluster nodes that do not yet have Python installed.

#### Component version changes

- Kubernetes `v1.15.2`

#### Known issues and limitations

Known issues and limitations don’t necessarily affect all customers, but might require changes to your environment to address specific scenarios.
The issues are grouped by feature, functional area, or component.
Where applicable, issue descriptions include one or more issue tracking identifiers enclosed in parenthesis for reference.

-   Docker provisioner reports potential issues if there are insufficient resources.

    If you attempt to deploy a cluster on a machine that does not have enough resources, you might see issues when the installation starts to deploy Addons.
    For example, if you see an error message similar to _could not check tiller installation_, the root cause is typically insufficient resources.

-   Dex should always be enabled.

    For this release of Konvoy, `dex`, `dex-k8s-auth`, and `traefik-auth` are tightly coupled and must all be enabled.
    Disabling any of these add-ons will prevent certain operations from working correctly.
    This tight coupling will be addressed in a future release.

-   The authentication token has no permissions.

    After logging in through an identity provider, regardless of the source (password, or otherwise), the identified user has no permissions assigned.
    To enable the authenticated user to perform administrative actions, you must manually add role bindings.

-   Upgrades might fail when `workers` is set to one.

    The upgrade command might fail when the cluster is configured with only one worker. To work around this issue, add an additional worker for the upgrade.

### Version 1.0.0 - Released 3 August 2019

[Download](#TBA)

| Kubernetes support | Version |
| ------------------ | ------- |
|**Minimum** | 1.15.1 |
|**Maximum** | 1.15.x |
|**Default** | 1.15.1 |

Konvoy is a complete, standalone distribution of Kubernetes that enables you to provision native Kubernetes clusters with a suite of
[Cloud Native Computing Foundation (CNCF)](https://www.cncf.io) and community tools.

This is the initial release.

If you have Konvoy deployed in a production environment, see [Known issues and limitations](#known-issues) to see if any potential operational changes for specific scenarios apply to your environment.

#### What's new in this release

This release includes new features and capabilities for installation and deployment, networking, security, storage, and cluster administration.

Highlights for the features and capabilities introduced in this release are grouped by functional area.

##### Installation

-   Supports provisioning of Kubernetes using multiple infrastructure providers, including:
    - AWS
    - Docker
    - None (on-prem)
-   Installs Kubernetes using `kubeadm`.
-   Installs and configures `containerd` runtime.
-   Provides an on-premise pre-flight check to ensure deployment readiness.
-   Supports `bastion` host-based installation.
-   Supports HTTP proxy.
-   Supports the use of labels and taints for creating node pools.
-   Installs a default set of platform service components (Addons) to support the following features:
    - Monitoring and alerting
    - Logging
    - Storage (CSI)
    - Networking
    - Identity broker
    - Dashboards

##### Networking

- Uses `keepalived` to maintain a highly-available control plane where an external load balancer is not available.
- Uses Calico CNI to provide pod-to-pod connectivity and network policy.
- Uses CoreDNS for service discovery.
- Allows the use of MetalLB for load balancing where an external load balancer is not available.
- Uses Traefik for layer-7 ingress.

##### Security

- Provides a federated OpenID Connect using `dex`.

##### Storage

- Provides the AWS EBS CSI provisioner for AWS installations.
- Provides the Static Local Volume Provisioner for on-prem installations.

##### Day 2 operations

-   Provides the following for monitoring:
    -   Prometheus
    -   Alert-manager
    -   Grafana
    -   Node exporter
    -   Kube-state-metrics
    -   Various service monitors
    -   Dashboards for kubernetes components, including:
        - Nodes
        - Pods
        - Kubelet
        - Scheduler
        - Kube-apiserver
        - StatefulSets
        - PersistentVolumes
    -   Dashboards for other services, including:
        - Traefik
        - Grafana
        - CoreDNS
        - Local volume provisioner
        - Etcd
        - Prometheus
        - Fluent Bit
        - Elasticsearch
        - Volume space usage
-   Provides pre-configured alerts ([see external link for list][prometheus-rules])
-   Provides the following for logging:
    - Elasticsearch
    - Fluent Bit
    - Kibana
-   Provides backup and restore services for the cluster state using Velero, including:
    - Scheduled automatic backups
    - Use of minio for S3 storage
-   Provides a method for upgrading Kubernetes and the associated Addons.
-   Provides a method for deleting the cluster.
-   Provides troubleshooting tools, including
    - Pre-flight checks
    - Node checks
    - Kubernetes checks
    - Addon checks
    - Diagnostics bundle generation

For more information about any of these features,see the [Konvoy documentation][konvoy-doc].

#### Known issues and limitations

Known issues and limitations don’t necessarily affect all customers, but might require changes to your environment to address specific scenarios.
The issues are grouped by feature, functional area, or component.
Where applicable, issue descriptions include one or more issue tracking identifiers enclosed in parenthesis for reference.

-   Docker provisioner reports potential issues if there are insufficient resources.

    If you attempt to deploy a cluster on a machine that does not have enough resources, you might see issues when the installation starts to deploy Addons.
    For example, if you see an error message similar to _could not check tiller installation_, the root cause is typically insufficient resources.

-   EL7-based distributions run out of memory.

    EL7-based distributions (CentOS 7, RHEL 7, and SLES 7) have a set of kernel bugs that leak memory when `kmem` accounting is enabled. Kubernetes `kubelet` enables `kmem` accounting.
    To address this issue, a separate distribution package (RPM) with `kmem` accounting disabled will be published in a hosted repository when available.

-   Dex should always be enabled.

    For this release of Konvoy, `dex`, `dex-k8s-auth`, and `traefik-auth` are tightly coupled and must all be enabled.
    Disabling any of these add-ons will prevent certain operations from working correctly.
    This tight coupling will be addressed in a future release.

-   The authentication token has no permissions.

    After logging in through an identity provider, regardless of the source (password, or otherwise), the identified user has no permissions assigned.
    To enable the authenticated user to perform administrative actions, you must manually add role bindings.

-   Upgrades might fail when `workers` is set to one.

    The upgrade command might fail when the cluster is configured with only one worker. To work around this issue, add an additional worker for the upgrade.

-   Kubernetes RPM packages might be upgraded unintentionally via yum.

    The Kubernetes RPM packages (kubelet, kubeadm, kubectl, containerd.io) are managed by Konvoy, and are not supposed to be upgraded when a user issues a `yum update`.
    Currently, these RPM packages are not excluded for update.
    To workaround that, the user needs to use the following command `yum update --exclude=kubelet*  --exclude=kubeadm* --exclude=kubctl* --exclude=containerd*` to update other system packages.
    One could also add `exclude=kubelet* kubeadm* kubctl* containerd*` to `/etc/yum.conf`.

<!--
### Previous releases
Add links to previous release notes
-->

### Additional resources

<!-- Add links to external documentation as needed -->

For information about installing and using Konvoy, see the [Konvoy documentation][konvoy-doc].

For information about working with native Kubernetes, see the [Kubernetes documentation][kubernetes-doc].

[prometheus-rules]: https://github.com/helm/charts/tree/master/stable/prometheus-operator/templates/prometheus/rules
[konvoy-doc]:https://docs.d2iq.com/ksphere/konvoy
[kubernetes-doc]:https://kubernetes.io/docs/home/

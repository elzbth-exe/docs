# Prerequisites

Kube-OVN is a CNI-compliant network system that depends on the Kubernetes environment and
the corresponding kernel network module for its operation.
Below are the operating systems and software versions tested,
the environment configuration and the ports that need to be opened.

## Software Version

- Kubernetes >= 1.23.
- Docker >= 1.12.6, Containerd >= 1.3.4.
- OS: CentOS 7/8, Ubuntu 16.04/18.04/20.04.
- For other Linux distributions, please make sure `geneve`, `openvswitch`, `ip_tables` and `iptable_nat` kernel modules exist.

*Attention*：

1. For CentOS kernel version 3.10.0-862, a bug exists in the `netfilter` modules that lead Kube-OVN embed nat and lb failure. Please update the kernel and check the following bug info: [Floating IPs broken after kernel upgrade to Centos/RHEL 7.5 - DNAT not working](https://bugs.launchpad.net/neutron/+bug/1776778){: target="_blank" }.
2. The kernel version 4.18.0-372.9.1.el8.x86_64 in Rocky Linux 8.6 has a TCP connection problem [(TCP connection failed in Rocky Linux 8.6)](https://github.com/kubeovn/kube-ovn/issues/1647){: target="_blank" }，please update the kernel to 4.18.0-372.13.1.el8_6.x86_64 or later。
3. For kernel version 4.4, the related `openvswitch` module has some issues for ct，please update the kernel version or manually compile the `openvswitch` kernel module.
4. When building a Geneve tunnel, IPv6 in the kernel should be enabled，check the kernel bootstrap options with `cat /proc/cmdline`.Check [Geneve tunnels don't work when ipv6 is disabled](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1794232){: target="_blank" } for the detail bug info.

## Environment Setup
Make sure:

- IPv6 is enabled, if the kernel bootstrap options contain `ipv6.disable=1`, it should be set to `0`.
- `kube-proxy` works, Kube-OVN can visit the `kube-apiserver` from the Service ClusterIP.
- `CNI` is enabled in the kubelet and cni-bin and cni-conf are in their default directories.
- Kubelet bootstrap options should contain `--network-plugin=cni --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d`.
- No other CNI is installed or that any preinstalled CNI has been removed. Check if any config files still exist in`/etc/cni/net.d/`.

## Ports Need Open

| Component           | Port                                          | Usage                               |
| ------------------- | --------------------------------------------- | ----------------------------------- |
| ovn-central         | 6641/tcp, 6642/tcp, 6643/tcp, 6644/tcp        | ovn-db and raft server listen ports |
| ovs-ovn             | Geneve 6081/udp, STT 7471/tcp, Vxlan 4789/udp | tunnel ports                        |
| kube-ovn-controller | 10660/tcp                                     | metrics port                        |
| kube-ovn-daemon     | 10665/tcp                                     | metrics port                        |
| kube-ovn-monitor    | 10661/tcp                                     | metrics port                        |

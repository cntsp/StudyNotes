kubelet安装失败



```shell
dpkg: dependency problems prevent configuration of kubelet:
 kubelet depends on ebtables; however:
  Package ebtables is not installed.

dpkg: error processing package kubelet (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 kubelet
Selecting previously unselected package kubectl.
(Reading database ... 110655 files and directories currently installed.)
Preparing to unpack .../kubectl_1.18.5-00_amd64.deb ...
Unpacking kubectl (1.18.5-00) ...
Setting up kubectl (1.18.5-00) ...
Selecting previously unselected package kubeadm.
(Reading database ... 110656 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.18.5-00_amd64.deb ...
Unpacking kubeadm (1.18.5-00) ...
dpkg: dependency problems prevent configuration of kubeadm:
 kubeadm depends on kubelet (>= 1.13.0); however:
  Package kubelet is not configured yet.

dpkg: error processing package kubeadm (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 kubeadm
```


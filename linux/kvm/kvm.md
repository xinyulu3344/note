# kvm

## kvm软件安装

查看CPU是否支持VT技术

```bash
cat /proc/cpuinfo | grep -E "vmx|svm"
```

卸载kvm

```bash
yum remove `rpm -qa | egrep 'qemu|virt|KVM'`
rm -rf /var/lib/libvirt /etc/libvirt
```

安装软件

```bash
yum install qemu-kvm libvirt
```


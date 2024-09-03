1. **What is KVM?**: KVM (Kernel-based Virtual Machine) is a popular open-source virtualization technology built into the Linux kernel. It allows you to run multiple virtual machines (VMs) on a single physical host, each with its own operating system, effectively turning a Linux system into a hypervisor.

### **KVM Interview Concepts**:

1. **Hypervisor Architecture**: KVM (Kernel-based Virtual Machine) is a Type 2 hypervisor running within the Linux kernel. It turns Linux into a bare-metal hypervisor with VM management.
    
2. **libvirt**: This provides an API for managing KVM VMs using `virsh` (command-line management), `virt-manager` (GUI), and other tools.
    
3. **Networking**: Be prepared to discuss bridge networking, which connects VMs directly to the host’s network, NAT for internal IPs, and macvtap interfaces.
    
4. **Storage**: Explain disk formats like QCOW2 for thin provisioning and snapshot support, RAW formats, and integration with storage technologies like LVM (Logical Volume Manager) and Ceph for distributed storage.
    
5. **Security**: Know about security mechanisms like SELinux and AppArmor for Linux, and how KVM uses sVirt to enforce mandatory access control (MAC) for VM isolation.
    
6. **Automation**: Highlight how tools like Ansible or Terraform can automate the provisioning of VMs and cloud-init for configuration management within VMs.
    
7. **Performance Tuning**: Understanding CPU pinning, NUMA (Non-Uniform Memory Access) configuration, and tuning memory and I/O for performance optimization is key in KVM environments.
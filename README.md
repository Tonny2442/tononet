# Tononet Instructions

I run a homelab and for portfolio and a place to put my thoughts and to inform others, this is a repository based on everything I do, how to do it and what it does.

# Computer instances

I have one hypervisor running Proxmox VE. The system itself is a Dell Precision T1700, with 16GB of DDR3. The systems I run under this are Windows Server 2019 domain controllers, WDS instances and WS Update Services instances. All of my Server systems run Windows Server 2019. My Linux systems run Debian, and my Windows clients run the latest October 2020 Update (20H2).

# Tools that I use

```
Active Directory Domain Services
Hyper-V Virtual Machines
QEMU-KVM Virtual Machines
Proxmox Virtual Environment
Windows Server Update Services
Windows Deployment Services
Microsoft Deployment Toolkit
System Center Configuration Endpoint
```
All of those tools are available under the [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/) or under the [GNU License Agreement](https://www.gnu.org/licenses/gpl-3.0.en.html).

# Why do I do this?

I run these instances because they give me control and experience over the operation of my Windows instances and my Linux instances. It also lets me run more than I could running on bare-metal hardware. 

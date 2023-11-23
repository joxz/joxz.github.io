---
date: 2023-11-22
description: "Collection of log locations for Azure VM extensions"
categories:
  - troubleshooting

tags:
  - vm-extensions
  - linux

comments: true
authors: [jo]
---

# Azure VM Agent and Extension Logs

This is a collection of links and file locations used to troubleshoot various Azure VM extensions or agents. Mainly done so I don't have to google it everytime

[:octicons-link-external-16: Azure virtual machine extensions and features](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/overview){target="_blank"}

<!-- more -->

## Azure Windows VM Agent / waagent

=== "Windows"
    !!! quote 
        The Microsoft Azure Windows VM Agent is a secure, lightweight process that manages virtual machine (VM) interaction with the Azure fabric controller. The Azure Windows VM Agent has a primary role in enabling and executing Azure virtual machine extensions. VM extensions enable post-deployment configuration of VMs, such as installing and configuring software. VM extensions also enable recovery features such as resetting the administrative password of a VM. Without the Azure Windows VM Agent, you can't run VM extensions.

=== "Linux"
    !!! quote 
        The Microsoft Azure Linux VM Agent (waagent) manages Linux and FreeBSD provisioning, along with virtual machine (VM) interaction with the Azure fabric controller. In addition to the Linux agent providing provisioning functionality, Azure provides the option of using cloud-init for some Linux operating systems.

[:simple-github: https://github.com/Azure/WindowsVMAgent](https://github.com/Azure/WindowsVMAgent){target="_blank"}

[:simple-github: https://github.com/Azure/WALinuxAgent](https://github.com/Azure/WALinuxAgent){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-windows](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-windows){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/windows-azure-guest-agent](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/windows-azure-guest-agent){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/troubleshoot](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/troubleshoot){target="_blank"}

=== "Windows"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Name | `WindowsVMAgent` (1) |
    | Log file | `C:\WindowsAzure\Logs\WaAppAgent.log` |
    | Installer logs | `C:\WindowsAzure\Logs\TransparentInstaller.log` |
    | Extension logs | `C:\WindowsAzure\Logs\Plugins\` |
    | Extension config | `C:\Packages\Plugins\` |

    </div>

    1.  !!! tip ""

        :octicons-light-bulb-16: Note: The Windows process is called `WindowsAzureGuestAgent.exe`

=== "Linux"
    | Name | Location |
    |---|---|
    | Name | `WALinuxAgent` |
    | Config file | `/etc/waagent.conf` |
    | Log file | `/var/log/waagent.log` |
    | Extension logs | `/var/log/azure/<extensionName>` |
    | Extension binaries | `/var/lib/waagent/<extensionName>` |
    | Verify agent is running | `systemctl status walinuxagent` |
    | journald logs | `journalctl -u walinuxagent` |


## Azure Arc Connected Machine Agent

![](https://learn.microsoft.com/en-us/azure/azure-arc/servers/media/agent-overview/connected-machine-agent.png){ loading=lazy }

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview](https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-arc/servers/troubleshoot-agent-onboard](https://learn.microsoft.com/en-us/azure/azure-arc/servers/troubleshoot-agent-onboard){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-arc/servers/troubleshoot-vm-extensions#general-troubleshooting](https://learn.microsoft.com/en-us/azure/azure-arc/servers/troubleshoot-vm-extensions#general-troubleshooting){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-arc/servers/azcmagent-check](https://learn.microsoft.com/en-us/azure/azure-arc/servers/azcmagent-check){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#agent-resources](https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#agent-resources){target="_blank"}

Arc enabled servers support the extensions listed in the links for [:octicons-link-external-16: Windows](https://learn.microsoft.com/en-us/azure/azure-arc/servers/manage-vm-extensions#windows-extensions) and [:octicons-link-external-16: Linux](https://learn.microsoft.com/en-us/azure/azure-arc/servers/manage-vm-extensions#linux-extensions). 

For example, the Qualys extension used in [:octicons-link-external-16: the article on how to deploy the Qualys extension with Azure Policy](qualys-azpolicy.md){target="_blank"} is not available for arc enabled servers.

=== "Windows"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Connected Machine Agent log | `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` |
    | Hybrid Instance Metadata Service (himds) log (2) | `%ProgramData%\AzureConnectedMachineAgent\Log\himds.log` |
    | Guest agent log | `%SystemDrive%\ProgramData\GuestConfig\ext_mgr_logs` |
    | Specific extension logs | `%SystemDrive%\ProgramData\GuestConfig\extension_logs\<Extension>` |
    | Download path for VM extensions | `%SystemDrive%\%ProgramFiles%\AzureConnectedMachineAgent\ExtensionService\downloads` |
    | Extension install dir | `%SystemDrive%\Packages\Plugins\<extension>` (1) |

    </div>

    1. :octicons-light-bulb-16: Note: Same path as the `WindowsAzureGuestAgent` installs extensions to
    2. [:octicons-link-external-16:  Metadata information](https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#instance-metadata){target="_blank"}  about a connected machine is collected after the Connected Machine agent registers with Azure Arc-enabled servers

    Collect all logs and store in a ZIP file:

    ```sh
    azcmagent logs --full --output "C:\temp\azcmagent-logs.zip"
    ```

=== "Linux"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Connected Machine Agent log | `/var/opt/azcmagent/log/azcmagent.log` |
    | Hybrid Instance Metadata Service (himds) log (2) | `/var/opt/azcmagent/log/himds.log` |
    | Guest agent log | `/var/lib/GuestConfig/ext_mgr_logs` |
    | Specific extension logs | `/var/lib/GuestConfig/extension_logs/` |
    | Download path for VM extensions | `/opt/GC_Ext/downloads` |
    | Extension install dir | `/var/lib/waagent/<extension>` (1) |

    </div>

    1. :octicons-light-bulb-16: Note: Same path as the `WALinuxAgent` installs extensions to
    2. [:octicons-link-external-16:  Metadata information](https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#instance-metadata){target="_blank"}  about a connected machine is collected after the Connected Machine agent registers with Azure Arc-enabled servers

    Collect all logs and store in a ZIP file

    ```sh
    azcmagent logs --full --output "/tmp/azcmagent-logs.zip"
    ```

Agent connectivity check:

```sh
azcmagent check --location "westeurope" -v
```

## Azure Monitor Agent

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-troubleshoot-linux-vm](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-troubleshoot-linux-vm){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-monitor/agents/use-azure-monitor-agent-troubleshooter](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/use-azure-monitor-agent-troubleshooter){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-monitor/agents/agents-overview](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/agents-overview){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-data-collection-endpoint](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-data-collection-endpoint){target="_blank"}

!!! info
    A system-assigned or user-assigned managed identity is necessary for the agent to work

=== "Windows"
    | Name | Location |
    |---|---|
    | Extension name | `AzureMonitorWindowsAgent` |
    | Runtime logs | `C:\Resources\Azure Monitor Agent\` |
    | Binary store | `C:\Program Files\Azure Monitor Agent\` |
    | Run installation with logging enabled | `Msiexec /I AzureMonitorAgentClientSetup.msi /L*V <logfilename>` |

    Run Windows Troubleshooter:

    ```bat
    cd "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\{version}\Troubleshooter"
    AgentTroubleshooter --ama
    ```

=== "Linux"
    | Name | Location |
    |---|---|
    | Extension name | `AzureMonitorLinuxAgent` |
    | Core agent logs | `/var/opt/microsoft/azuremonitoragent/log/mdsd.*` |
    | Download location for DCRs | `/etc/opt/microsoft/azuremonitoragent/config-cache/configchunks/` |
    | Configuration store | `/etc/opt/microsoft/azuremonitoragent/config-cache/configchunks/` |
    | Verify agent is running | `systemctl status azuremonitoragent` |

    Run Linux Troubleshooter:

    ```sh
    cd /var/lib/waagent/Microsoft.Azure.Monitor.AzureMonitorLinuxAgent-{version}/ama_tst
    sudo sh ama_troubleshooter.sh -A

    # create zip file with logs (you'll be asked for a file location to create the zip file):
    sudo sh ama_troubleshooter.sh -A L
    ```

## Guest Configuration Extension

My first steps in diving into Azure Guestconfiguration (Azure Automanage Machine Configuration) are written down in [:octicons-link-external-16: part 1](az-guestconfig-p1.md) and [:octicons-link-external-16: part 2](az-guestconfig-p2.md).

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/governance/machine-configuration/overview](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/overview){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/guest-configuration](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/guest-configuration){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/governance/machine-configuration/agent-release-notes](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/agent-release-notes){target="_blank"}

[:simple-github: https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Guest%20Configuration](https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Guest%20Configuration){target="_blank"}

[:simple-github: https://github.com/Azure/azure-policy/tree/master/samples/GuestConfiguration/package-samples/resource-modules](https://github.com/Azure/azure-policy/tree/master/samples/GuestConfiguration/package-samples/resource-modules){target="_blank"}

[:simple-github: https://github.com/azure/nxtools#getting-started](https://github.com/azure/nxtools#getting-started){target="_blank"}

!!! info
    A system-assigned managed identity is necessary for the agent to work

=== "Windows"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Extension name | `AzurePolicyforWindows` |
    | Client log files Azure VM | `C:\ProgramData\GuestConfig\gc_agent_logs\gc_agent.log` |
    | Client log files Arc | `C:\ProgramData\GuestConfig\arc_policy_logs\gc_agent.log` |
    | Agent files (1) | `C:\ProgramData\guestconfig\configuration\` |

    </div>

    1.  The machine configuration agent downloads content packages to a machine and extracts the contents

=== "Linux"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Extension name | `AzurePolicyforLinux` |
    | Client log files Azure VM | `/var/lib/GuestConfig/gc_agent_logs/gc_agent.log` |    
    | Client log files Arc | `/var/lib/GuestConfig/arc_policy_logs/gc_agent.log` |
    | Agent files (1) | `/var/lib/GuestConfig/Configuration/` | 

    </div>

    1.  The machine configuration agent downloads content packages to a machine and extracts the contents


## Custom Script Extension

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux){target="_blank"}

[:simple-github: https://github.com/Azure/custom-script-extension-linux](https://github.com/Azure/custom-script-extension-linux){target="_blank"}

=== "Windows"
    <div class="annotate" markdown>

    | Name | Location |
    |---|---|
    | Extension name | `CustomScriptExtension` |
    | Extension output filepath | `C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\` |
    | Extension file download path | `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>` (1) |

    </div>

    1. In the preceding path, `<n>` is a decimal integer that might change between executions of the extension. The `1.*` value matches the actual, current typeHandlerVersion value of the extension. For example, the actual directory could be `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2`. Read [ :octicons-link-external-16: here](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows#troubleshoot-and-support){target="_blank"}

=== "Linux"
    | Name | Location |
    |---|---|
    | Extension name | `customScript` |
    | waagent logs containing `customScript` | `sudo cat /var/log/waagent.log | grep "Microsoft.Azure.Extensions.customScript` |
    | Extension logs | `/var/log/azure/custom-script/handler.log` |
    | Extension file download path | `/var/lib/waagent/custom-script/download/0/` |

## Dependency Agent

[https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-dependency-windows](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-dependency-windows)

[https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-dependency-linux](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-dependency-linux)

=== "Windows"
    | Name | Location |
    |---|---|
    | Extension name | `DependencyAgentWindows` |
    | Extension logs | `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Monitoring.DependencyAgent\` |

=== "Linux"
    | Name | Location |
    |---|---|
    | Extension name | `DependencyAgentLinux` |
    | Extensions logs | `/var/opt/microsoft/dependency-agent/log/install.log` |

## Network Watcher Extension

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-linux](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-linux){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-windows](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-windows){target="_blank"}

[:octicons-link-external-16: https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-update](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/network-watcher-update){target="_blank"}

=== "Windows"
    | Name | Location |
    |---|---|
    | Extension name | `NetworkWatcherAgentWindows` |
    | Extension logs | `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.NetworkWatcher.NetworkWatcherAgentWindows\` |

=== "Linux"
    | Name | Location |
    |---|---|
    | Extension name | `NetworkWatcherAgentLinux` |
    | Extension logs | `/var/log/Microsoft/Azure/NetworkWatcherAgent/Logs` |
    

## Qualys VM extension

Please also see [:octicons-link-external-16: the article on how to deploy the Qualys extension with Azure Policy](qualys-azpolicy.md){target="_blank"}

=== "Windows"
    | Name | Location |
    |---|---|
    | Extension name | `QualysAgent` |
    | Agent logs | `C:\WindowsAzure\Logs\Qualys.QualysAgent\3.1.3.34\Asclog.txt` |
    | Agent installer | `C:\Packages\Plugins\Qualys.QualysAgent\3.1.3.34\` |

=== "Linux"
    | Name | Location |
    |---|---|
    | Extension name | `QualysAgentLinux` |
    | Agent logs | `/var/log/qualys/qualys-cloud-agent.log` |
    | Extension logs | `/var/log/azure/Qualys.QualysAgentLinux/lxagent.log` |
    | Install directory | `/usr/local/qualys/cloud-agent/bin` |
    | Install scripts | `/var/lib/waagent/Qualys.QualysAgentLinux-1.6.1.4/bin/avme_install.sh` |
    | Config files | `/etc/qualys/cloud-agent/` |

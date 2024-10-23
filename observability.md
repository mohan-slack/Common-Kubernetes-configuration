Overview
"VM Insights provides a quick and easy method for getting started with monitoring the client workloads on your virtual machines and virtual machine scale sets."

src.: https://learn.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-overview



In an ideal world this is happens as follows:




 
Virtual machines run the Azure Monitor Agent (AMA)
this is provisioned as an extension and governed by a policy
The agent gets instructions on what to collect from a Data Collection Rule
This rule specifies OS level data sources (counters, logs, events) and (optionally multiple) destination(s) (Azure Monitor, Log Analytics)
For the agent to receive instructions, the VM needs to be associated with the DCR
this is governed by a policy
The Log Analytics workspace is provisioned with the VM Insights extension to which data is being sent




Challenges
until recently the OMS (Log Analytics Agent on Linux, Microsoft Monitoring Agent on Windows) was used to manage this data stream
the agent is limited, only a single destination can be chosen
the agent is on a deprecation path and will be retired in August of 2024
migrating to the new agent sets has been difficult
Azure Portal VM Insights dashboards are laggy at best, unreliable for the most part
even when all conditions above are met (compliant with all policies, data demonstrably available in LAW) VMs may still be marked as "Not monitored"
it is therefore better not to rely on this dashboard but rather on the source data itself (Grafana: https://monitor.pru.intranet.asia/d/bfb55323-189f-46b1-9a48-1c8111edf364/, more details below)


Use Cases
Azure monitor alerting for virtual machines by default has access only to telemetry data visible from the hypervisor
this crucially does not include memory utilization, details such as swap file usage
the agents provide an insight into the operating system itself, it's resource management challenges and details regarding the running applications, processes
more granular insights into storage consumption (i.e. filesystem utilization) is available from the operating system only
with this data subsequently in Log Analytics, we can write alert rules to notify when anomalies occur
this process has been automated and is handled largely by Cloud Operations in self service


IAAS Monitoring Coverage - How can we tell if a VM is correctly onboarded
The following Grafana dashboard aims to provide this answer. The dashboard is intended to serve as a starting point and should continue to be developed in order to meet the requirements more precisely. For the time being it is split into three sections:







At the very top we find a selection of drop down fields populated with relevant data. Underneath we find the details we're interested it:

subscription level monitoring coverage
compliance details regarding the policies involved to manage VM Insights components
VM details to confirm, that a particular virtual machine is emitting data to LAW as expected






Subscription level overview provides a quick insight into the compliance data for the selected Policy Assignment. Each subscription globally has been set up with the same policy assignments.

Note: This is based on Azure resource graph and reflects a point in time. The view does not respond to the time range picker.



Example: you are concerned about a particular Linux virtual machine and want to ensure that it is configured correctly. You know the LBU and subscription of said VM. Here you get a brief glance at the picture across the subscription as a whole.







Compliance level details provide a bit more insight into VMs and resource groups. In the table on the left, VMs are grouped based on their compliance state against the select policy assignment. From the screenshot we can see 3 Linux VMs in the subscription to be non-compliant (reasons can be plentiful: the VM may not be powered on, agent installation may have failed etc.). The table on the right counts the number of VMs per resource group which have agents running. If the agent is correctly installed and running, it will send a Heartbeat every so often. 

Note: The compliance details on the left are based on Azure Resource Graph and do not respond to the time range picker widget. The table on the right however is based on live data in Log Analytics. If you choose a time range during which you know an agent was not running, than this will be reflected in the table.



Example: from here we can quickly and easily identify our Linux VM and ensure it is in the desired compliance state. Should the VM of choice not be in the left table than it means the selected policy is not associated with the VM. Reasons for this include but are not limited to unsupported operating systems (i.e. 3rd party appliances).





Finally under VM details we find the widgets which respond to the VM drop down field. To confirm that your VM of choice is correctly transmitting telemetry data to VM Insights you may select it from the list above.



Note: The list contains all virtual machines in the subscription and does not update based on the selected policy, OS type etc. The field and widgets in the bottom section of the dashboard are independent of the other sections.

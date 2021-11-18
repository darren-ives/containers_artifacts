///How many requests are coming to your cluster?
ContainerLog
| join kind=inner ContainerInventory on ContainerID
| where Image1 contains "ingress"
| summarize count() by bin(TimeOfCommand,1s)
| render timechart 

///How much memory is allocatable per node in your cluster?
Perf
| where ObjectName == "K8SNode" and CounterName=="memoryAllocatableBytes"
| summarize AvgMemoryAllocatableBytes = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName

///What is the CPU usage of your workload? What is the CPU usage of internal Kubernetes tools?
KubePodInventory
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(ControllerName, '/', tostring(split(ContainerName, '/')[1]))
| join kind=inner Perf on Computer, InstanceName
| where ObjectName == "K8SContainer" and CounterName == "cpuUsageNanoCores"
| summarize AvgCPUUsageNanoCores = avg(CounterValue) by bin(TimeGenerated, 30m), Namespace

///How many pods are currently pending?
KubePodInventory
| where Namespace == "api" or Namespace == "web"
| summarize arg_max(TimeGenerated, PodStatus) by Name

///Which pod is consuming the most memory?
KubePodInventory
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(ControllerName, '/', tostring(split(ContainerName, '/')[1]))
| join kind=inner Perf on Computer, InstanceName
| where ObjectName == "K8SContainer" and CounterName == "memoryWorkingSetBytes"
| summarize AvgCPUUsageNanoCores = arg_max(TimeGenerated, CounterValue) by Name
| order by CounterValue desc 
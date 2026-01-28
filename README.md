<h1>Highly Available Web App</h1>
In this project I designed a simple web app across a Virtual Machine Scale Set behind a load balancer. I wanted to touch on basic concepts of design, automation, high availability and elasticity using what I have learned from AZ-104 prep.  <br />


```mermaid
  graph TD
    %% Definitions of external components
    User(("Internet User\n(Browser)"))
    LB["Azure Public Load Balancer\n(Public IP & Health Probes)"]
    Monitor["Azure Monitor &\nAutoscale Engine"]

    %% VNet Boundary encompassing your infrastructure
    subgraph VNet ["Azure Virtual Network (VNet)"]
        subgraph Subnet ["Web Subnet & NSG (Allow Port 80)"]
            %% The main VMSS resource managing the instances
            VMSS["Virtual Machine Scale Set (VMSS)\n(Ubuntu 22.04 + Nginx)"]
            
            %% Visual representation of the instances
            subgraph Instances ["Backend Server Pool"]
                VM1["Server Instance_0\n('Hello from Simeon')"]
                VM2["Server Instance_1\n('Hello from Simeon')"]
                VM3["Server Instance_2\n(Created by Autoscale spike)"]
            end
        end
    end

    %% --- Traffic Flow Path ---
    User -- "HTTP Request" --> LB
    LB -- "Load Balances Traffic" --> VM1
    LB -- "Load Balances Traffic" --> VM2
    LB -.-> |"Traffic during high load"| VM3

    %% --- Autoscale Logic Path ---
    VM1 -.-> |"CPU Metrics"| Monitor
    VM2 -.-> |"CPU Metrics"| Monitor
    VM3 -.-> |"CPU Metrics"| Monitor
    
    Monitor == "Scale Out Rule\n(CPU > 70%)" ==> VMSS
    Monitor == "Scale In Rule\n(CPU < 30%)" ==> VMSS
    VMSS -- "Provisions / Deletes" --> VM3

    %% Styling for Azure-like look
    classDef azure fill:#0078d4,stroke:#ffffff,stroke-width:2px,color:#ffffff;
    class LB,VMSS,Monitor azure;
    style VNet fill:#f5f5f5,stroke:#666,stroke-width:2px,stroke-dasharray: 5 5;
    style Instances fill:#e6f7ff,stroke:#0078d4;
    style VM3 stroke-dasharray: 5 5;
```
<br />
<h2>Environments and Technologies Used</h2>

- Microsoft Azure
- Azure Load Balancer
- Virtual Machine Scale Sets (VMSS)
- Custom Script Extension (Linux)
- Azure Monitor & Insights
- Nginx Web Server
- Stress Utility

<h2>Operating Systems Used </h2>

- Ubuntu Server

<h2>Deployment and Configuration Steps</h2>

<p>
In this lab, I created a Virtual Machine Scale Set paired with an Azure Load Balancer. This setup ensures that if one server fails, the Load Balancer’s "Health Probes" will redirect traffic to healthy instances. Furthermore, I implemented Autoscale Rules that monitor CPU utilization, automatically adding or removing server capacity based on demand.

Configuring a highly available and elastic web app in Azure involves defining settings that control how traffic is distributed across multiple virtual machine instances and how the cluster reacts to performance demands.

I've learned a lot about business continuity and wanted to showcase how enterprises can ensure applications stay online and costs can be optimized.

I started by creating a Resource Group as a logical container for all resources.
</p>

<img width="633" height="456" alt="Pasted image 20260118072518" src="https://github.com/user-attachments/assets/7183bd8e-ae86-4606-8c49-81f293ccf8c1" />


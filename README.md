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

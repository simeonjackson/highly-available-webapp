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
- Nginx Web Server
- Azure Montior & Insights
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

<img width="633" height="456" alt="Image" src="https://github.com/user-attachments/assets/d0c71789-6e4b-459f-9beb-28d0dd9199b4" />
<img width="634" height="457" alt="Image" src="https://github.com/user-attachments/assets/d6c16102-5014-4d8f-b736-4777704752eb" />

<p>
  
I created a Virtual Network and a specific subnet for my web servers.

</p>

<img width="634" height="458" alt="Image" src="https://github.com/user-attachments/assets/d9b35fe1-ec91-41a1-9be4-cde37c2afe01" />
<p> 
  
IPv4 address space defined with a subnet `172.16.0.0/24`
</p>
<img width="634" height="458" alt="Image" src="https://github.com/user-attachments/assets/0580ebf4-b213-4e53-8448-a6e623577862" />
<img width="632" height="458" alt="Image" src="https://github.com/user-attachments/assets/9e3bf9c6-f53b-4e08-ac0e-e010608350a3" />


<p>
  
For my web app I configured a Virtual Machine Scale Set (VMSS) with an Ubuntu Server image and two instances. Since I want these machines to run the same workload, I choose `Uniform` for the Orchestration Mode.

</p>

<img width="635" height="458" alt="Image" src="https://github.com/user-attachments/assets/af0074c6-3492-4a54-8040-4131e670bc4a" />
<img width="635" height="457" alt="Image" src="https://github.com/user-attachments/assets/eb22c5d7-f6d0-4fa1-9c6e-65b8983208da" />
<img width="635" height="457" alt="Image" src="https://github.com/user-attachments/assets/22b02a5e-55fb-40d3-83e0-4ba4f52e7b6a" />

<p>
  
On the Networking tab, I chose the virtual network `tws_highlyavailable_rg` to keep all my resources in the same network and
I edited the network interface to choose the subnet I created when configuring the virtual network.

</p>

<img width="632" height="247" alt="Image" src="https://github.com/user-attachments/assets/45721a4b-370d-4cb5-82dc-19745b5892e4" />
<img width="935" height="464" alt="Image" src="https://github.com/user-attachments/assets/3160f2d0-a4b8-407b-9688-e9949fd92527" />

<p>
  
Here is the following result. The Overview page shows that I have two instances of my VM. Next, I will upload a script that will install the NGINX web server as well as some custom text representing our web app.

</p>

<img width="636" height="458" alt="Image" src="https://github.com/user-attachments/assets/ae15d4a0-f461-4668-ae7f-ab6c7035c81e" />


<p>
  
To do this I will navigated to Extensions and Applications under the Settings tab within the VMSS resource. The extension I am installing is called Custom Script for Linux.

</p>

<img width="635" height="458" alt="Image" src="https://github.com/user-attachments/assets/3f85442b-4d19-4bfb-a9ed-19020e334851" />
<img width="632" height="460" alt="Image" src="https://github.com/user-attachments/assets/c320ef82-3994-43e3-a12a-f736f73fe898" />

<p>
  
I prepared a text file that contains a script that will install NGINX as well as display text saying "Hello from Simeon" and list the VM instance that is running. To use this in Custom Script I uploaded a file called `tws_nginx.sh` into  a storage container. 

Browsing and choosing that file, then making sure the command matches my file name in this instance `sh tws_nginx.sh`, I'm telling Azure to download this script onto my VMs and execute it. This automates the installation of the web server on my VMs so I don't have to do it individually.

</p>

<img width="495" height="298" alt="Image" src="https://github.com/user-attachments/assets/87c55a81-faac-4734-8d4d-fce6e078fd51" />
<img width="938" height="463" alt="Image" src="https://github.com/user-attachments/assets/8ce540c0-246a-4b8d-a682-d5f7b4a20f3a" />

<p>

A few more steps before this is going to work. I need to make sure that our Network Security Group (NSG) is allowing inbound web traffic. We are installing NGINX web server on our VMs but currently we will not be able to access this server from the internet. So I created an inbound rule allowing port 80 (HTTP) traffic to our network.

</p>

<img width="932" height="460" alt="Image" src="https://github.com/user-attachments/assets/6ba0c100-94c8-4832-a838-ea3e5bc9280c" />
<img width="935" height="463" alt="Image" src="https://github.com/user-attachments/assets/f74d7f8e-037a-4cff-b9e2-2e9610d0582e" />


<p>

VMSS is currently set to update manually so to see the changes I made, I go to Instances and run the upgrade to the latest model.
 
</p>

<img width="941" height="464" alt="Image" src="https://github.com/user-attachments/assets/b8f01e8a-1833-4eb5-ab83-fe2d41b63efb" />


<p>
  
Finally I can test the public IP address and see that the server is running and I can access it via the web. Note the server name matches the computer name.

</p>

<img width="938" height="464" alt="Image" src="https://github.com/user-attachments/assets/e775b099-5d7d-4789-bc11-8adcb85e6ca8" />
<img width="958" height="513" alt="Image" src="https://github.com/user-attachments/assets/df2fd909-8e5a-4b55-92bd-744da4fd39a1" />

<p>
  
Currently, both instances of my VMSS can only be accessed if one has their public IP address. I want these VMs to run as a highly available, scalable set. In order to do this I will put the VMSS behind a load balancer. A load balancer will put the instances behind one public IP and will use a health probe to gauge and determine how to route traffic.

This ensures that if one of our instances goes offline, our web app is still accessible. This is a common staple in building a scalable architecture ensuring availability of resources no matter the circumstance.

I configured a load balancer on port 80.

</p>

<img width="923" height="424" alt="Image" src="https://github.com/user-attachments/assets/c64af0cc-be38-4f51-9d9d-e5e4cc0af8df" />
<img width="926" height="430" alt="Image" src="https://github.com/user-attachments/assets/57f70d35-3477-4973-a807-5548a6d24802" />


<p>
  
The frontend of the load balancer is the entry point. Traffic comes through the Frontend IP and is distributed to the backend pool.

</p>

<img width="928" height="424" alt="Image" src="https://github.com/user-attachments/assets/4667e6e1-54fc-4298-9db4-8d6e9f335cca" />


<p>
  
The backend pool is the collection of servers, VMs, or resources that receive and process incoming traffic from the load balancer.

</p>

<img width="940" height="461" alt="Image" src="https://github.com/user-attachments/assets/266eb286-f6de-4b22-823d-817d1013dac9" />

<p>
  
The health probe is looking at backend instances to see if they are "healthy" it does this by sending requests. If an instance in the pool is unhealthy, the load balancer will stop sending traffic there.

</p>

<img width="929" height="428" alt="Image" src="https://github.com/user-attachments/assets/91cb01c3-2f1e-40a7-9129-4af3f8c720c6" />

<p>
  
Lastly, the load balancing rules which is where you set all the components that I mentioned previously.

</p>

<img width="938" height="464" alt="Image" src="https://github.com/user-attachments/assets/02ca4676-01a7-48df-9053-bda43c5899a1" />

<p>

With the load balancer configured, I can use its Public IP in order to gain access to the web server.

</p>

<img width="937" height="463" alt="Image" src="https://github.com/user-attachments/assets/60c85cd2-5882-406b-ade7-f891d4a2db6f" />
<img width="958" height="515" alt="Image" src="https://github.com/user-attachments/assets/36f910ac-6a78-4cf5-bf23-92511570951e" />

<p>

The load balancer connected me to `webscale_0` (tws_highl000000). Now I am going to stop this machine which won't allow me access to this specific VM instance. However, since we have placed two instances in this scale set and both sit behind the load balancer, I should be able to refresh the page and still connect to the server.

</p>

<img width="938" height="469" alt="Image" src="https://github.com/user-attachments/assets/912256af-f697-4fb9-ab2f-0a82e4ff1ac3" />

<img width="958" height="513" alt="Image" src="https://github.com/user-attachments/assets/df2fd909-8e5a-4b55-92bd-744da4fd39a1" />

<p>

I wanted to add another component to this project adding an autoscale rule to add another server instance if the CPU hits  a certain threshold. This is a key component to efficient scaling of resources and something that is common in any business with a cloud environment.

I was able to set up my VMSS to scale up but due to the limitations of my Azure trial, my third VM failed.

Here is my walkthrough anyway.

Under the Scaling tab in my VMSS resource, their are two options to scale resources. Manual Scale keeps a fixed instace count, where Autoscale can be configured to scale on metrics.

</p>

<img width="932" height="465" alt="Image" src="https://github.com/user-attachments/assets/f453b178-253e-41db-b3ba-7f5150c26c28" />

<p>

I added a few rules that would trigger scaling up or down the number of VM instances available.

Adding an instance when CPU goes above 70% average over 5 minutes.

</p>

<img width="938" height="463" alt="Image" src="https://github.com/user-attachments/assets/0ee9337c-e9b1-408d-bae4-615d616f5ffc" />

<p>

And removing an instance when the CPU goes below 30% average over 5 minutes.

You can also see that I set Instance Limits. The minimum, maximum and default number of instances that will be running.

</p>

<img width="939" height="462" alt="Image" src="https://github.com/user-attachments/assets/1e90c697-e31b-4aae-b1ea-4eff30054ad6" />

<p>

In order to test my new scale settings, I ran a stress test on one of the instances. This sets the CPU at 100% and will trigger the autoscale rule to set in.

</p>

<img width="576" height="284" alt="Image" src="https://github.com/user-attachments/assets/be59c4b3-65b4-460e-97d9-dba3e94a8c0f" />

<img width="577" height="305" alt="Image" src="https://github.com/user-attachments/assets/a4db86f8-f3ef-45a9-878a-ebc74c524b35" />

<p>

Unfortunately this is where my lab ended. My Azure trial subscription would not allow my VMSS to provision the third VM when CPU hit my threshold set. The trial only allows for 3 public IPs which I already had.

I did take a screenshot of the activity log showing that the scale up was initiated. It also shows where the operation failed as it would put me over my VM quota.


</p>

<img width="939" height="464" alt="Image" src="https://github.com/user-attachments/assets/e16b4ea7-486e-4568-81ee-cbd72098ddb0" />

<img width="938" height="464" alt="Image" src="https://github.com/user-attachments/assets/71ae5e26-ade5-48c2-a208-dd485ddc384f" />


<p>

In a real environment I could request an increase in quota but for this lab I just wanted to get the basic point across.

Normally you would see rhe third instance provision and the load balancer would have another option to route traffic. When the average CPU drops below 30% again. The third instance would deprovision.

Thanks for taking your time to go through my lab!

-Simeon-

</p>




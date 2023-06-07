System Requirments & Reference Architecture



This guide describes recommended best practices for infrastructure architects and operators to follow when deploying Vault using the Integrated Storage (Raft) storage backend in a production environment.

This guide includes general guidance as well as specific recommendations for popular cloud infrastructure platforms. These recommendations have also been encoded into official Terraform modules for AWS, Azure, and GCP.

Recommended architecture
The following diagram shows the recommended architecture for deploying a single Vault cluster with maximum resiliency:
With five nodes in the Vault cluster distributed between three availability zones, this architecture can withstand the loss of two nodes from within the cluster or the loss of an entire availability zone.

If deploying to three availability zones is not possible, the same architecture may be used across two or one availability zones, at the expense of significant reliability risk in case of an availability zone outage.

System requirements
This section contains specific hardware capacity recommendations, network requirements, and additional infrastructure considerations. Since every hosting environment is different and every customer's Vault usage profile is different, these recommendations should only serve as a starting point from which each customer's operations staff may observe and adjust to meet the unique needs of each deployment.

Hardware sizing for Vault servers
Sizing recommendations have been divided into two common cluster sizes.

Small clusters would be appropriate for most initial production deployments or for development and testing environments.

Large clusters are production environments with a consistently high workload. That might be a large number of transactions, a large number of secrets, or a combination of the two.

Size	CPU	Memory	Disk Capacity	Disk IO	Disk Throughput
Small	2-4 core	8-16 GB RAM	100+ GB	3000+ IOPS	75+ MB/s
Large	4-8 core	32-64 GB RAM	200+ GB	10000+ IOPS	250+ MB/s

For each cluster size, the following table gives recommended hardware specs for each major cloud infrastructure provider.

Provider	Size	Instance/VM Types	Disk Volume Specs
AWS	Small	m5.large, m5.xlarge	100+GB gp3, 3000 IOPS, 125MB/s
Large	m5.2xlarge, m5.4xlarge	200+GB gp3, 10000 IOPS, 250MB/s
Azure	Small	Standard_D2s_v3, Standard_D4s_v3	1024GB* Premium_LRS
Large	Standard_D8s_v3, Standard_D16s_v3	1024GB* Premium_LRS
GCP	Small	n2-standard-2, n2-standard-4	500GB* pd-balanced
Large	n2-standard-8, n2-standard-16	1000GB* pd-ssd

Network connectivity
The following table outlines the network connectivity requirements for Vault cluster nodes. If general network egress is restricted, particular attention must be paid to granting outgoing access from the Vault servers to any external integration providers (for example, authentication and secret provider backends) as well as external log handlers, metrics collection, security and config management providers, and backup and restore systems.

Source	Destination	port	protocol	Direction	Purpose
Client machines	Load balancer	443	tcp	incoming	Request distribution
Load balancer	Vault servers	8200	tcp	incoming	Vault API
Vault servers	Vault servers	8200	tcp	bidirectional	Cluster bootstrapping
Vault servers	Vault servers	8201	tcp	bidirectional	Raft, replication, request forwarding
Vault servers	External systems	various	various	various	External APIs

Load balancer
A load balancer is a system that distributes network requests across multiple servers. It may be a managed service from a cloud provider, a physical network appliance, a piece of software, or a service discovery platform such as Consul.

Each major cloud provider offers one or more managed load balancing services:

Cloud	Layer	Managed Load Balancing Service
AWS	Layer 4	Network Load Balancer
Layer 7	Application Load Balancer
Azure	Layer 4	Azure Load Balancer
Layer 7	Azure Application Gateway
GCP	Layer 4/7	Cloud Load Balancing

Autoscaling
Autoscaling is the process of automatically scaling computational resources based on service activity. Autoscaling may be either horizontal, meaning to add more machines into the pool of resources, or vertical, meaning to increase the capacity of existing machines.

Each major cloud provider offers a managed autoscaling service:

Cloud	Managed Autoscaling Service
AWS	Auto Scaling Groups
Azure	Virtual Machine Scale Sets
GCP	Managed Instance Groups


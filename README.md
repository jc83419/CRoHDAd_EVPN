# Cumulus Routing on the Host Docker Advertisement daemon (CRoHDAd) - Ethernet Virtual Private Network (EVPN)
In this github project the Cumulus Routing on the Host Docker Advertisement daemon (CRoHDAd) was used in combination with Ethernet Virtual Private Network (EVPN) in order to advertise host (/32) routes of containers on one node to other nodes in a spine leaf data center environment. This was done through a VXLAN overlay between the nodes. Furthermore, the nodes were shared by multiple tenants. In order to allow for the sharing of resources, but maintain security and privacy of those tenants, a Virtual Routing and Forwarding (VRF) table was created per tenant. This VRF table contains only the routes of that tenant its containers. Meaning, the tenant does not know about other tenants and how to reach them. Meaning that no network policies are needed as done by other Kubernet Container Network Interfaces (CNIs) such as Calico or Cilium. Also a virtual bridge per tenant was used, to avoid L2 sniffing, since the other tenants could be potentially malicious. The created architecture for multi-tenancy with EVPN can be seen below.


![image](images/Architecture.png)


The experiments and results with EVPN as CNI on a Kubernetes environment can be found in [experiments](experiments). Whereas experiment 1 focuses on inter-host connectivity and the traffic flow and experiment 2 focuses on multi-tenancy and traffic isolation.

In order to recreate the experiments or create a similar environment, one should first install the Kubernetes Master and nodes as is for example done in [How to install Kubernetes on Ubuntu 18.04 Bionic Beaver Linux](https://linuxconfig.org/how-to-install-kubernetes-on-ubuntu-18-04-bionic-beaver-linux). The dependencies could also be installed from the [kube_install script](scripts/kube_install.sh).



When all the nodes are joined to the Kubernetes master, the bash [scripts](scripts) (kubernetes-startup-master and kubernetes-startup-node) should be run. These scripts create 3 bridges and configures them based on the default /24 per node allocation from Kubernetes. After the bridges are created, [CRoHDAd](crohdad) was startup. CRoHDAd is a privileged container that checks for Kubernetes/Docker events. When the Kubernetes Master schedules a container to the Kubelet agent on the node, the Kubelet agent creates the container with Docker. The creation or deletion of a container triggers an event, which the Cumulus Routing on the Host Docker Advertisement daemon (CRoHDAd) listens to. The original CRoHDAd checks for the IP address of the container and adds or deletes the host route to a default routing table. We adjusted the CRoHDAd, such that based on the labels provided in the Kubernetes deployment of the container, CRoHDAd adds or deletes the host route to the corresponding VRF table. When the host routes are added to the tenant VRF, they get imported to FRR and advertised by BGP-EVPN. The process is shown in the figure below.

![image](images/Crohdad.png)

After the kubernetes Master and nodes are setup, and CRoHDAd is running. The containers can be deployed. For the containers, the YAML files in [deployments](deployments) were used. The deployments of exp1 were used for experiment1 and the deployments of exp2 were used for experiment2. However, one can choose to deploy its own YAML files for the containers.

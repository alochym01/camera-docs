# **Table Of Contents**

## **1.** **Introducing Docker and Kubernetes**  
    1.1 Understanding Docker Containers and Images  
      1.1.1 Building your own Docker images  
      1.1.2 Sharing images with other Registries  
      1.1.3 Using Docker Volumes for persistent storage  
      1.1.4 Understanding how Docker runs containers  
      1.1.5 Run multi-container apps with Docker Compose  
    1.2 Introducing Kubernetes  
      1.2.1 Understanding Kubernetes Architecture  
      1.2.2 Running an application in Kubernetes  
      1.2.3 Understanding the benifits of using Kubernetes  

## **2.** **Core concepts**  
    2.1 Pods: running containers in Kubernetes  
      2.1.1 Introducing pods  
      2.1.2 Creating pods from YAML or JSON descriptiors  
      2.1.3 Orginizing Pods with labes  
      2.1.4 Listing subsets of Pods through label selectors  
      2.1.5 Using labels and selectors to constrain Pod  
      2.1.6 Annotating Pods  
      2.1.7 Using namespace to group resources  
      2.1.8 Stopping and removing Pods  
    2.2 Replication and other controllers: deploying managed pods  
      2.2.1 Keeping Pods healthy  
      2.2.2 Introducing Replication Controllers  
      2.2.3 Using ReplicaSets instead of Replication Controllers  
      2.2.4 Running exactly one Pod on each Pod with DaemonSets  
      2.2.5 Running Pods that perform a single completable task  
      2.2.6 Scheduling Jobs to run periodically or once in the future  
    2.3 Services: enabling clients to discover and talk to Pods  
      2.3.1 Introducing services  
      2.3.2 Connecting to services living outside the cluster  
      2.3.3 Exposing services to external clients  
      2.3.4 Exposing services externally through an Ingress resource  
      2.3.5 Signaling when a pod is ready is accept connections  
      2.3.6 Using a headless service for discovering individual Pods  
      2.3.7.Troubleshooting services  
    2.4 Volumes: attaching disk storage to containers  
      2.4.1 Introducing volumes  
      2.4.2 Using volumes to share data between containers  
      2.4.3 Accessing files on the worker node's Filesystem  
      2.4.4 Using persistent storage  
      2.4.5 Decoupling Pods from the underlying storage technology  
      2.4.6 Dynamic provisioning of PersistentVolumes  
    2.5 ConfigMaps and Secrets: configuring applications    
      2.5.1 Configuring containerized applications   
      2.5.2 Passing command-line arguments to containers  
      2.5.3 Setting environment variables for a container  
      2.5.4 Decoupling configuration with a ConfigMap  
      2.5.5 Using Secrets to pass sensitive data to containers  
    2.6 Accessing Pod metadata and other resource from applications  
      2.6.1 Passing metadata through the Downward API  
      2.6.2 Talking to the Kubernetes API Server  
    2.7 Deployments: updating applications declaratively  
      2.7.1 Updating applications running in Pods  
      2.7.2 Performing an automatic rolling update with    ReplicationController  
      2.7.3 Using Deployments for updating apps declaratively  
    2.8 StatefulSets: deploying replicated stateful applications  
      2.8.1 Replicating stateful pods  
      2.8.2 Understanding StatefulSets  
      2.8.3 Using a StatefulSet  
      2.8.4 Discovering peers in a StatefulSet  
      2.8.5 Understanding how StatefulSets deal with node failures  
## **3.** **Advanced Kubernetes**  
    3.1 Networking  
      3.1.1 Communication between containers in the same pod  
      3.1.2 Communication between pods on the same node  
      3.1.3 Communication between pods on different nodes  
      3.1.4 Communication between pods and services  
      3.1.5 How does DNS work? How do we discover IP addresses?  
    3.2 Securing the Kubernetes API server  
      3.2.1.Understanding authentication  
      3.2.2 Securing the cluster with role-based access control  
    3.3 Securing cluster nodes and the network  
      3.3.1 Using the host node's namespaces in a pod  
      3.3.2 Configuring the container's security context  
      3.3.3 Restricting the use of security-related features  
      3.3.4 Isolating the pod network  
    3.4 Managing pod's computational resources  
      3.4.1 Requesting resources for a pod's containers  
      3.4.2 Limiting resources available to a container  
      3.4.3 Understanding pod QoS classes  
      3.4.4 Namespaces  
      3.4.5 Setting default requests and limits for pods per namespace  
      3.4.6 Limiting the total resources available in a namespace  
      3.4.7 Monitoring pod resource usage  
    3.5 Automatic Scaling of pods and cluster nodes  
      3.5.1 Horizontal pod autoscaling  
      3.5.2 Vertical pod autoscaling  
      3.5.3 Horizontal scaling of cluster nodes  
    3.6 Advanced scheduling  
      3.6.1 Using taints and tolerations to repel pods from certain nodes  
      3.6.2 Using node affinity to attract pods to certain nodes  
      3.6.3 Co-locating pods with pod affinity and anti-affinity  
    3.7 Monitoring and Logging  
      3.7.1 Accessing the Logs of a Container  
      3.7.2 Working with Kubernetes logs  
      3.7.3 Working with etcd log  
      3.7.4 Monitoring master and node  
    3.8 Extending Kubernetes  
      3.8.1 Defining custom API objects  
      3.8.2 Extending Kubernetes with the Kubernetes Service Catalog  
      3.8.3 Platforms built on top of Kubernetes  
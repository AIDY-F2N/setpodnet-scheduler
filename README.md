# setpodnet-scheduler: Latency and Bandwidth-Aware scheduler for QoS-Sensitive Applications Using a Reinforcement Learning-Based Scheduler with Kubernetes 

<div align="center">
    <img src="figures/1_IconsAll_Hori.png" alt="AIDY-F2N">
</div>

## Description
Coscheduling refers to the ability to schedule a group of pods at once, as opposed to the default Kubernetes behavior that schedules pods one-by-one.
'setpodnet-scheduler' is a custom Kubernetes scheduler designed to optimize the deployment of multi-pod applications by addressing the limitations of Kubernetes’ default scheduling approach. Unlike the default scheduler, which deploys pods independently, setpodnet-scheduler considers latency and bandwidth constraints between nodes and prioritizes co-locating connected pods on the node. This approach improves resource efficiency, reduces inter-pod latency, and enhances overall cluster performance and resource management, especially for applications with high inter-pod communication demands.
## Contributor

- Massinissa AIT ABA, massinissa.ait-aba@davidson.fr
- Abdenour Yasser BRAHMI, abdenour-yasser.brahmi@telecom-sudparis.eu

## Table of Contents

- [setpodnet-scheduler](#setpodnet-scheduler)
- [Deploy setpodnet-scheduler on a kubernetes cluster](#Deploy-setpodnet-scheduler-on-a-kubernetes-cluster)
- [Use setpodnet-scheduler to deploy K8s applications](#Use-setpodnet-scheduler-to-deploy-K8s-applications)
- [Example1: Illustrating Deployment of setpodnet-scheduler in a Kind Cluster and Application Deployment](#Example1-Illustrating-Deployment-of-setpodnet-scheduler-in-a-Kind-Cluster-and-Application-Deployment)
- [Accessing the setpodnet-scheduler Pod for Debugging](#Accessing-the-setpodnet-scheduler-Pod-for-Debugging)
- [Discussion](#Discussion)


## setpodnet-scheduler

K8s uses a scheduling framework that allows different scheduling algorithms to be implemented and used with the Kube-scheduler component. Unlike the default scheduler, setpodnet-scheduler first checks whether it is possible to find a feasible assignment of the pods to the cluster nodes respecting resources and labels constraints. Only if such an assignment is found, the scheduler starts the deployment of the pods composing the application sequentially following a given order. Otherwise, it rejects the application and no pod is deployed.

## Deploy setpodnet-scheduler on a kubernetes cluster
To deploy setpodnet-scheduler, run the following command:
```bash[language=bash]
kubectl apply -f setpodnet-scheduler.yaml
```
## Use setpodnet-scheduler to deploy K8s applications 
To use setpodnet-scheduler, specific parameters need to be added within the context of each pod in the application. To utilize the setpodnet-scheduler for your application's pods, ensure to include the following configuration within the `spec` section of each pod:

```yaml
spec:
  schedulerName: setpodnet-scheduler
```

Furthermore, make sure to include the following configurations within the pod's metadata section:

```yaml
metadata:
  labels:
    id: "1"
    app_ad: "1" 
    nb_pods: "3" 
    update: "True"
    timelimit: "10"
  spec:
    schedulerName: setpodnet-scheduler
  annotations:
    communication-with: "pod2,pod3"
    latency-pod2: "1"
    bandwidth-pod2: "5"
    latency-pod3: "10"
    bandwidth-pod3: "10"
```

Here's an explanation of each label:

- 'id': This label signifies an identifier that uniquely distinguishes the pod within the context of its application. It helps differentiate between multiple pods belonging to the same application. Additionally, in the context of deployment sequencing, this identifier establishes the order in which pods are deployed. Pods will be deployed in numerical order according to their assigned IDs. If "id=1" is used for all pods, they will be deployed in parallel.

- 'app_ad': The 'app_ad' label serves as an application differentiator, enabling the identification and grouping of pods belonging to different applications. It aids in categorizing pods based on their application affiliation.

- 'nb_pods': This label indicates the total number of pods associated with the specific application. It provides information about the quantity of pods that are part of the same application.

- 'update': The 'update' label determines whether the scheduler should measure latency and bandwidth before each application deployment. By default, this value is set to "True".

- 'timelimit': The 'timelimit' label, when provided, specifies the maximum allowed time for the scheduling algorithm to find a suitable deployment or assignment solution for all the pods of the application. Adjust this value based on the time allocated for the scheduling algorithm to make optimal pod assignments.

- 'communication-with': Specifies the target pods with which communication is required (e.g., "pod2" and "pod3" in this example).

- 'latency-"name_of_the_pod"': Defines the latency requirement for communication with the specified pod ("pod2" and "pod3" in this example) on a scale from 1 to 10, where 1 represents low latency importance and 10 indicates a high priority for low latency.

- 'bandwidth-"name_of_the_pod"': Sets the bandwidth requirement for communication with the specified pod ("pod2" and "pod3" in this example) on a scale from 1 to 10, where 1 means minimal bandwidth needs and 10 represents a high demand for bandwidth.

## Example1: Illustrating Deployment of setpodnet-scheduler in a Kind Cluster and Application Deployment

In this example, we'll demonstrate the process of:

- Creating a Kind cluster configuration with a specific node setup.
- Deploying the setpodnet-scheduler into the newly created Kind cluster.
- Deploying an application consisting of three pods within this cluster, utilizing the setpodnet-scheduler for pod management.

1. If you already have a cluster, you can skip this step. Begin by setting up a YAML configuration file (`cluster-config.yaml`) for the Kind cluster, specifying the required node roles.

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
```
To create a Kind cluster consisting of one control-plane and one worker node, use the following command:
```bash[language=bash]
kind create cluster --name my-k8s-cluster --config kind-config.yaml
```



2. Deploy the setpodnet-scheduler within the newly established Kind cluster to enable its utilization for pod scheduling and management.
```bash[language=bash]
kubectl apply -f setpodnet-scheduler.yaml
```



3. Deploy the 'threepods-example' application by deploying the three pods 'testPod1.yaml', 'testPod2.yaml' and 'testPod3.yaml' located in the 'example1' folder using the following three commands:

```bash[language=bash]
kubectl apply -f example1/testPod1.yaml
```

Then:

```bash[language=bash]
kubectl apply -f example1/testPod2.yaml
```

Then:

```bash[language=bash]
kubectl apply -f example1/testPod3.yaml
```
## Accessing the setpodnet-scheduler Pod for Debugging

If you need to troubleshoot the setpodnet-scheduler pod in your Kubernetes cluster, you can access it using the following command:

```bash
kubectl exec -it $(kubectl get pods --all-namespaces | awk '/setpodnet-scheduler/{print $2}') -n kube-system -- /bin/bash
```
Once inside the pod, you can view the logs by running:

```bash
cat std.log
```
This will display the logs of the setpodnet-scheduler pod


## Discussion 
setpodnet-scheduler is not part of the default Kubernetes installation. However, it can be configured and activated separately. Using setpod-schedule requires modification of the manifest of each pod in the application. it can be used to deploy different types of worklods (Deployment, StatefulSet, ...).  In
addition, since setpodnet-scheduler works independently of the default Kubernetes scheduler,
it is important to ensure that the use of setpodnet-scheduler has no impact on other types of workloads that
do not use it. If you're interested in exploring the source code further or have any questions, please feel free to contact me directly.

To delete the Kind cluster, use the following command:
```bash[language=bash]
kind delete cluster --name my-k8s-cluster
```

# Optimizing Kubernetes Resource Allocation with VPA Autoscaler

## VPA Autoscaler Overview

The **Vertical Pod Autoscaler (VPA)** is a Kubernetes component that automatically adjusts the resource requests for pods to improve resource utilization and performance. It helps in optimizing resource allocation based on actual usage using metrics provided by Prometheus or metrics-server. **Prometheus** offers more accurate metrics for this purpose.

The VPA uses historical pod usage data (over the last 8 days) to calculate the average usage and sets recommendations for each pod accordingly.

### Benefits of Using VPA:
- 🚀 **Eliminates Resource Inefficiencies**:
  - Pending pods causing delays
  - Bottlenecks that block teams from progress
  - Overtime work to resolve resource issues

## Installing Metrics Server via Helm

1. Add the Metrics Server Helm repository:
    ```bash
    helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
    ```

2. Install Metrics Server using Helm:
    ```bash
    helm install my-metrics-server metrics-server/metrics-server --version 3.12.1
    ```

## Installing VPA via Helm

1. Add the VPA Helm repository:
    ```bash
    helm repo add fairwinds-stable https://charts.fairwinds.com/stable
    ```

2. Install VPA using Helm:
    ```bash
    helm install my-vpa fairwinds-stable/vpa --version 4.5.0
    ```
![image](https://github.com/user-attachments/assets/62504f45-fdb1-4d2b-b665-6a720f60dbe7)

## VPA Components

Let's take a deep dive into the components of VPA, which are crucial when configuring VPA.

### 1. VPA Recommender
- **Function**: The central component responsible for analyzing the resource usage (CPU and memory) of pods in a Kubernetes cluster. It collects usage data from the metric server and tracks events such as out-of-memory errors or evictions due to memory pressure. Based on this data, it generates recommendations for the resource requests of the pods. This component is the "brain" of VPA.
- **Algorithm**: Utilizes a percentile-based approach, similar to Google's Borg infrastructure. It collects a histogram of usage samples over the last eight days and recommends a high percentile as the target resource usage.
<img src="https://github.com/user-attachments/assets/b21978b7-09ff-49f0-94fb-e7c42784abdb" width="300px" />
### 2. VPA Admission Controller
- **Function**: Integrates with Kubernetes' pod creation process via a mutating admission webhook. When a new pod is created, the plugin modifies the pod’s resource requests based on the recommendations from the VPA Recommender.

### 3. VPA Updater
- **Function**: Monitors running pods and evicts/restarts them if their resource requests significantly deviate from the recommended values. It uses Kubernetes' eviction API and considers pod disruption budgets to minimize impact on workloads.
- **Disruption Handling**: Respects custom shutdown periods (graceful termination) and pod disruption budgets to control the rate at which pods are evicted and restarted.

## VPA Modes

VPA operates in different modes:

- **Off Mode**: Generates recommendations without applying them to running pods.
- **Recreate Mode**: Updates resource requests only when new pods are created. Managed by the VPA Admission Controller.
- **Auto Mode**: Actively manages pod resources, restarting pods when necessary to apply new recommendations. Managed by the VPA Updater.

## VPA Object (Custom Resource Definition - CRD)
<img src="https://github.com/user-attachments/assets/bee760eb-432f-4b7f-adfb-d6ffe86fe715" width="300px" />

### Key Sections:

- **Spec Section**: Defines what and how to scale, including update and resource policies (e.g., specifying CPU and memory limits).
- **Status Section**: Contains recommendations for the resource requests (target, lower, and upper bounds) for each container in the scaled pods. The lower and upper bounds serve as confidence indicators for the recommendations, which appear only when you apply the VPA object.
<img src="https://github.com/user-attachments/assets/8806414d-98cb-4ec0-b140-d09f1df43a06" width="300px" />

## Streamlining VPA Object Creation with Goldilocks

The **Goldilocks** tool simplifies the creation and management of VPA objects.

1. **Installation**:
    ```bash
    helm install goldilocks --namespace goldilocks fairwinds-stable/goldilocks --create-namespace
    ```

2. **Enable Namespace**:
    Pick an application namespace and label it to view data:
    ```bash
    kubectl label ns goldilocks goldilocks.fairwinds.com/enabled=true
    ```

3. **View VPA Objects**:
    After labeling the namespace, VPA objects should start appearing. To enable automatic updates for VPA objects in the default namespace, use:
    ```bash
    kubectl label ns default goldilocks.fairwinds.com/vpa-update-mode="AUTO"
    ```

## Further Learning

### Practical References:
- [Optimizing Kubernetes with VPA - YouTube](https://www.youtube.com/watch?v=3h-vDDTZrm8)
- [VPA Autoscaler in Action - YouTube](https://www.youtube.com/watch?v=jcHQ5SKKTLM)

### General Introduction:
- [Understanding Kubernetes Resource Allocation - YouTube](https://www.youtube.com/watch?v=Y4vnYaqhS74)
- [VPA Documentation - GitHub](https://github.com/FairwindsOps/charts/tree/master/stable/vpa)
- [Goldilocks Documentation](https://goldilocks.docs.fairwinds.com/installation/#installation-2)

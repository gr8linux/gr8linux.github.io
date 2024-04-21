---
layout: page
title: K3S GPU & AI on Kuberntes
permalink: /k3s-gpu/
---


# Setting Up an Nvidia GPU Node on k3s with Nvidia GPU Operator

This guide provides detailed steps on how to configure and run an Nvidia GPU node in a k3s cluster using the Nvidia GPU Operator. This setup allows you to leverage GPU resources for Kubernetes workloads.

## Prerequisites

- **k3s Cluster**: You must have a k3s cluster already running. If not, you can install it following the official [k3s installation guide](https://rancher.com/docs/k3s/latest/en/installation/).
- **Nvidia GPU**: Ensure that the node has an Nvidia GPU installed.
- **Nvidia Drivers**: Nvidia drivers should be installed on the node where the GPU is located.

## Step 1: Install the Nvidia GPU Operator

The Nvidia GPU Operator simplifies the management of GPU devices in Kubernetes environments.

1. **Add the Nvidia GPU Operator repository to Helm**:
   ```bash
   helm repo add nvidia https://nvidia.github.io/gpu-operator
   helm repo update
   ```

2. **Install the Nvidia GPU Operator using Helm**:
   ```bash
   helm install --wait --generate-name nvidia/gpu-operator
   ```

## Step 2: Configure k3s to Use the Nvidia GPU

To enable k3s to use the Nvidia GPU, some additional settings are required:

1. **Label the GPU node**:
   Label your GPU node so that the Nvidia GPU Operator can target it correctly.
   ```bash
   kubectl label node <your-gpu-node-name> nvidia.com/gpu=true
   ```

2. **Check the installation**:
   After labeling the node, check if the Nvidia GPU Operator has deployed the components successfully.
   ```bash
   kubectl get pods -n gpu-operator-resources
   ```

## Step 3: Deploy a Test Application

To verify that everything is set up correctly, deploy a test application that utilizes the GPU.

1. **Create a test deployment file** (`cuda-test.yaml`):
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: cuda-vector-add
   spec:
     containers:
       - name: cuda-vector-add
         image: nvidia/cuda:11.0-base
         command: ["sh", "-c", "curl -s https://raw.githubusercontent.com/NVIDIA/cuda-samples/master/Samples/vectorAdd/vectorAdd.cu -o /tmp/vectorAdd.cu && nvcc /tmp/vectorAdd.cu -o /tmp/vectorAdd && /tmp/vectorAdd"]
         resources:
           limits:
             nvidia.com/gpu: 1
   ```

2. **Deploy the application**:
   ```bash
   kubectl apply -f cuda-test.yaml
   ```

3. **Check the application output**:
   Verify that the application is using the GPU by checking its logs.
   ```bash
   kubectl logs cuda-vector-add
   ```

## Conclusion

You have successfully configured a k3s cluster to utilize an Nvidia GPU using the Nvidia GPU Operator. This setup allows for the efficient deployment of GPU-accelerated applications in a Kubernetes environment.
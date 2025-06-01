# Deployment documentation

This document details the deployment structure and data flow in our application. After reading this, you should be able to understand the overall design of our application to the point where you could contribute to a design discussion with us :)

# Table of contents
[TODO]

# Deployment overview
- six repos, of which two containers, three libraries, and one containing mostly configs (operation, this one)

## Kubernetes deployment structure
- Custom Kubernetes cluster provisioned through Vagrant and Ansible
- Application is deployed into this cluster with a [Helm chart](https://github.com/remla25-team12/operation/tree/main/helm/myapp) 
- Additional tools: Prometheus, Grafana, Istio (see next section)

[WIP: a visualization that shows how the containers and other k8s resources are linked to each other.]

## Data flow for incoming requests
We use an Istio service mesh for dynamic traffic routing and distribution. Its deployment configuration is defined separately in the file [istio.yaml](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/istio.yaml). See also the figure below.

- The Istio Gateway allows requests for the host `myapp.local` to enter the Istio service mesh.
- The DestinationRules `*-model-dr` and `*-app-dr` define subsets of model-service and app, respectively, for versioned routing.
- The VirtualService `*-model-vs` defines the routing rules. Currently, 90% of incoming requests are routed to the primary version (v1) of the application, while the remaining 10% is directed to a canary version (v2). Sticky sessions are implemented using the `x-newvers` HTTP header, which ensures that once a user is assigned a specific version, subsequent requests from the same user continue to be directed to that version.
    > The 90/10 split is variable and can be set in [Values.yaml](https://github.com/remla25-team12/operation/blob/main/helm/myapp/values.yaml) by editing `istio.modelTrafficSplit.v1Weight` and `istio.modelTrafficSplit.v2Weight`. 

- The VirtualService `*-app-vs` [TODO ???]


[TODO Figure]


## ML Pipeline
- Model training is supported by DVC
- Consists of four stages
![Visualization of the training pipeline for the restaurant sentiment analysis model.](imgs/ML_pipeline.drawio.png)\
Figure x: Visualization of the training pipeline for the restaurant sentiment analysis model.
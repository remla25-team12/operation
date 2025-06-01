# Deployment documentation

This document details the deployment structure and data flow in our application. After reading this, you should be able to understand the overall design of our application to the point where you could contribute to a design discussion with us :)

# Table of contents


# Overall design
- six repos, of which two containers, three libraries, and one containing mostly configs (operation, this one)

## Kubernetes cluster
- Custom Kubernetes cluster provisioned through Vagrant and Ansible
- Application is deployed into this cluster with a Helm chart [link to helm chart folder in operation]
- 

[In progress: a flowchart/graphic showing how the containers are linked to each other]

## Dynamic routing
- Istio

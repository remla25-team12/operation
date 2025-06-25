# Operation

REMLA Group 12

<details>
  <summary><b>Table of Contents</b></summary>
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents** _generated with [DocToc](https://github.com/thlorenz/doctoc)_

- [Operation](#operation)
- [About the Project](#about-the-project)
  - [Relevant Repositories](#relevant-repositories)
- [Getting Started](#getting-started)
  - [Deployment with Docker](#deployment-with-docker)
    - [Prerequisites](#prerequisites)
    - [Install and Run](#install-and-run)
  - [Provisioning the Kubernetes Cluster](#provisioning-the-kubernetes-cluster)
    - [Prerequisites](#prerequisites-1)
    - [Install and Run](#install-and-run-1)
  - [Deployment on Kubernetes Cluster](#deployment-on-kubernetes-cluster)
    - [Prerequisites](#prerequisites-2)
    - [Install and run](#install-and-run-2)
- [Usage](#usage)
  - [Webapp access](#webapp-access)
  - [Sticky sessions](#sticky-sessions)
  - [Prometheus, Grafana and Alert Manager](#prometheus-grafana-and-alert-manager)
    - [Prometheus](#prometheus)
    - [Grafana](#grafana)
    - [Alert Manager](#alert-manager)
- [Troubleshooting](#troubleshooting)
- [Final State of our Assignment](#final-state-of-our-assignment)
- [Continuous Progress Log](#continuous-progress-log)
  - [Assignment 1](#assignment-1)
  - [Assignment 2](#assignment-2)
  - [Assignment 3](#assignment-3)
  - [Assignment 4](#assignment-4)
  - [Assignment 5](#assignment-5)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

</details>

# About the Project

This is an adaptation and application of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project.
This repository serves as the central point of the project, containing the Docker and Kubernetes deployment configurations, installation instructions, and links to other relevant components and repositories.

## Relevant Repositories

| Repository                                                         | Purpose                                                                                   |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [model-training](https://github.com/remla25-team12/model-training) | Training pipeline for the restaurant sentiment analysis model.                            |
| [model-service](https://github.com/remla25-team12/model-service)   | Model wrapper for communication with other components of the project, e.g. for inference. |
| [lib-ml](https://github.com/remla25-team12/lib-ml)                 | Preprocessing logic for input data, used during model training and inference.             |
| [lib-version](https://github.com/remla25-team12/lib-version)       | A simple version-aware libary that reports its own version.                               |
| [app](https://github.com/remla25-team12/app)                       | Webapp (frontend + service) to interface with the model.                                  |

---

# Getting Started

## Deployment with Docker

### Prerequisites

- Linux or macOS (recommended host operating system)
- [Docker Compose](https://docs.docker.com/compose/install/) (included with Docker Desktop)

### Install and Run

1. Clone this repository and navigate into the root folder.

   ```bash
   git clone https://github.com/remla25-team12/operation.git
   cd operation
   ```

2. Deploy the project using Docker Compose by running this command in the project's root folder.

   ```bash
   docker compose up -d
   ```

3. Navigate to http://localhost:8080 to access the application homepage.
4. When you're done, stop the running containers and clean up resources.
   ```bash
   docker compose down
   ```

---

## Provisioning the Kubernetes Cluster

### Prerequisites

- macOS or Linux (host operating system)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://developer.hashicorp.com/vagrant/install) and [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (provisioning tools)
- kubectl (for controlling the cluster from the host)

### Install and Run

1. Clone this repository:

   ```bash
    git clone https://github.com/remla25-team12/operation.git
   ```

2. Navigate into the repository's root folder and install the required Ansible collections:

   ```bash
    cd operation
    ansible-galaxy collection install -r requirements.yml
   ```

3. Before continuing, you may want to reset the host-only networks in VirtualBox. For some users, this prevents conflicts with stale or broken network configurations from previous VirtualBox setups.
   To do this, **open the VirtualBox GUI**, go to `Tools > Network`, and **remove any existing "Host-only Networks"** listed under that section. The list should now be empty:
   ![Empty host-only adapter list](imgs/vb_empty.png)

4. For convenient SSH access to the VMs in later steps, add your public SSH key (ends in`.pub`) to the `./provisioning/keys` folder:

   ```bash
    cp ~/.ssh/<your_key>.pub provisioning/keys/<your_key>.pub
   ```

5. Start the provisioning process from the repository's root folder. This operation may take up to 5 minutes to complete.

   ```bash
   vagrant up --provision
   ```

6. Once the VMs are up and provisioned, run the following Ansible playbook to finalize the Kubernetes setup:

   ```bash
    ansible-playbook -u vagrant -i 192.168.56.100, provisioning/finalization.yml \
    --private-key=.vagrant/machines/ctrl/virtualbox/private_key
   ```

7. To access the Kubernetes dashboard, do the following **on your host machine**:

   - Add `192.168.56.91 dashboard.local` to your `/etc/hosts` file. For example, use the following command:

     ```bash
     sudo sh -c 'echo "192.168.56.91 dashboard.local >> /etc/hosts'
     ```

   - Navigate to https://dashboard.local (note the https) and enter the token displayed in the terminal.

     ![Token as shown in the terminal](imgs/terminal_token.png)

     > **Note**: The token was generated during the final provisioning step, just before Istio was installed. If you cannot find the token in the terminal output anymore, run `vagrant ssh ctrl`, followed by `kubectl -n kubernetes-dashboard create token admin-user` to generate a new one.

8. To communicate with the cluster from the host, a kubeconfig file (`admin.conf`) has been exported by Ansible. For example, you can run:
   ```bash
   kubectl get ns --kubeconfig ./provisioning/admin.conf
   ```
   You can also set the filepath as an environment variable or add it to `~/.bashrc`, so that you do not need to use the `--kubeconfig` flag every time:
   ```bash
   export KUBECONFIG="./provisioning/admin.conf"
   ```

---

## Deployment on Kubernetes Cluster

### Prerequisites

- macOS or Linux (host operating system)
- [Helm 3 CLI](https://helm.sh/docs/intro/install/)
- [Istioctl](https://istio.io/latest/docs/setup/install/istioctl/) 1.25.2 or higher
- A functional Kubernetes cluster.
  - Recommended: VM cluster from the [Provisioning the Kubernetes Cluster](#provisioning-the-kubernetes-cluster) section. In the instructions below, it is assumed you already have this cluster up and running.
  - Alternatively, you can install and use [Minikube](https://minikube.sigs.k8s.io/docs/start/) for a local Kubernetes cluster.
    > **Note**: Minikube compatibility is not actively being prioritized and maintained by us. If anything does not work as expected, please use the VM cluster.

### Install and run

**Unless specified otherwise, all commands in this "Install and run" section are applicable to both Minikube and VM clusters.**

1. Clone this repository and navigate into the root folder (if you haven't done so already):

   ```shell
   git clone https://github.com/remla25-team12/operation.git
   cd operation
   ```

2. Prepare the cluster for app installation.

   1. For the **Kubernetes VM cluster**, most settings and configurations have already been applied during provisioning. Simply SSH into the control node and navigate to the shared folder directory. **This means that all subsequent commands in this section should be executed on the control node.**

      ```shell
      vagrant ssh ctrl
      $ cd /mnt/shared/
      ```

   2. For **Minikube**, additional work is required. Clean up any pevious Minikube instance, launch a new instance, enable ingresses, and install Istio manually with its Helm Charts:

      ```shell
      minikube delete
      minikube start --memory=4096 --cpus=4 --driver=docker
      minikube addons enable ingress
      helm repo add istio https://istio-release.storage.googleapis.com/charts
      helm repo update
      helm install istio-base istio/base -n istio-system --create-namespace
      helm install istiod istio/istiod -n istio-system
      helm install istio-ingress istio/gateway -n istio-system
      ```

      > **Note:** If you are using Fedora, you may need to run `sudo setenforce 0` first to allow Minikube to use the Docker driver.

3. Enable Istio sidecar injection in the default namespace:

   ```shell
   kubectl label namespace default istio-injection=enabled
   ```

4. Install and deploy the Prometheus stack in the istio-system namespace with custom Alertmanager configuration (defined in `helm/myapp/values-myprom.yaml`)

   ```bash
   helm repo add prom-repo https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install myprom prom-repo/kube-prometheus-stack -n istio-system -f ./helm/myapp/values-myprom.yaml
   ```

5. Install and deploy our application:

   ```shell
   export ENCRYPTED_SMTP_PASSWORD="c2V3dSB5cGNqIGJscmYgaG9uYg=="

   #  VM Cluster (make sure you are inside /mnt/shared):
   helm install myapp-dev ./helm/myapp --set smtp.encodedPassword=$ENCRYPTED_SMTP_PASSWORD

   # Minikube (disable VM shared folder):
   helm install myapp-dev ./helm/myapp --set useHostPathSharedFolder=false --set smtp.encodedPassword=$ENCRYPTED_SMTP_PASSWORD
   ```

6. If you make changes to the Helm chart or want to update the deployment, use the following command:
   ```bash
   helm upgrade --install myapp-dev ./helm/myapp
   ```

# Usage

**All commands in this "Usage" section should be executed on your host machine.**

**Add kubectl to your PATH if needed, see [Provisioning the Kubernetes Cluster, step 8](#provisioning-the-kubernetes-cluster).**

## Webapp access

To access the deployed application, you need to be able to resolve `myapp.local`. The exact access methods depends on your cluster:

- On the **VM Cluster**, Istio has a fixed IP, so you can simply run the following **on your host machine** (not the ctrl node):

  ```bash
  sudo sh -c 'echo "192.168.56.99  myapp.local" >> /etc/hosts'
  ```

- On **Minikube**, just use Minikube tunnel:
  ```bash
  minikube tunnel  # keep this terminal tab open for as long as you need to access myapp.local
  ```

Then access the application at http://myapp.local

Metrics are available at http://myapp.local/metrics

> If you see "no healthy upstream", please wait for the app pods to initialize. Check with `kubectl get pods`.

## Sticky sessions

To test Sticky sessions and primary/canary release routing, you can use curl with the `x-newvers` header.

- If `x-newvers=true`, it will always show v2
- If `x-newvers=false`, it will always show v1
- If `x-newvers` is not specified (i.e. in case of a new user), there is an 80% chance to see v1 and 20% to see v2.

Example for sticky session to v2:

```bash
# VM Cluster
for i in {1..10}; do curl -s -H "x-newvers: true" -H "x-user-id: testuser" http://myapp.local ; done | grep "App Version"

# Minikube (make sure you have a second terminal tab open with minikube tunnel)
for i in {1..10}; do curl -s -H "Host: myapp.local" -H "x-newvers: true" -H "x-user-id: testuser" http://localhost:8080 ; done | grep "App Version" # In another terminal tab
```

## Prometheus, Grafana and Alert Manager

### Prometheus

Access Prometheus at http://localhost:9090 (or the Minikube URL) on your host machine:

```bash
# VM cluster (Run below on host):
export PROMETHEUS_POD_NAME=$(kubectl -n istio-system get pod -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=myprom-kube-prometheus-sta-prometheus" -oname)
kubectl -n istio-system port-forward $PROMETHEUS_POD_NAME 9090

# Minikube:
minikube service myprom-kube-prometheus-sta-prometheus --url
```

### Grafana

Access Grafana at http://localhost:3000 (or the Minikube URL) on your host machine:

```bash
# VM Cluster (Run below on host):
export GRAFANA_POD_NAME=$(kubectl -n istio-system get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=myprom" -oname)
kubectl -n istio-system port-forward $GRAFANA_POD_NAME 3000

# Minikube:
minikube service myprom-grafana --url
```

Grafana login credentials:

- Username: `admin`
- Password: Run `kubectl --namespace istio-system get secrets myprom-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo`

The dashboard configurations inside the folder `helm/myapp/grafana/` are **automatically imported through a ConfigMap**, so no manual installation is required. Simply go to the Dashboards tab in Grafana and load the 'Restaurant sentiment analysis' dashboard:

![Grafana dashboards tab showing our dashboard](imgs/grafana_load_dashboard.png)

It should look like this:

![Grafana dashboard](imgs/grafana_dashboard.png)

### Alert Manager

Access Alert Manager at http://localhost:9093/:

```bash
export ALERTMANAGER_POD=$(kubectl -n istio-system get pod -l app.kubernetes.io/name=alertmanager -o name)
kubectl -n istio-system port-forward "$ALERTMANAGER_POD" 9093:9093
```

# Troubleshooting

This section lists some backup methods and workarounds for problems that we have encountered. We list them separately to keep the main installation instructions readable.

**Problem**: kubectl does not work from host, only from the ctrl node. Cannot access Prometheus/Grafana.
**Solution**: Since kubectl still works on ctrl, just use ctrl's IP address instead of localhost. For example, to access Grafana at http://192.168.56.100:3000:

```bash
vagrant ssh ctrl
cd /mnt/shared

export GRAFANA_POD_NAME=$(kubectl -n istio-system get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=myprom" -oname)
kubectl -n istio-system port-forward --address=0.0.0.0 $GRAFANA_POD_NAME 3000
```

And for Prometheus at http://192.168.56.100:9090:

```bash
# VM Cluster (Backup option)
vagrant ssh ctrl
cd /mnt/shared

export PROMETHEUS_POD_NAME=$(kubectl -n istio-system get pod -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=myprom-kube-prometheus-sta-prometheus" -oname)
kubectl -n istio-system port-forward --address=0.0.0.0 $PROMETHEUS_POD_NAME 9090
```

# Final State of our Assignment

We aimed to meet the excellent criteria for all rubric points. We prepared a detailed rubric document called [RUBRIC.md](https://github.com/remla25-team12/operation/blob/main/RUBRIC.md) to describe the status of our project for each rubric criterion and to add additional comments that might need consideration during grading.

# Continuous Progress Log

## Assignment 1

The docker-compose.yaml successfully launches both containers (`app` and `model-service`). They communicate with each other over a shared Docker network, and only `app`'s port is exposed to the host. The app frontend can be used to query the model for predictions. All libraries and containers are released using automatic versioning via a GitHub workflow. We do not yet have additional interaction options (e.g. flagging incorrect predictions).

## Assignment 2

The Kubernetes-based deployment infrastructure is fully operational. MetalLB and NGINX Ingress Controller are installed and configured with fixed IP addresses. The Kubernetes Dashboard is deployed via Helm and exposed via an Ingress using HTTPS with self-signed certificates. The dashboard loads correctly at `https://dashboard.local`, and token-based login is functional through the command line, although UI login fails with a 401. This setup lays the groundwork for future feature deployments and secure service exposure. All steps described in the assignment document, including the optional Step 23, are implemented.

In Step 14, coping config to hosts is not done fully. While the current ctrl.yaml copies it to privisioning folder, the step in the assignment **"config should be usable from the host via KUBECONFIG environment variable or through --kubeconfig argument:"** will still need to be implemented.

Please closely follow the steps described under **Setting Up the Kubernetes Cluster** section of this README to test the functionality of our Kubernetes-based deployment infrastructure.

## Assignment 3

All issues and pending solutions described in Assignment 2 have been resolved.

Our project status for Assignment 3 is as follows:

| Category          | Expected Rating | Notes                                                                                                              |
| ----------------- | --------------- | ------------------------------------------------------------------------------------------------------------------ |
| Kubernetes Usage  | **Excellent**   | All criteria described in the Assignment 3 rubric is implemented.                                                  |
| Helm Installation | **Excellent**   | All criteria described in the Assignment 3 rubric is implemented.                                                  |
| App Monitoring    | **Good**        | Our AlertManager implementation is not yet fully functional. We are currently having troubles with sending emails. |
| Grafana           | **Excellent**   | All criteria described in the Assignment 3 rubric is implemented.                                                  |

## Assignment 4

For A4, all criteria described in the rubric is implemented for our [model-training](https://github.com/remla25-team12/model-training/tree/a4) pipeline.

Our project status for Assignment 4 is as follows:

| Category                     | Expected Rating | Notes                                                                                    |
| ---------------------------- | --------------- | ---------------------------------------------------------------------------------------- |
| Project Organization         | **Excellent**   | The model-training reporsitory is rearranged based on the Cookiecutter template.         |
| Pipeline Management with DVC | **Excellent**   | Our model-training pipeline is managed by DVC and uses a cloud-based remote storage.     |
| Code Quality                 | **Excellent**   | Our project applies multiple linters and implements at least one custom pylint rule.     |
| Automated Tests              | **Excellent**   | Test coverage is automatically measured.                                                 |
| Continuous Training          | **Excellent**   | Test adequacy score and test coverage are added and automatically updated in the README. |

## Assignment 5

For A5, traffic management is implemented. The app defines a Gateway and VirtualServices. Our application is accessible through the IngressGateway. It uses DestinationRules and weights to enable a 90/10 routing of the app service. The versions of model-service and app are consistent. Also, we implemented the Sticky sessions (excellent criteria). These changes are visible on the [operation repository](https://github.com/remla25-team12/operation).

For the purpose of traffic management, new versions for both model-service and app is defined. The new version of model-service uses a logistic classifier model released by model-training repo to make sentiment predictions. Check the [model-training](https://github.com/remla25-team12/model-training) and [model-service](https://github.com/remla25-team12/model-service) repositories for the updates.

On the second version of the app, changes on the Frontend design is introduced. A back button is added on the second page instead of the _analyze another review_ button at the bottom of the page. Also, a placeholder text is added to the review submission box to guide the users in their reviews. Check the [app](https://github.com/remla25-team12/app) repository for the updates.

For documentation, a template documentation file is defined but it only satisfies the sufficient criteria for now.

Our project status for Assignment 5 is as follows:

| Category                   | Expected Rating   | Notes                                                                                                       |
| -------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------- |
| Traffic Management         | **Excellent**     | All criteria for this category are implemented (described in detail in the above paragraphs).               |
| Additional Use-case        | **Still a To-do** | An additional use-case is not implemented yet.                                                              |
| Continuous Experimentation | **Still a To-do** |                                                                                                             |
| Deployment Documentation   | **Sufficient**    | For documentation, a template documentation file is defined. The documentation is still a work-in-progress. |
| Extension Proposal         | **Still a To-do** |                                                                                                             |

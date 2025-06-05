# Operation

REMLA Group 12

<!-- TABLE OF CONTENTS -->
<details>
  <summary><b>Table of Contents</b></summary>
  <ol>
    <li>
      <a href="#about-the-project">About the Project</a>
      <ul>
        <li>
          <a href="#relevant-repositories">Relevant Repositories</a>
        </li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#deployment-with-docker">Deployment with Docker</a></li>
        <li><a href="#kubernetes-cluster-setup">Kubernetes Cluster setup</a></li>
        <li><a href="#deployment-on-kubernetes-cluster-helm">Deployment on Kubernetes Cluster (Helm)</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
         <ul>
        <li><a href="#webapp-access">Webapp Access</a></li>
        <li><a href="#traffic-management">Traffic Management</a></li>
        <li><a href="#prometheus-and-grafana">Prometheus and Grafana</a></li>
      </ul>
    <li><a href="#continuous-progress-log">Continuous Progress Log</a></li>
      <ul>
        <li><a href="#assignment-1">Assignment 1</a></li>
        <li><a href="#assignment-2">Assignment 2</a></li>
        <li><a href="#assignment-3">Assignment 3</a></li>
        <li><a href="#assignment-4">Assignment 4</a></li>
        <li><a href="#assignment-5">Assignment 5</a></li>
      </ul>
  </ol>
</details>

# 1. About the Project

This is an adaptation and application of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project.
This repository serves as the central point of the project, containing the Docker and Kubernetes deployment configurations, installation instructions, and links to other relevant components and repositories.

## 1.1 Relevant Repositories

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

## Kubernetes Cluster setup

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

3. Before continuing, it is important to reset the host-only networks in VirtualBox. This prevents conflicts with stale or broken network configurations from previous VirtualBox setups.
   To do this, **open the VirtualBox GUI**, go to `Tools > Network`, and **remove any existing "Host-only Networks"** listed under that section. The list should now be empty:
   ![Empty host-only adapter list](imgs/vb_empty.png)

4. Register SSH Key

   Follow the below steps to register your SSH key:

   <!-- - Let's say my name is: **abc**.
   - Add SSH key under the directory **provisioning/keys/abc.pub**. The format of the file should be: **“abc_key: <my_ssh_key>”**.
   - Run the encryption command, replacing abc with your name:
     ```bash
      ansible-vault encrypt provisioning/keys/abc.pub
     ```
   - When prompted, the vault password you need to use is: **remla25-team12-vagrant**.
   - Then save the password file in your home directory with the following commands, as it is accessed by the Vagrantfile:
     ```bash
      echo 'remla25-team12-vagrant' > ~/.vault_pass.txt
      chmod 600 ~/.vault_pass.txt
     ``` -->

   First generate the key:

   ```bash
    ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible-provision-key -C "ansible provision key"
   ```

   Move your public key to provisioning/keys folder with your name:

   ```bash
    mv ~/.ssh/ansible-provision-key.pub provisioning/keys/<name>-key.pub
   ```

5. Start the virtual environment from the repository's root folder:

   ```bash
    cd operation
    vagrant up
    vagrant provision
   ```

   This operation may take a while to complete.

6. Once the VMs are up and provisioned, run the following Ansible playbook to finalize the Kubernetes setup:

   ```bash
    ansible-playbook -u vagrant -i 192.168.56.100, provisioning/finalization.yml \
    --private-key=.vagrant/machines/ctrl/virtualbox/private_key 
   ```

7. To access the Kubernetes dashboard, do the following **on your host machine**:

   - Add `dashboard.local` to your `/etc/hosts` file. For example, use the following command:

     ```bash
     sudo sh -c 'echo "192.168.56.91 dashboard.local >> /etc/hosts'
     ```

   - Navigate to https://dashboard.local (note the https) and enter the token displayed in the terminal.

     ![Token as shown in the terminal](imgs/terminal_token.png)

     > **Note**: The token was generated during the final provisioning step. If you cannot find the token in the terminal output anymore, run `vagrant ssh ctrl`, followed by `kubectl -n kubernetes-dashboard create token admin-user` to generate a new one.

8. To communicate with the cluster from the host, a kubeconfig file (`admin.conf`) has been exported by Ansible. For example, you can run:
   ```bash
    kubectl get ns --kubeconfig ./provisioning/admin.conf
   ```
   You can also set the filepath as an environment variable or add it to `~/.bashrc`, so that you do not need to use the `--kubeconfig` flag every time:
   ```bash
    export KUBECONFIG="./provisioning/admin.conf"
   ```

---

## Deployment on Kubernetes Cluster (Helm)

### Prerequisites

- macOS or Linux (host operating system)
- [Helm 3 CLI](https://helm.sh/docs/intro/install/)
- [Istioctl](https://istio.io/latest/docs/setup/install/istioctl/) 1.25.2 or higher
- A functional Kubernetes cluster.
  - Recommended: VM cluster from the [Kubernetes Cluster setup](#kubernetes-cluster-setup) section. In the instructions below, it is assumed you already have this cluster up and running.
  - Alternatively, you can install and use [Minikube](https://minikube.sigs.k8s.io/docs/start/) for a local Kubernetes cluster. 

### Install and run

1. Clone this repository and navigate into the root folder:

   ```bash
   git clone https://github.com/remla25-team12/operation.git
   cd operation
   ```

2. Prepare the cluster for app installation.

   1. For the **Kubernetes VM cluster**, SSH into the control node and navigate to the shared folder directory.
      ```bash
      vagrant ssh ctrl
      cd /mnt/shared/
      ```

   2. For **Minikube**, it is recommended to first clean up any previous Minikube instance and then launch a new cluster by running the following commands:

      ```bash
      minikube delete
      minikube start --memory=4096 --cpus=4 --driver=docker
      minikube addons enable ingress
      istioctl install --set profile=default -y 
      ```

      > **Note:** If you are using Fedora, you may need to run the following command first to allow Minikube to use the Docker driver:

      ```bash
      sudo setenforce 0
      ```

3. Enable Istio sidecar injection in the (default) namespace:

   ```bash
   kubectl label namespace default istio-injection=enabled
   ```

4. Install and deploy the Prometheus stack (in a different namespace, without Istio sidecar injection):

   ```bash
   kubectl create namespace monitoring
   kubectl label namespace monitoring istio-injection=disabled
   
   helm repo add prom-repo https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install myprom prom-repo/kube-prometheus-stack -n monitoring \
       --set prometheus.prometheusSpec.maximumStartupDurationSeconds=120
   ```

5. Install and deploy our application. One of the flags used in this command will differ depending on your cluster setup.
   i. For the **Kubernetes VM cluster**, use `useHostPathSharedFolder=true`:
   ```bash
   helm install myapp-dev ./helm/myapp \
      --set useHostPathSharedFolder=true
   ```
   > **Note:** In Values.yaml, `useHostPathSharedFolder` is set to `false` by default.

      
   ii. For **Minikube**, use `useHostPathSharedFolder=false`:
   ```bash
   helm install myapp-dev ./helm/myapp \
       --set useHostPathSharedFolder=false
   ```

7. If you make changes to the Helm chart or want to update the deployment, use the following command:
   ```bash
   helm upgrade --install myapp-dev ./helm/myapp
   ```

## Usage 
### Webapp access
**VM Cluster**\
To access the deployed application, you must add `myapp.local` to your hosts file. 
- Run `kubectl get svc istio-ingressgateway -n istio-system` 
- Take note of the EXTERNAL-IP. 
- Add `EXTERNAL-IP  myapp.local` to your hosts file.
   > The IP is not fixed: if you restart the cluster, it may have changed, so always check that the EXTERNAL-IP and what's in your host file match. Making IP fixed is TODO (excellent requirement)

Then access the application at http://myapp.local
Metrics are available at http://myapp.local/metrics

**Minikube**\
Minikube requires a port forward, so to access the deployed application, you must add `127.0.0.1 myapp.local` to your hosts file.

```bash
# Add the host entry (one-time setup)
sudo sh -c 'echo "127.0.0.1 myapp.local" >> /etc/hosts'

# Start the port-forward
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```
Then access the application at http://localhost:8080
Metrics are available at http://localhost:8080/metrics

### Traffic management
To test the traffic management and primary/canary release routing, you can use curl with Host header:
```bash
# First, port-forward the Istio ingress
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# Then in another terminal:
# For sticky session to v2 (always v2):
for i in {1..5}; do curl -s -H "Host: myapp.local" -H "x-newvers: true" -H "x-user-id: testuser" http://localhost:8080 ; done

# For sticky session to v1 (always v1):
for i in {1..5}; do curl -s -H "Host: myapp.local" -H "x-newvers: false" -H "x-user-id: testuser" http://localhost:8080 ; done

# For normal split (as defined in values.yaml), just omit x-newvers:
for i in {1..5}; do curl -s -H "Host: myapp.local" http://localhost:8080 ; done
```
> Replace https://localhost:8080 with the EXTERNAL-IP of the IngressGateway if you're on VM Cluster instead of Minikube.

#### Prometheus and Grafana
**VM Cluster**
Access Prometeus by [TODO]
```bash


```


Access Grafana by running the following **on your host**:
```bash
export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=myprom" -oname)
kubectl --namespace monitoring port-forward $POD_NAME 3000:3000
```
Then navigate to http://localhost:3000. 

Grafana login credentials:
- Username: `admin`
- Password: Run `kubectl --namespace monitoring get secrets myprom-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo`


The dashboard configuration (`helm/myapp/grafana/dashboard.json`) is automatically imported through a ConfigMap, so no manual installation is required. Simply go to the Dashboards tab in Grafana and load the 'Restaurant sentiment analysis' dashboard:

![Grafana dashboards tab showing our dashboard](imgs/grafana_load_dashboard.png)

It should look like this:

![Grafana dashboard](imgs/grafana_dashboard.png)


**Minikube**
Access Prometheus through the URL generated by the following command:

```bash
minikube service myprom-kube-prometheus-sta-prometheus --url
```

Access Grafana through the URL generated by the following command:

```bash
minikube service myprom-grafana --url
```

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
...
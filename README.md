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
        <!-- <ul>
          <li><a href="#prerequisites-for-docker">Prerequisites</a></li>
          <li><a href="#install-and-run-with-docker">Install and run</a></li>
        </ul> -->
        <li><a href="#kubernetes-cluster-setup">Kubernetes Cluster setup</a></li>
        <li><a href="#deployment-on-kubernetes-cluster-helm">Deployment on Kubernetes Cluster (Helm)</a></li>
      </ul>
    </li>
    <li><a href="#app-usage-minikube">App Usage (Minikube)</a></li>
    <li><a href="#continuous-progress-log">Continuous Progress Log</a></li>
      <ul>
        <li><a href="#assignment-1">Assignment 1</a></li>
        <li><a href="#assignment-2">Assignment 2</a></li>
        <li><a href="#assignment-3">Assignment 3</a></li>
      </ul>
  </ol>
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
    ansible-playbook \
    -u vagrant \
    -i 192.168.56.100, \
    --private-key=.vagrant/machines/ctrl/virtualbox/private_key \
    provisioning/finalization.yml
   ```

7. To access the Kubernetes dashboard, do the following **on your host machine**:

   - Edit the `/etc/hosts` file to resolve https://dashboard.local. First, open the file by running:

     ```bash
      sudo nano /etc/hosts
     ```

     Then append the following line to the file.

     ```plaintext
      192.168.56.91 dashboard.local
     ```

   - Navigate to https://dashboard.local (note the https) and enter the token displayed in the terminal.

     ![Token as shown in the terminal](imgs/terminal_token.png)

     > **Note**: The token is generated in the previous step. If you cannot find the token in the terminal output, run `vagrant ssh ctrl`, followed by `kubectl -n kubernetes-dashboard create token admin-user` to generate a new one.

8. To communicate with the cluster from the host, a kubeconfig file (`admin.conf`) has been exported by Ansible. For example, you can run:
   ```bash
    kubectl get ns --kubeconfig ./provisioning/admin.conf
   ```
   You can also set the filepath as an environment variable, so that you do not need to use the `--kubeconfig` flag every time:
   ```bash
    export KUBECONFIG="./provisioning/admin.conf"
   ```

---

## Deployment on Kubernetes Cluster (Helm)

### Prerequisites

- macOS or Linux (host operating system)
- [Helm 3 CLI](https://helm.sh/docs/intro/install/)
- A functional Kubernetes cluster.
  - See the [Kubernetes Cluster setup](#kubernetes-cluster-setup) instructions above for a VM-based cluster.
  - Alternatively, you can install and use [Minikube](https://minikube.sigs.k8s.io/docs/start/) for a local Kubernetes cluster.

### Install and run

1. Clone this repository and navigate into the root folder:

   ```bash
   git clone https://github.com/remla25-team12/operation.git
   cd operation
   ```

2. Prepare the cluster for app installation.

   1. For **Minikube**, it is recommended to first clean up any previous Minikube instance and launch a new cluster by running the following commands:

      ```bash
      minikube delete
      minikube start --driver=docker
      minikube addons enable ingress
      ```

      > **Note:** If you are using Fedora, you may need to run the following command first to allow Minikube to use the Docker driver:

      ```bash
      sudo setenforce 0
      ```

   2. For the **Kubernetes VM cluster**, SSH into the control node and navigate to the shared folder directory.
      ```bash
      vagrant ssh ctrl
      cd /mnt/shared/
      ```
3. Install and deploy the Prometheus stack and Istio:
```bash
   helm repo add istio https://istio-release.storage.googleapis.com/charts
   helm repo update
   helm install istio-base istio/base -n istio-system --create-namespace
   helm install istiod istio/istiod -n istio-system 
   helm install istio-ingress istio/gateway -n istio-system 
   kubectl label namespace default istio-injection=enabled
```



   ```bash
   kubectl create namespace monitoring
   kubectl label namespace monitoring istio-injection=disabled
   helm repo add prom-repo https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install myprom prom-repo/kube-prometheus-stack -n monitoring
   ```

4. Install and deploy our application. One of the flags used in this command will differ depending on your cluster setup.

   i. For the **Kubernetes VM cluster**, use `useHostPathSharedFolder=true`:

   ```bash
   helm install myapp-dev ./helm/myapp \
   --set app.image.tag=latest \
   --set model.image.tag=latest \
   --set model.port=5000 \
   --set app.port=8080 \
   --set useHostPathSharedFolder=true
   ```

   ii. For **Minikube**, use `useHostPathSharedFolder=false`:

   ```bash
   helm install myapp-dev ./helm/myapp \
   --set app.image.tag=latest \
   --set model.image.tag=latest \
   --set model.port=5000 \
   --set app.port=8080 \
   --set useHostPathSharedFolder=false
   ```

5. If you make changes to the Helm chart or want to update the deployment, use the following command:
   ```bash
   helm upgrade --install myapp-dev ./helm/myapp \
   --set app.image.tag=latest \
   --set model.image.tag=latest \
   --set model.port=5000 \
   --set app.port=8080 \
   --set useHostPathSharedFolder=true
   ```
   > Do not forget to set `useHostPathSharedFolder` based on your type of cluster (see previous step)

## App Usage (Minikube)

### Webapp

To access the deployed application at http://localhost:8080 (through a port-forward):

```bash
kubectl port-forward svc/myapp-dev-myapp-app 8080:8080
```

> Make sure that none of the pods are in a pending state before port forwarding. You can check the status of your pods with:

```bash
kubectl get pods
```

Metrics are available at http://localhost:8080/metrics

### Prometheus

To access Prometheus, use the URL generated by the following command:

```bash
minikube service myprom-kube-prometheus-sta-prometheus --url
```

> There should be a ServiceMonitor/default/myapp-dev-myapp/0 under status->TargetHealth that is greent/up.

### Grafana

To access Grafana, use the URL generated by the following command:

```bash
minikube service myprom-grafana --url
```

The password for Grafana's `admin` account can be generated with:

```bash
kubectl --namespace default get secrets myprom-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

As the username, enter `admin`. As the password, enter the generated password.

The dashboard configuration (`helm/myapp/grafana/dashboard.json`) is automatically imported through a ConfigMap, so no manual installation is required. Simply go to the Dashboards tab in Grafana and load the 'Restaurant sentiment analysis' dashboard:

![Grafana dashboards tab showing our dashboard](imgs/grafana_load_dashboard.png)

It should look like this:

![Grafana dashboard](imgs/grafana_dashboard.png)

---

# Continuous Progress Log

_Add a new paragraph for each assignment as a continuous progress log that (briefly) describes which assignment parts have been implemented to support the peer-review process._

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

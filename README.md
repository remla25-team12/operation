# Operation

REMLA Group 12

## About

This project is an adaptation of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project.
This repository serves as the central point of the project, containing the Docker and Kubernetes deployment configurations, installation instructions, and links to other components.

## Relevant Repositories

| Repository                                                         | Purpose                                                                                   |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [model-training](https://github.com/remla25-team12/model-training) | Training pipeline for the restaurant sentiment analysis model.                            |
| [model-service](https://github.com/remla25-team12/model-service)   | Model wrapper for communication with other components of the project, e.g. for inference. |
| [lib-ml](https://github.com/remla25-team12/lib-ml)                 | Preprocessing logic for input data, used during model training and inference.             |
| [lib-version](https://github.com/remla25-team12/lib-version)       | A simple version-aware libary that reports its own version.                               |
| [app](https://github.com/remla25-team12/app)                       | Webapp (frontend + service) to interface with the model.                                  |

## Deployment with Docker

### Requirements

- Linux or macOS
- Docker and Docker Compose

### Install and Run

1. Clone this repository.
   ```bash
    git clone https://github.com/remla25-team12/operation.git
   ```
2. Deploy the project using Docker Compose by running this command in the project's root folder.

   ```bash
   docker compose up -d
   ```

3. Navigate to http://localhost:8080 to access the application homepage.

## Kubernetes Cluster setup

### Prerequisites

- **Host Operating System**: macOS or Linux
- **Virtualization**: VirtualBox
- **Provisioning Tools**: Vagrant and Ansible

### Install and Run

1. Clone this repository:

   ```bash
    git clone https://github.com/remla25-team12/operation.git
   ```

2. Navigate into the repository's root folder and install the required Ansible collections:

   ```bash
    ansible-galaxy collection install -r requirements.yml
   ```

3. Before continuing, it is important to reset the host-only networks in VirtualBox. This prevents conflicts with stale or broken network configurations from previous VirtualBox setups.
   To do this, **open the VirtualBox GUI**, go to `Tools > Network`, and **remove any existing "Host-only Networks"** listed under that section. The list should now be empty:
   ![Empty host-only adapter list](imgs/vb_empty.png)

4. Start the virtual environment from the repository's root folder:

   ```bash
    cd operation
    vagrant up
    vagrant provision
   ```

   This operation may take a while to complete.

5. Once the VMs are up and provisioned, run the following Ansible playbook to finalize the Kubernetes setup:

   ```bash
    ansible-playbook -u vagrant -i 192.168.56.100, provisioning/finalization.yml
   ```

6. To access the Kubernetes dashboard, do the following **on your host machine**:

   - Edit the `/etc/hosts` file to resolve https://dashboard.local, by running:

   ```bash
    sudo nano /etc/hosts/
   ```

   - Then append below text to the file.

   ```plaintext
    192.168.56.91 dashboard.local
   ```

   - Navigate to https://dashboard.local (note the https) and enter the token displayed in the terminal.

   ![Token as shown in the terminal](imgs/terminal_token.png)

   > **Note**: The token is generated in the previous step. If you cannot find the token in the terminal output, run `vagrant ssh ctrl`, followed by `kubectl -n kubernetes-dashboard create token admin-user` to generate a new one.

7. To communicate with the cluster from the host, a kubeconfig file (`admin.conf`) has been exported by Ansible. For example, you can run:
   ```bash
   kubectl get ns --kubeconfig ./provisioning/admin.conf
   ```
   You can also set the filepath as an environment variable, so that you do not need to use the `--kubeconfig` flag every time:
   ```bash
   export KUBECONFIG="./provisioning/admin.conf"
   ```

## Setting up the Application with Helm

### Prerequisites for Helm Deployment

Ensure you have the following components ready in your environment:

- **Helm 3 Installed:** You'll need the Helm 3 command-line interface (CLI) installed on your local machine. Helm is the package manager for Kubernetes.

- **Running Kubernetes Cluster:** A functional Kubernetes cluster is essential for deploying applications using Helm. To provision a local Kubernetes cluster with Minikube and enable the ingress controller, you can run these commands:

If you are using Fedora, you may need to run the following command to allow Minikube to use the Docker driver:

```bash
 sudo setenforce 0
```

then continue normally:

```bash
 minikube delete
 minikube start --driver=docker
 minikube addons enable ingress
```

> **Note:** Cleaning up any previous Minikube instance is recommended with the `minikube delete` command described above.

### Deploying the Application using the Helm Chart

1.  Install kube-prometheus-stack using Helm if on YOUR SYSTEM

    ```bash
     helm repo add prom-repo https://prometheus-community.github.io/helm-charts
     helm repo update
     helm install myprom prom-repo/kube-prometheus-stack
    ```

    only then:

    ```bash
     helm install myapp-dev ./helm/myapp \
     --set app.image.tag=latest \
     --set model.image.tag=latest \
     --set model.port=5000 \
     --set app.port=8080
    ```

1.1
   IF you're on Vagrant VM's
   vagrant ssh ctrl after vagrant up or vagrant provision and running provisioning/finalization.yml

      ```bash
      cd /mnt/shared/
      ```

    ```bash
     helm repo add prom-repo https://prometheus-community.github.io/helm-charts
     helm repo update
     helm install myprom prom-repo/kube-prometheus-stack
    ```

    only then:

    ```bash
     helm install myapp-dev ./helm/myapp \
     --set app.image.tag=latest \
     --set model.image.tag=latest \
     --set model.port=5000 \
     --set app.port=8080
    ```
2.  (Optional) Upgrading or re-running the Helm chart:
    If you make changes to your Helm chart or need to update the deployed application with new configurations, you can run the below `helm upgrade` command.

    ```bash
     helm upgrade --install myapp-dev ./helm/myapp \
     --set model.image.tag=latest \
     --set app.image.tag=latest \
     --set model.port=5000 \
     --set app.port=8080
    ```

3.  Access the deployed application:

    ```bash
     kubectl port-forward svc/myapp-dev-myapp-app 8080:8080
    ```

    > Navigate to `http://localhost:8080` to access the application.

4.  Access Prometheus

    ```bash
     minikube service myprom-kube-prometheus-sta-prometheus --url
    ```

    There should be a ServiceMonitor/default/myapp-dev-myapp/0 under status->TargetHealth that is greent/up.

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

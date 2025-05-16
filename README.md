# Operation

REMLA Group 12

## About

This project is an adaptation of the [Restaurant Sentiment Analysis](https://github.com/proksch/restaurant-sentiment) project.
This repository serves as the central point of the project, containing the deployment configuration (`docker-compose.yaml`), installation instructions, and links to other components.

## Relevant Repositories

| Repository                                                         | Purpose                                                                                   |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [model-training](https://github.com/remla25-team12/model-training) | Training pipeline for the restaurant sentiment analysis model.                            |
| [model-service](https://github.com/remla25-team12/model-service)   | Model wrapper for communication with other components of the project, e.g. for inference. |
| [lib-ml](https://github.com/remla25-team12/lib-ml)                 | Preprocessing logic for input data, used during model training and inference.             |
| [lib-version](https://github.com/remla25-team12/lib-version)       | A simple version-aware libary that reports its own version.                               |
| [app](https://github.com/remla25-team12/app)                       | Webapp (frontend + service) to interface with the model.                                  |

# Getting Started

## Requirements

- Linux or MacOS
- Docker and Docker Compose

## Install and Run

1. Clone this repository.
2. Deploy the project using Docker Compose by running this command in the project's root folder.

```
$ docker compose up -d
```

3. Navigate to `http://localhost:8080` to access the application homepage.

## Setting Up the Kubernetes Cluster

### Prerequisites

- **Operating System**: macOS or Linux
- **Virtualization**: VirtualBox
- **Provisioning Tools**: Vagrant and Ansible

### Install and Run

1. Clone the repository:

   ```bash
    git clone https://github.com/remla25-team12/operation.git
    cd operation
   ```

2. Start the virtual environment:

   ```bash
    vagrant up
    vagrant provision
   ```

3. Reset host-only networks in VirtualBox:

   Before continuing, **open the VirtualBox GUI** and:

   - Go to `Tools > Network`
   - **Remove any existing "Host-only Networks"** listed under that section

   > This prevents conflicts with stale or broken network configurations from previous VirtualBox setups.

4. Run the Ansible playbook

   Once the VM is up, run the following Ansible playbook to finalize the Kubernetes setup:

   ```bash
    ansible-playbook -u vagrant -i 192.168.56.100, provisioning/finalization.yml
   ```

   When running this command, you can receive an error. For example, in the **Check if ingress-nginx is already installed** step, below error could occur:

   ```
   fatal: [192.168.56.100]: FAILED! => {"changed": true, "cmd": ["helm", "status", "ingress-nginx", "-n", "ingress-nginx"], "delta": "0:00:00.048696", "end": "2025-05-13 09:13:26.349148", "msg": "non-zero return code", "rc": 1, "start": "2025-05-13 09:13:26.300452", "stderr": "Error: release: not found", "stderr_lines": ["Error: release: not found"], "stdout": "", "stdout_lines": []}.
   ```

   We hypothesize that the VM takes some time to set everything from the backend. In such a case, just re-run the command.

   ```bash
    ansible-playbook -u vagrant -i 192.168.56.100, provisioning/finalization.yml
   ```

5. Access the Kubernetes Dashboard

   Once provisioning is complete, the Kubernetes Dashboard will be available at:

   ```bash
    dashboard.local
   ```

   > Open this URL in your browser.

6. Add host entry

   To resolve `dashboard.local`, you must update your `/etc/hosts` file by running:

   ```bash
    sudo nano /etc/hosts/
   ```

   Then append below text to the file.

   ```plaintext
    192.168.56.91 dashboard.local
   ```

7. Log in to the Dashboard

   Access the control node by running:

   ```
    vagrant ssh ctrl
   ```

   Generate a token with:

   ```bash
    kubectl -n kubernetes-dashboard create token admin-user
   ```

   > Or check the logs of the RBAC YAML used to provision `admin-user` to find the pre-generated token.

   Paste the token into the dashboard login screen.

   > **Note:** Logging in via browser may display the UI but result in `401 Unauthorized`. This is a known issue.
   >
   > However, login using the token **via `kubectl` works correctly.**

## For HELM
### ✅ 1. Prerequisites

- Helm 3 installed
- Kubernetes cluster running (e.g., Minikube)

Optional for Minikube:

If you are using Fedora, you may need to run the following command to allow Minikube to use the Docker driver:
```bash
sudo setenforce 0
```
then continue normally:
```bash
minikube start --driver=docker
minikube addons enable ingress
```
### Install the helm chart and prometheus dependency
Install kube-prometheus-stack using helm
```bash
helm repo add prom-repo https://prometheus-community.github.io/helm-charts
helm repo update
helm install myprom prom-repo/kube-prometheus-stack
```

```bash
helm install myapp-dev ./helm/myapp  --set app.image.tag=latest   --set model.image.tag=latest   --set model.port=5000   --set app.port=8080

```

if you want to change after making any changes, do 
```bash
helm upgrade --install myapp-dev ./helm/myapp   --set model.image.tag=latest   --set app.image.tag=latest   --set model.port=5000   --set app.port=8080
```
### Access Prometheus

If you havent done the port forwarding in the previous step do it (you may have to wait for it to be up and running)
```bash
 kubectl port-forward svc/myapp-dev-myapp 8080:8080
```

Then, to access the Prometheus UI, run the following command (in a new terminal):
```bash  
minikube service myprom-kube-prometheus-sta-prometheus --url
```
There should be a ServiceMonitor/default/myapp-dev-myapp/0 under status->TargetHealth that is greent/up.

## Config

The .env file in this repository allows you to choose which version of the `app` and `model-service` container you'd like to use. If none are specified, the latest images will be pulled by default.

# Continuous Progress Log

_Add a new paragraph for each assignment as a continuous progress log that (briefly) describes which assignment parts have been implemented to support the peer-review process._

## Assignment 1

The docker-compose.yaml successfully launches both containers (`app` and `model-service`). They communicate with each other over a shared Docker network, and only `app`'s port is exposed to the host. The app frontend can be used to query the model for predictions. All libraries and containers are released using automatic versioning via a GitHub workflow. We do not yet have additional interaction options (e.g. flagging incorrect predictions).

## Assignment 2

The Kubernetes-based deployment infrastructure is fully operational. MetalLB and NGINX Ingress Controller are installed and configured with fixed IP addresses. The Kubernetes Dashboard is deployed via Helm and exposed via an Ingress using HTTPS with self-signed certificates. The dashboard loads correctly at `https://dashboard.local`, and token-based login is functional through the command line, although UI login fails with a 401. This setup lays the groundwork for future feature deployments and secure service exposure. All steps described in the assignment document, including the optional Step 23, are implemented.

In Step 14, coping config to hosts is not done fully. While the current ctrl.yaml copies it to privisioning folder, the step in the assignment **"config should be usable from the host via KUBECONFIG environment variable or through --kubeconfig argument:"** will still need to be implemented.

Please closely follow the steps described under **Setting Up the Kubernetes Cluster** section of this README to test the functionality of our Kubernetes-based deployment infrastructure.

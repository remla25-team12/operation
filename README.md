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
   ```
if needed run vagrant provision and then

like step 1.4 says run ansible-playbook -u vagrant -i 192.168.56.100, finalization.yml
Once provisioning is complete, the Kubernetes Dashboard will be available via Ingress at:

- `https://dashboard.local`

> **Note**: You must add the following to your `/etc/hosts` file:
> ```
> 192.168.56.91 dashboard.local
> ```

To log in:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```
or see the token already automatically created for you in the logs of the yaml


Paste the token on the login screen. Accessing the dashboard **via browser** currently shows the interface but login results in 401 Unauthorized. However, login **via kubectl token works correctly**.

## Config

The .env file in this repository allows you to choose which version of the `app` and `model-service` container you'd like to use. If none are specified, the latest images will be pulled by default.

# Continuous Progress Log

_Add a new paragraph for each assignment as a continuous progress log that (briefly) describes which assignment parts have been implemented to support the peer-review process._

## Assignment 1

The docker-compose.yaml successfully launches both containers (`app` and `model-service`). They communicate with each other over a shared Docker network, and only `app`'s port is exposed to the host. The app frontend can be used to query the model for predictions. All libraries and containers are released using automatic versioning via a GitHub workflow. We do not yet have additional interaction options (e.g. flagging incorrect predictions).

## Assignment 2

The Kubernetes-based deployment infrastructure is fully operational. MetalLB and NGINX Ingress Controller are installed and configured with fixed IP addresses. The Kubernetes Dashboard is deployed via Helm and exposed via an Ingress using HTTPS with self-signed certificates. The dashboard loads correctly at `https://dashboard.local`, and token-based login is functional through the command line, although UI login fails with a 401. This setup lays the groundwork for future feature deployments and secure service exposure.
All steps including optional until the end are done.
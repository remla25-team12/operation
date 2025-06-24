# A1 Versions, Releases, and Containerization

## A1.1 Basic Requirements

### Data Availability - Pass

- We followed the GitHub organization template as requested.
- All repositories have at least a basic README on how to contribute to the codebase.
- [operation](https://github.com/remla25-team12/operation) hosts the detailed README for deploying the entire project.

### Sensible Use Case - Pass

- Our frontend can query the model.
- Users are shown the predicted sentiment of the review and can indicate if the prediction is correct or not.
- User feedback on predictions is collected and stored in a new dataset.
- Our frontend has a "People" page with our pictures that redirect to LinkedIn pages when clicked (used in continuous experimentation).

## A1.2 Versioning & Releases

### Automated Release Process - Excellent

- [app](https://github.com/remla25-team12/app) and [model-service](https://github.com/remla25-team12/model-service) have auto-patch bumps and multiple pre-release automation.
- [lib-ml](https://github.com/remla25-team12/lib-ml) and [lib-version](https://github.com/remla25-team12/lib-version) are tagged and released manually.

### Software Reuse in Libraries - Excellent

- The version string in lib-version is updated with an [auto-generated .txt]() file in the repository.

## A1.3 Containers & Orchestration

### Containers & Orchestration - Excellent

- See [.env]() file.

### Docker Compose Operation - Excellent

- Docker Secret was not implemented. We discussed in class that this was not possible because Secrets only work with Docker Swarms.

# A2 Provisioning a Kubernetes Cluster

## A2.1 Provisioning

### Setting up (Virtual) Infrastructure - Excellent

- Loops and template arithmetic are used to define node IPs
- CPU/memory/num workers are variables in Vagrantfile
- Number of workers passed from Vagrant to Ansible
- [inventory.cfg]()

### Setting up Software Environment

Good:

- Both built-in modules (community.general) and Kubernetes.core modules are used for idempotency
- Several variables are registered, e.g. "register: dashboard_token"
- Loop used for SSH keys
- Cluster does not get re-inizialied upon reprovisioning:
  - general.yaml, ctrl.yaml and node.yaml are fully idempotent (no [changed] tasks after re- provisioning).
  - finalization.yaml is as idempotent as possible. Only the installation of Istio will always show [changed] (despite nothing actually changing).

Excellent:

- [Jinja2 template]() is used to dynamically generate a `/etc/hosts` file.
- Waiting step: "Waiting for MetalLB webhook pod to be ready" in finalization.yaml
- Idempotent [regex-based replacement]()

### Setting up Kubernetes - Excellent

Sufficient:

- kubectl config: `./provisioning/admin.conf`
- Host-based kubectl can communicate with control plane after exporting `admin.conf` as environment variable.

Good:

- MetalLB installed
- HTTP Ingress Controller for Kubernetes Dashboard
- Istio Gateway for our app

Excellent:

- Kubernetes Dashboard directly reachable at dashboard.local (or 192.168.56.91)
- Nginx Ingress Controller fixed IP: 192.168.56.91
- Istio IngressGateway fixed IP: 192.168.56.99
- HTTPS Nginx Ingress Controller with self-signed certs for Kubernetes Dashboard (https://dashboard.local)
  - Your browser may still throw a security warning because the TLS certs are not signed by LetsEncrypt or another reputable Certificate Authority.

# A3 Operate and Monitor Kubernetes
## A3.1 Kubernetes & Monitoring
### Kubernetes Usage - Excellent

### Helm - Excellent

### App Monitoring - Excellent

### Grafana - Excellent
- No manual installation instructions, because we use a [Configmap]() to install the dashboard.json automatically.
- Gauges, Counters, variable timeframes for parameterizable queries, and rate/avg functions are all used.


# A4

# A5

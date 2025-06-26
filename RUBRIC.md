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

- [lib-version](https://github.com/remla25-team12/lib-version) has a VersionUtil class that retrieves the package version using PEP 621-compliant metadata or falls back to a "dev" string if not installed as a package.

## A1.3 Containers & Orchestration

### Containers & Orchestration - Excellent

- See [.env](https://github.com/remla25-team12/operation/blob/main/.env) file.

### Docker Compose Operation - Excellent

- Docker Secret was not implemented. We discussed in class that this was not possible because Secrets only work with Docker Swarms.

# A2 Provisioning a Kubernetes Cluster

## A2.1 Provisioning

### Setting up (Virtual) Infrastructure - Excellent

- Loops and template arithmetic are used to define node IPs.
- CPU/memory/num workers are variables in Vagrantfile.
- Number of workers passed from Vagrant to Ansible.
- `./provisioning/inventory.cfg` is generated after provisioning.

### Setting up Software Environment

Good:

- Both built-in modules (community.general) and Kubernetes.core modules are used for idempotency
- Several variables are registered, e.g. "register: dashboard_token"
- Loop used for SSH keys
- Cluster does not get re-initialized upon reprovisioning:
  - general.yaml, ctrl.yaml and node.yaml are fully idempotent (no [changed] tasks after re-provisioning).
  - finalization.yaml is as idempotent as possible. Only the installation of Istio will always show [changed] (despite nothing actually changing).

Excellent:

- [Jinja2 template](https://github.com/remla25-team12/operation/blob/main/provisioning/generate_hosts.j2) is used to dynamically generate a `/etc/hosts` file.
- Waiting step: "Waiting for MetalLB webhook pod to be ready" in [finalization.yaml](https://github.com/remla25-team12/operation/blob/main/provisioning/finalization.yml)
- Idempotent [regex-based replacement](https://github.com/remla25-team12/operation/blob/e0ee9fec4e10556e4de1efaae205570eb067ed61/provisioning/ctrl.yaml#L91)

### Setting up Kubernetes - Excellent

Sufficient:

- kubectl config: `./provisioning/admin.conf`
- Host-based kubectl can communicate with control plane after exporting `admin.conf` as environment variable.

Good:

- MetalLB, HTTP Ingress Controller for Kubernetes Dashboard, and Istio all installed in [finalization.yaml](https://github.com/remla25-team12/operation/blob/main/provisioning/finalization.yml)

Excellent:

- Kubernetes Dashboard directly reachable at dashboard.local (or 192.168.56.91)
- Nginx Ingress Controller fixed IP: 192.168.56.91
- Istio IngressGateway fixed IP: 192.168.56.99
- HTTPS Nginx Ingress Controller with self-signed certs for Kubernetes Dashboard (https://dashboard.local) in [finalization.yaml](https://github.com/remla25-team12/operation/blob/main/provisioning/finalization.yml#L176)
  - Your browser may still throw a security warning because the TLS certs are not signed by LetsEncrypt or another reputable Certificate Authority.

# A3 Operate and Monitor Kubernetes

## A3.1 Kubernetes & Monitoring

### Kubernetes Usage - Excellent

- Model service location defined as [environment variable](https://github.com/remla25-team12/operation/blob/c432605a8a93291e5b3acb41a59e6bbb9b3ff0b4/helm/myapp/templates/app-deployment.yaml#L27) with variable names, so can be relocated just by changing Kubernetes config.
- [ConfigMap](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/grafana-dashboard-configmap.yaml) and [Secret](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/alertmanager-secret.yaml)

Excellent:

- /mnt/shared is mounted in [Vagrantfile](https://github.com/remla25-team12/operation/blob/cfdc4b9c7ed25955658588abf1637150ee66ebc4/Vagrantfile#L140)

### Helm - Excellent

- Helm chart: `./helm/myapp`, contains the complete deployment except Prometheus/Grafana/AlertManager stack (these are deployed separately as `myprom`)
- [values.yaml](https://github.com/remla25-team12/operation/blob/main/helm/myapp/values.yaml) (not .xml though)
- Can be installed more than once through the use of the "myapp.fullName" variable in all templates.

### App Monitoring - Excellent

Sufficient/Good:

- Our app-specific metrics are defined in [remla25-team12/app, app.py](https://github.com/remla25-team12/app/blob/main/app.py):
  - Counters:
    - total_reviews_submitted ('app-version' label)
    - total_correct_predictions
    - total_incorrect_predictions
    - profile_clicks ('member_name' and 'app_version' labels)
  - Histogram:
    - all_review_length_histogram ('app_version' label)
    - review_length_per_feedback_histogram ('prediction_outcome' label)
  - Gauge
    - current_percentage_of_correct_predictions_gauge ('model_version' label)
- Automatically discovered through a [ServiceMonitor](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/service-monitor.yaml)

Excellent:

- We use AlertManager and notifications successfully reach our GMail inbox: ![GMail inbox AlertManager](imgs/rubric_gmail_alertmanager.png)
- There is no password in the [AlertManager Secret](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/alertmanager-secret.yaml). The password must be passed as an environment variable. For the sake of being able to test our codebase, we have included the password to our Google account in base64 in the README.

### Grafana - Excellent

- No manual installation instructions, because we use a [Configmap](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/grafana-dashboard-configmap.yaml) for Grafana to install the [dashboard.json](https://github.com/remla25-team12/operation/blob/main/helm/myapp/grafana/dashboard.json) automatically.
- Gauges, Counters, variable timeframes for parameterizable queries, and rate/avg functions are all used.

# A4 ML Configuration Management

## A4.1 ML Testing

### Automated Tests - Good

Sufficient/Good:

- All 4 ML Test Score categories are applied and clearly labelled with comments in [run_tests.py](https://github.com/remla25-team12/model-training/blob/main/tests/run_tests.py)
- Non-determinism robustness test: [test_robustness.py](https://github.com/remla25-team12/model-training/blob/main/tests/test_robustness.py)
- Performance tests: [test_performance.py](https://github.com/remla25-team12/model-training/blob/main/tests/test_performance.py)
- Feature cost test : [test_feature_cost_importance](https://github.com/remla25-team12/model-training/blob/dbccc56ffbae17195fc71c9f518979d6b3400394/tests/test_aa_utils.py#L33)

Excellent:

- Coverage measured in [run_tests.py](https://github.com/remla25-team12/model-training/blob/main/tests/run_tests.py)
- Mutamorphic testing : [test_mutamorphic_with_synonym_replacement](https://github.com/remla25-team12/model-training/blob/dbccc56ffbae17195fc71c9f518979d6b3400394/tests/test_robustness.py#L69) and [test_synonym_metamorphic](https://github.com/remla25-team12/model-training/blob/dbccc56ffbae17195fc71c9f518979d6b3400394/tests/test_aa_utils.py#L61)

### Continuous Training - Excellent

- pytest and pylint in a [GitHub workflow called quality.yaml](https://github.com/remla25-team12/model-training/blob/main/.github/workflows/quality.yml)
- pylint score, test adequacy and test coverage all calculated and reported automatically in [README](https://github.com/remla25-team12/model-training/blob/main/README.md) with a badge.

## A4.2

### Project Organization - Excellent

Sufficient:

- Python project with pipeline scripts in [./src](https://github.com/remla25-team12/model-training/tree/main/src)
- [requirements.txt](https://github.com/remla25-team12/model-training/blob/main/requirements.txt)
- Loosely follows Cookiecutter template

Good:

- Dataset not stored on our GitHub repository ([./data](https://github.com/remla25-team12/model-training/tree/main/data) folder is empty)
- There is no exploratory code in our repository
- Only code required for training & evaluation is included in ./src
- Versioned [release](https://github.com/remla25-team12/model-training/releases) of the model

Excellent:

- [Automated](https://github.com/remla25-team12/model-training/blob/main/.github/workflows/release.yaml) versioned [release](https://github.com/remla25-team12/model-training/releases) of the model.

### Pipeline Management with DVC - Excellent

- Cloud remote storage (Google Drive) instructions provided in [README](https://github.com/remla25-team12/model-training/blob/main/README.md) and works on our side with our Google Account.
  - Due to authentication problems with Google and the fact that you do not have our password/2FA keys, it might not work for you.
  - We also included local remote storage instructions.
- `metrics.json` for accuracy metrics generated in [./src/evaluate.py](https://github.com/remla25-team12/model-training/blob/main/src/evaluate.py).
  - Registered in [eval_model stage](link to line in dvc.yaml).
  - Also reports non-accuracy metrics such as precision, recall, F1-score and support.

### Code Quality - Excellent

- Non-standard [.pylintrc]()
- No warnings (else our GitHub action for linting would fail)
- Multiple linters: black and flake8 defined in [setup.cfg](https://github.com/remla25-team12/model-training/blob/main/setup.cfg)
- Custom pylint rule for ML-Specific code smells: We relaxed the naming convention rules to accommodate commonly used variable names in ML pipelines, such as X, X_train, X_test, y, and y_pred.

# A5 Istio Service Mesh

## A5.1 Implementation

### Traffic Management - Excellent

- Accessible through IngressGateway after adding "myapp.local 192.168.56.99" to hostsfile.
- Gateway, VirtualServices, DestinationRules and weights for 90/10 split are defined in [istio.yaml](https://github.com/remla25-team12/operation/blob/main/helm/myapp/templates/istio.yaml)
- `app` and `model-service` versions are consistent.
- We implemented Sticky Sessions using illustrative curl requests with the header variable `x-newvers`, see [README](https://github.com/remla25-team12/operation/blob/main/README.md#sticky-sessions)

### Additional Use Case - Excellent

- We fully realized rate limiting with Istio: maximum 10 requests globally and maximum 2 model queries (/predict endpoint) per minute.

### Continuous Experimentation - Excellent

Sufficient/good:

- Experiment is described clearly: LinkedIn icon vs no LinkedIn icon, which generated more clicks?
- Both versions of `app` (with and without LinkedIn icon) are deployed with Istio and reachable through the traffic split.
- Prometheus picks up the metric `profile_clicks` automatically and it is visualized in Grafana in the automatically-imported dashboard.

Excellent:

- We explain the decision process in detail.

## A5.2 Documentation

### Deployment Documentation - Excellent

- In [deployment.md](https://github.com/remla25-team12/operation/blob/main/docs/deployment.md), we discuss both the deployment structure and the data flow on several levels (Kubernetes, Istio service mesh, HTTP requests).
- In our opinion, the documentation is indeed visually appearing and clear, with useful visualizations connected to the text.

### Extension Proposal - Excellent

- In [extension.md](https://github.com/remla25-team12/operation/blob/main/docs/extension.md), we analyze 2 shortcomings, critically reflect, present extensions to improve these shortcomings, and describe how an improvement can be measured in an experiment.
- In our opinion, the documentation is general and applicable beyond the current status of our project.

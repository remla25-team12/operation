# Critical Reflection and Extension Proposal

In our GitHub organization for our Restaurant Sentiment Analysis project, we are maintaining six repositories, namely [operation](https://github.com/remla25-team12/operation), [app](https://github.com/remla25-team12/app), [model-service](https://github.com/remla25-team12/model-service), [model-training](https://github.com/remla25-team12/model-training), [lib-ml](https://github.com/remla25-team12/lib-ml), and [lib-version](https://github.com/remla25-team12/lib-version).

We identified two main shortcomings in our project: manual Kubernetes provisioning and deployment, and local storage usage in DVC. Each shortcoming is described with respect to (1) current state analysis, (2) critical reflection, (3) proposed extension to overcome the shortcoming, (4) measurable improvement, and (5) validation experiment design.

## Shortcoming #1. Manual Kubernetes Provisioning and Deployment

### A. Current State Analysis

Related repository: [Operation](https://github.com/remla25-team12/operation)

Provisioning and deploying the application on a Kubernetes cluster is currently done manually by following the steps outlined in the repository's README. The manual work consists of manually executing each step in the Kubernetes provisioning and deployment process—ranging from VM setup with Vagrant, SSH access, Ansible playbook execution, and Istio configuration to Helm-based application deployment—by following a long sequence of commands described in the README.

### B. Critical Reflection

This manual process is time-consuming, error-prone, and discourages rapid iteration and testing by the developers of the system. Furthermore, as new integrations are made to our application, each new feature requires a new step to be added to the README, making our README very long and unreadable for new developers and users.

### C. Proposed Extension

We propose to create a set of bash scripts [1] for:

1. Automating cluster provisioning
2. Applying Kubernetes setup and app deployment steps in the correct order
3. Making sure that none of the steps are skipped

This automated Kubernetes provisioning and deployment approach will make the process less time-consuming, less error-prone, and encourage rapid iteration and testing [1].

### D. Measurable Improvement

To measure the impact of our introduced improvement on automating the Kubernetes provisioning and deployment, we can measure the following:

1. Comparing the average onboarding time for new developers to deploy our application for the first time. Measuring the onboarding time of these new developers is crucial because it might take them a lot more time to understand what is going on with our system when our README is crowded with manual setup steps. This onboarding time could be measured in minutes, in hours, or even in days.

2. Logging the frequency of deployment errors or missing steps in manual and automated runs. It is expected that the frequency of deployment errors will be significantly higher for the manual setup runs.

### E. Validation Experiment Design

**Objective:** To empirically evaluate whether scripting the provisioning and deployment of Kubernetes resources leads to faster onboarding and fewer errors compared to the current manual approach.

**Experiment Setup:** Collect 10 developers, who are new to our system. Divide them into two groups, where one group will follow the manual steps and the other will follow the automated steps. Ask both groups to:

1. Set up a Kubernetes cluster
2. Deploy the application using the specified method
3. Validate that the deployed app is running

**Measurement:** While conducting these steps, measure the onboarding time of the developers in minutes, hours, or days. Also, measure the number of failed attempts, misconfigurations, or missing steps during Kubernetes setup and provisioning.

## Shortcoming #2. DVC on Google Drive Blocked: Local Storage Fall-back

### A. Current State Analysis

**Related repository:** [Model-training](https://github.com/remla25-team12/model-training)

Our team originally used Google Drive as the DVC remote for our training pipeline. However, due to Google Drive's security policies, sending a high number of DVC pull and push requests from different IPs during the testing phase triggered an account blockage. This made our remote storage setup unsustainable. Hence, we had to fall back to using local storage and describing the steps in our README to ensure the reproduction of our approach.

### B. Critical Reflection

Cloud remotes are necessary for distributed collaboration, ensuring all users and developers have access to the same, latest trained model version. Furthermore, maintaining local storage is not ideal for CI/CD integration.

### C. Proposed Extension

We propose to select a different remote storage backend than Google Drive. As described in the official documentation of Data Version Control (DVC) [2], it is recommended to use Amazon S3 [3] or Azure Blob Storage [4] for this purpose.

The mentioned cloud services have their advantages and disadvantages. Hence, there is no certain preference towards using one type of remote storage backend over the other. While Amazon S3 shows high performance, strong scalability, and easy integration with other AWS services, Azure Blob Storage offers robust security, is cost-effective with tiered storage options, and integrates easily with the Microsoft Azure ecosystem [5].

For the purpose of this extension proposal, we choose to use Amazon S3 as our DVC remote storage.

### D. Measurable Improvement

To measure the impact of our introduced improvement on migrating to a shared storage from a local storage, we can measure the following:

1. Recording the latency until a new developer accesses the latest trained model with the current vs. proposed method. With local storage, it is expected that every time a new user joins, they have to run the entire pipeline via `dvc repro` and push it to the storage, whereas with the proposed method, it is expected that they only need to run `dvc pull` to access the model.

2. Counting the steps or errors encountered by a new developer while setting up the repository with the current vs. proposed method and while accessing the latest trained model from the storage.

### E. Validation Experiment Design

**Objective:** To evaluate whether migrating from local storage to Amazon S3 for DVC remote access improves latency, accessibility, and developer experience during onboarding and experimentation.

**Experiment Setup:** Divide the participants into two groups, where one group will follow the current method and the other will follow the proposed method. Here are the tasks that will be conducted by the participants:

- Pull the latest trained model using:
  - Current method: Local storage (dvc repro, then dvc push)
  - Proposed method: dvc pull from Amazon S3

**Measurement:** While conducting these steps, measure the time until the trained model is available locally (setup time), measure the number of steps that needs to be followed and the error frequency relating to authentication, missing data, or permission issues.

## References

[1] O. Kawonise, “Simplifying Kubernetes Installation on Ubuntu using a Bash Shell Script,” Medium, Aug. 27, 2023. [Online]. Available: https://medium.com/@olorunfemikawonise_56441/simplifying-kubernetes-installation-on-ubuntu-using-a-bash-shell-script-d75fed68a31

[2] DVC Team, “Remote storage,” DVC Documentation, Jun. 2025. [Online]. Available: https://dvc.org/doc/user-guide/data-management/remote-storage

[3] DVC Team, “Amazon S3,” DVC Documentation – Remote Storage, Jun. 2025. [Online]. Available: https://dvc.org/doc/user-guide/data-management/remote-storage/amazon-s3

[4] DVC Team, “Azure Blob Storage,” DVC Documentation – Remote Storage, Jun. 2025. [Online]. Available: https://dvc.org/doc/user-guide/data-management/remote-storage/azure-blob-storage

[5] A. Prakash, “Cloud Data Storage Deep Dive: S3, GCS, and Azure Blob Storage Compared,” Airbyte Data Engineering Resources, July 10, 2023. [Online]. Available: https://airbyte.com/data-engineering-resources/s3-gcs-and-azure-blob-storage-compared

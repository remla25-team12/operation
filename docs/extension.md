# Critical Reflection and Extension Proposal

In our GitHub organization for our Restaurant Sentiment Analysis project, we are maintaining six repositories, namely [operation](https://github.com/remla25-team12/operation), [app](https://github.com/remla25-team12/app), [model-service](https://github.com/remla25-team12/model-service), [model-training](https://github.com/remla25-team12/model-training), [lib-ml](https://github.com/remla25-team12/lib-ml), and [lib-version](https://github.com/remla25-team12/lib-version).

We identified two main shortcomings in our project: manual Kubernetes provisioning and deployment, and local storage usage in DVC. Each shortcoming is described with respect to (1) current state analysis, (2) critical reflection, (3) proposed extension to overcome the shortcoming, and (4) measurable improvement.

## Shortcoming #1. Manual Kubernetes Provisioning and Deployment

### A. Current State Analysis

Related repository: [Operation](https://github.com/remla25-team12/operation)

Provisioning and deploying the application on a Kubernetes cluster is currently done manually by following the steps outlined in the repository's README. This includes setting up the cluster, configuring namespaces, applying manifests, and potentially exposing services.

### B. Critical Reflection

This manual process is time-consuming, error-prone, and discourages rapid iteration and testing by the developers of the system. Furthermore, as new integrations are made to our application, each new feature requires a new step to be added to the README, making our README very long and unreadable for new developers and users.

### C. Proposed Extension

We propose to create a set of bash scripts for:

1. Automating cluster provisioning
2. Applying Kubernetes setup and app deployment steps in the correct order
3. Making sure that none of the steps are skipped

This automated Kubernetes provisioning and deployment approach will make the process less time-consuming, less error-prone, and encourage rapid iteration and testing.

### D. Measurable Improvement

TODO: Add a way of measuring this improvement

## Shortcoming #2. DVC on Google Drive Blocked: Local Storage Fall-back

### A. Current State Analysis

**Related repository:** [Model-training](https://github.com/remla25-team12/model-training)

Our team originally used Google Drive as the DVC remote for our training pipeline. However, due to Google Drive's security policies, sending a high number of DVC pull and push requests from different IPs during the testing phase triggered an account blockage. This made our remote storage setup unsustainable. Hence, we had to fall back to using local storage and describing the steps in our README to ensure the reproduction of our approach.

### B. Critical Reflection

Cloud remotes are necessary for distributed collaboration, ensuring all users and developers have access to the same, latest trained model version. Furthermore, maintaining local storage is not ideal for CI/CD integration.

### C. Proposed Extension

We propose to select a different remote storage backend than Google Drive. As described in the official documentation of Data Version Control (DVC) [2], it is recommended to use Amazon S3 or Azure Blob Storage for this purpose.

TODO: Select one of the options and explain why it would be a good fit.

### D. Measurable Improvement

To measure the impact of our introduced improvement, we can measure the following:

1. Recording the latency until a new developer accesses the latest trained model with the current vs. proposed method, because with local storage, it is expected that every time a new user joins, they have to run the entire pipeline via `dvc repro` and push it to the storage, whereas with the proposed method, it is expected that they only need to run `dvc pull` to access the model.

2. Counting the steps or errors encountered by a new developer while setting up the repository with the current vs. proposed method and while accessing the latest trained model from the storage.

## Validation Experiment Design

TODO

## References

[1] TODO: Add a reference for the first shortcoming

[2] DVC Team, “Remote storage,” DVC Documentation, Jun. 2025. [Online]. Available: https://dvc.org/doc/user-guide/data-management/remote-storage

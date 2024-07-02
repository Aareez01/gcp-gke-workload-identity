# GKE Workload Identity

#### This guide outlines the steps required to set up Workload Identity on a Google Kubernetes Engine (GKE) cluster. Workload Identity allows Kubernetes service accounts to act as IAM service accounts, providing a secure way to access Google Cloud resources.

## Steps
### Step 1: Create IAM Service Account

##### First, create a service account in IAM on GCP and assign the necessary roles/permissions. Ensure it has the roles/iam.workloadIdentityUser role.
### Step 2: Create or Update GKE Cluster
##### Create a New Cluster Enable Workload Identity and the GKE Metadata Server in the node pool configuration when creating the cluster:
```
gcloud container clusters create my-cluster \
    --location=us-central1-c \
    --workload-pool=my-project-id.svc.id.goog
```

#### To Update an Existing Cluster

##### Enable Workload Identity on an existing cluster:

```
gcloud container clusters update disearch-cluster \
    --location=us-central1-c \
    --workload-pool=my-project-id.svc.id.goog
```

### Step 3: Create or Update Node Pool with Metadata Server
#### To Create a New Node Pool

##### Enable the GKE Metadata Server when creating the node pool:

```
gcloud container node-pools create my-node-pool \
    --cluster=my-cluster \
    --region=us-central1 \
    --workload-metadata=GKE_METADATA
```

#### To Update an Existing Node Pool

##### Enable the GKE Metadata Server on an existing node pool:

```
gcloud container node-pools update my-node-pool \
    --cluster=my-cluster \
    --region=us-central1 \
    --workload-metadata=GKE_METADATA
```

### Step 4: Connect to the Cluster

#### Connect to the GKE cluster:

```
gcloud container clusters get-credentials my-cluster \
    --location=us-central1-c
```

### Step 5: Create Kubernetes Service Account

#### Create a Service Account resource using the following YAML:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    iam.gke.io/gcp-service-account: iam-service-account@my-project-id.iam.gserviceaccount.com
  name: gke-sa
  namespace: default
```

### Step 6: Bind IAM Policy

#### Grant the Kubernetes Service Account permission to impersonate the IAM service account:

```
gcloud iam service-accounts add-iam-policy-binding iam-service-account@my-project-id.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:my-project-id.svc.id.goog[default/gke-sa]"
```

### Step 7: Configure Kubernetes Resource

#### Mount the service account in your pod/deployment or any other Kubernetes resource by adding these lines in the spec section:

```
spec:
  serviceAccountName: gke-sa
  nodeSelector:
    iam.gke.io/gke-metadata-server-enabled: "true"
```

## Step 8: You're Good to Go!

Your application can now securely access GCP services.

# Skaffold Developer Journey Tutorial

## Introduction

### What is this project?

PLACEHOLDER

### What you'll learn

* How to develop on minikube locally and move to an enterprise managed kubernetes cluster with ease
* How to easily switch buildpack versions to comply with security demands

___

**Time to complete**: <walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>
Click the **Start** button to move to the next step.

## First steps

### Prepare the Environment
Create a project called `skaffold-tutorial` with a valid billing account in the Google Cloud [Manage Resources Console](https://console.cloud.google.com/cloud-resource-manager)

Run the start script to deploy a GKE cluster we will use later and install the latest version of Skaffold. Although Skaffold is already installed on cloud shell, we'll install the latest version for good measure.
```bash
chmod +x start.sh && ./start.sh
```

### Start a Minikube cluster

We'll use `minikube` as our local kubernetes cluster of choice.

Run:
```bash
minikube start
```

## Run the App to Minikube Using Skaffold

PLACEHOLDER

### Deploy the App
Run:
```bash
skaffold dev
```

**Important:** note the software versions under the `DETECTING` phase of the buildpack output. These will be important later.

Open a second terminal. In order to connect to the application to make sure its working, you need to start the minikube load balancer by running:
```bash
minikube tunnel
```

Open a third terminal. Find out the load balancer IP by recording the external IP as `EXTERNAL_IP` after running:
```bash
kubectl get service
```

Ensure the app is responding by running:
```bash
curl http://EXTERNAL_IP:8080
```

### Change the App and Trigger a Redeploy

In the Cloud Shell Editor, open `src/main/java/hello/HelloController.java` and change the return line to `return "Hello, Skaffold!"`.

Switch back to the first terminal and see that its rebuilding and redeploying the app.

Switch back to the third terminal and run the following command to watch until only one pod is running:
```bash
watch kubectl get pods
```

Once there is only one pod running, meaning the latest pod is deployed, see that your app changes are live:
```bash
curl http://EXTERNAL_IP:8080
```

### Updating Buildpacks

Perhaps the best benefit of buildpacks is that it reduces how much work developers need to do to patch their applications if the security team highlights a library vulnerability.

In Cloud Shell Editor, open `skaffold.yaml` and update the builder line to `gcr.io/buildpacks/builder:v1`, which will use the latest builder that has more up-to-date buildpacks.

Switch back to the first terminal and see that its rebuilding and redeploying the app.

**IMPORTANT:** compare the software versions under the `DETECTING` phase of the buildpack output to the ones you saw before. The builder is now using newer buildpack versions.

Switch back to the third terminal and run the following command to watch until only one pod is running:
```bash
watch kubectl get pods
```

Once there is only one pod running, meaning the latest pod is deployed, see that your app changes are live:
```bash
curl http://EXTERNAL_IP:8080
```

## Deploy the App to Enterprise GKE Cluster Using Skaffold

### Switch kubectl Context to Enterprise GKE Cluster

Switch your local kubectl context to the enterprise GKE cluster and get the latest credentials:
```bash
gcloud container clusters get-credentials skaffold-tutorial-cluster --region us-central1-a
```

See that kubectl is now configured to use the remote kubernetes cluster instead of minikube (denoted by the asterisk)
```bash
kubectl config get-contexts
```

### Deploy the App

Attempt to deploy the app by running:
```bash
skaffold dev --default-repo=gcr.io/skaffold-tutorial
```

**IMPORTANT:** the deployment will fail because the default GCE service account, inherited by the kubernetes worker instances deployed earlier, is not automatically granted permission to pull from the Google Cloud Storage bucket created when skaffold pushed the container image generated during the build process.

To give the default GCE service account permission to pull images from `gcr.io/skaffold-tutorial`, perform the following:
* In Cloud Shell, run `gcloud projects list` and get the PROJECT_NUMBER for the skaffold-tutorial project
* Open the [Google Cloud Storage Browser](https://console.cloud.google.com/storage/browser)
* Click on the storage bucket named `artifacts.skaffold-tutorial.appspot.com` and select the **Permissions** tab
* Click the **ADD** button
* In the **New members** field enter `PROJECT_NUMBER-compute@developer.gserviceaccount.com` and press Enter
* In the **Role** field enter `Storage Object Viewer` and select the appropriate role
* Click **Save**

Deploy the app again by running:
```bash
skaffold dev --default-repo=gcr.io/skaffold-tutorial
```

Switch back to the third terminal and run the following command to watch until only one pod is running:
```bash
watch kubectl get pods
```

Run the following command. Once an external IP is assigned to the service, record it as `EXTERNAL_IP`:
```bash
watch kubectl get service
```

See that your app changes are now live on an Internet-accessible IP:
```bash
curl http://EXTERNAL_IP:8080
```
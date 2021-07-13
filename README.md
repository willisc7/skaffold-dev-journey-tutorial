# Skaffold Developer Journey

## Local Development 

PLACEHOLDER DESCRIPTION

### Local App Development
* `minikube start`
* `git clone --depth 1 https://github.com/GoogleContainerTools/skaffold && cd skaffold/examples/buildpacks-java/`
* `skaffold dev`
  * Note the software versions under the `DETECTING` phase of the buildpack output. These will be important later.
* Open a second terminal and run `minikube tunnel` to start up a load balancer
* Open a third terminal and run `kubectl get service` to find out external IP
* `curl http://EXTERNAL_IP:8080` and see that the app is running
* `vim src/main/java/hello/HelloController.java`, change return line to `return "Hello, Skaffold!"`, and type `:wq` and press Enter
* Switch back to the first terminal and see that its rebuilding and redeploying the app
* Switch back to the third terminal and run `watch kubectl get pods` until you see only one pod running
* `curl http://EXTERNAL_IP:8080` and see that the app now includes your change

### 
* Deploy GKE cluster
* Create GCS bucket backing GCR repo (`artifacts.PROJECT_NAME.appspot.com`)
* Give default GCE service account (`PROJECT_ID-compute@developer.gserviceaccount.com`) **Storage Object Viewer** access to `artifacts.PROJECT_NAME.appspot.com`

#### Cloud Shell Tutorial

##### Updating Buildpacks
* `vim skaffold.yaml` and update the builder line to `gcr.io/buildpacks/builder:v1`, which will use the latest builder that has more up-to-date buildpacks 
* Switch back to the first terminal and see that its rebuilding and redeploying the app
  * Compare the software versions under the `DETECTING` phase of the buildpack output to the ones you saw before. The builder is now using newer buildpack versions.
* Switch back to the third terminal and run `watch kubectl get pods` until you see only one pod running
* `curl http://EXTERNAL_IP:8080` and see that the app now includes your change

##### Remote App Deployment
* `gcloud container clusters get-credentials CLUSTER_NAME --region REGION`
* `kubectl config get-contexts` to see that you're now configured to use the remote kubernetes cluster (denoted by the asterisk)
* `skaffold dev --default-repo=gcr.io/PROJECT_NAME`
* Switch back to the third terminal and run `kubectl get service` to find out the external IP
* `curl http://34.94.160.33:8080` and see that the app is now running with an Internet-accessible IP
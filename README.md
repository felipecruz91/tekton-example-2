
# Tekton tutorial

A simple CI/CD pipeline built with Tekton that clones a Git repo, builds a Dockerfile and uploads the resulting image to a registry.


## Installation 

Install Tekton Core Components

```bash 
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Install Tekton Dashboard
```bash
kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
```    

Assuming tekton-pipelines is the install namespace for the Dashboard, run the following command:

```
kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
```

Visit http://localhost:9097 to access the Tekton Dashboard.

           
## Run Locally

Clone the project

```bash
git clone https://github.com/felipecruz91/tekton-example-2
```

Go to the project directory

```bash
cd tekton-example-2
```

### Environment Variables

To run this project in a corporate environment, you will need to configure the proxy settings in two places:

- [skaffold-git.yaml](./pipeline-resources/skaffold-git.yaml#L13)

```yaml
- name: httpsProxy
  value: "<YOUR_CORPORATE_PROXY>"
```

- [build-docker-image-from-git-source.yaml](build-docker-image-from-git-source.yaml#L33)

```yaml
  - name: https_proxy
    value: "<YOUR_CORPORATE_PROXY>"
```

### Secrets

You must create a secret to push your image to your desired image registry:

```bash
kubectl create secret docker-registry regcred \
                    --docker-server=<your-registry-server> \
                    --docker-username=<your-name> \
                    --docker-password=<your-pword> \
                    --docker-email=<your-email>
```        

Create the pipeline resources such as the Git repository and the image:

```bash
kubectl apply -f pipeline-resources/
```

Create a service account that uses the `regcred` credentials to push the image to your registry:

```bash
kubectl apply -f tutorial-serviceaccount.yaml
```

Create a Task which defines the git input and image output introduced earlier:

```bash
kubectl apply -f build-docker-image-from-git-source.yaml
```

You are now ready for your first TaskRun!

A TaskRun binds the inputs and outputs to already defined PipelineResources, sets values for variable substitution parameters, and executes the Steps in the Task.

```bash
kubectl apply -f build-docker-image-from-git-source-task-run.yaml
```

A Pipeline defines an ordered series of Tasks that you want to execute along with the corresponding inputs and outputs for each Task.

```bash
kubectl apply -f tutorial-pipeline.yaml
```

Finally, run the pipeline:

```bash
kubectl apply -f tutorial-pipeline-run-1.yaml
```
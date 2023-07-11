---
layout: post
title: Automating Build and Deployment with Google Cloud Build and Kubernetes
categories:
- k8s
- linux
- bash
- github
tags:
- gcp
- cloudbuild
- gke
---
In modern software development workflows, automation plays a crucial role in streamlining the build and deployment processes. Google Cloud Build, combined with Kubernetes, offers a powerful solution for automating the building and deployment of applications on Google Cloud Platform. In this blog post, we'll explore a sample cloudbuild.yaml file and a Kubernetes manifest file that work together to automate the build and deployment of a project.
There are two repositories, one of the app source code and the other of the environment.
```bash
MyApp
├── client
│    └── Dockerfile
├── cloudbuild.yaml
└── server
    └── Dockerfile
env-repo
└── kubernetes.yaml
└── cloudbuild.yaml
```
# MyApp Repo
## The cloudbuild.yaml

The cloudbuild.yaml file is the configuration file used in Google Cloud Build to define the build steps and deployment process for your project. It contains a series of steps that are executed sequentially to build and deploy your application. Let's dive into the individual steps in the cloudbuild.yaml file:

1. **Pulling the Latest Client Image**: The first step is to pull the latest version of the client image from the registry. This step ensures that you are working with the most up-to-date client image.

2. **Building the Client Docker Image**: The second step involves building the client Docker image using the Dockerfile file located in the './client' directory. This file contains the instructions for building the client image. The built image is tagged with a specific version based on the commit SHA.

3. **Pulling the Latest Server Image**: Similar to the client image, this step pulls the latest version of the server image from the registry.

4. **Building the Server Docker Image**: In this step, the server Docker image is built using the './server' directory. The server image is also tagged with a specific version based on the commit SHA.

5. **Pushing the Client Docker Image**: After building the client image, it is pushed to the registry. This step ensures that the latest version of the client image is available for deployment.

6. **Pushing the Server Docker Image**: Following the client image, the server image is pushed to the registry. This step makes the latest version of the server image available for deployment.

7. **Cloning the Environment Repository**: This step clones the 'env-repo' repository, which contains environment configuration files for your project.

8. **Generating the Manifest**: The next step involves generating a new manifest file (kubernetes.yaml) by replacing the placeholder COMMIT_SHA in the template file (kubernetes.yaml.tpl) with the actual short commit SHA obtained during the build process.

9. **Pushing the Manifest to the Environment Repository**: Finally, the updated manifest file is pushed to the 'env-repo' repository. This step ensures that the manifest file reflecting the latest changes is available for deployment.

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
  args: ['-c', 'docker pull gcr.io/$PROJECT_ID/client:latest || exit 0']
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-f', './client/Dockerfile', '-t', 'gcr.io/$PROJECT_ID/client:$SHORT_SHA', './client', '--cache-from', 'gcr.io/$PROJECT_ID/client:latest']
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
  args: ['-c', 'docker pull gcr.io/$PROJECT_ID/server:latest || exit 0']
- name: 'gcr.io/cloud-builders/docker'
  args: ['build','-t','gcr.io/$PROJECT_ID/server:$SHORT_SHA','./server', '--cache-from', 'gcr.io/$PROJECT_ID/server:latest']
- name: 'gcr.io/cloud-builders/docker'
  args: ["push", "gcr.io/$PROJECT_ID/client:$SHORT_SHA"]
- name: 'gcr.io/cloud-builders/docker'
  args: ["push", "gcr.io/$PROJECT_ID/server:$SHORT_SHA"]
# [START cloudbuild-trigger-cd]
# This step clones the env repository
- name: 'gcr.io/cloud-builders/gcloud'
  id: Clone env repository
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    gcloud source repos clone env-repo && \
    cd env-repo && \
    git checkout test && \
    git config user.email $(gcloud auth list --filter=status:ACTIVE --format='value(account)')

# This step generates the new manifest
- name: 'gcr.io/cloud-builders/gcloud'
  id: Generate manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
     sed "s/COMMIT_SHA/${SHORT_SHA}/g" kubernetes.yaml.tpl > env-repo/kubernetes.yaml

# This step pushes the manifest back to the env repository
- name: 'gcr.io/cloud-builders/gcloud'
  id: Push manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    set -x && \
    cd env-repo && \
    git add kubernetes.yaml && \
    git commit -m "Deploying image for build:${SHORT_SHA}
    Built from commit ${COMMIT_SHA} of repository MyApp
    Author: $(git log --format='%an <%ae>' -n 1 HEAD)" && \
    git push origin test
```

## The Kubernetes Manifest

The Kubernetes manifest (kubernetes.yaml) defines the deployment, services, and other resources required to deploy and manage your application on a Kubernetes cluster. Let's explore the components defined in the Kubernetes manifest:

1. **Deployment**: The deployment section configures the deployment of your application's server and client containers. It specifies the desired number of replicas and sets the container image to be deployed, including the version based on the commit SHA.

2. **Service**: The service section defines a service that exposes the server and client containers within the Kubernetes cluster. It enables other services within the cluster to communicate with the deployed containers.

3. **Ingress**: The ingress section configures an ingress resource, which acts as an entry point for external traffic to access your application. It defines the rules for routing external traffic to the appropriate services, allowing users to access your application using a custom domain name.

4. **ConfigMap**: The ConfigMap section provides configuration data to your application as key-value pairs. It allows you to separate configuration from the application code and modify the configuration without redeploying the entire application.

5. **Additional Deployments and Services**: The manifest also includes additional deployments and services for Redis, which is used as a message broker for Celery tasks. These resources enable scalable and reliable task processing within your application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp-client
  name: myapp-client
  namespace: myapp-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-client
  template:
    metadata:
      labels:
        app: myapp-client
    spec:
      containers:
      - image: gcr.io/myproject/client:COMMIT_SHA
        name: client
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-client
  namespace: myapp-test
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: myapp-client
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp-test
  annotations: 
    cert-manager.io/issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - test.myapp.com
    secretName: myapp-tls
  rules:
  - host: test.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-client
            port:
              number: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-vars
  namespace: myapp-test
data:
  # ConfigMap data goes here
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-server
  labels:
    app: myapp-server
  namespace: myapp-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-server
  template:
    metadata:
      labels:
        app: myapp-server
    spec:
      # Server container spec goes here
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-server
  namespace: myapp-test
spec:
  ports:
  - port: 80
    targetPort: 8000
    protocol: TCP
  selector:
    app: myapp-server
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-backend
  namespace: myapp-test
  annotations: 
    cert-manager.io/issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.myapp.com
    secretName: myapp-server-tls
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-server
            port:
              number: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
  namespace: myapp-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
  namespace: myapp-test
spec:
  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    app: redis
```

# Environment Repo
## The `cloudbuild.yaml`

```
# [START cloudbuild-delivery]
steps:
- name: 'gcr.io/cloud-builders/kubectl'
  id: Deploy
  args:
  - 'apply'
  - '-f'
  - 'kubernetes.yaml'
  - '-n'
  - 'myapp-test'
  env:
  - 'CLOUDSDK_COMPUTE_REGION=us-west1-c'
  - 'CLOUDSDK_CONTAINER_CLUSTER=myapp'
```

The `steps` section specifies the sequence of actions to be executed during the deployment process. In this case, we have a single step defined for deployment. Let's dive into the details of this deployment step:

- `name: 'gcr.io/cloud-builders/kubectl'`: The `name` field specifies the name of the builder image to be used for this step. In this case, we are using the `kubectl` builder image, which provides the necessary tools for interacting with the Kubernetes cluster.

- `id: Deploy`: The `id` field provides an identifier for this step. It can be used to reference this step in subsequent steps or for logging purposes.

- `args`: The `args` field contains the command and arguments that will be executed by the builder image. In this case, we are using `kubectl` to apply a Kubernetes manifest file (`kubernetes.yaml`) to the cluster.

- `env`: The `env` field defines any environment variables that need to be set during the execution of this step. In this example, we are setting the `CLOUDSDK_COMPUTE_REGION` and `CLOUDSDK_CONTAINER_CLUSTER` variables to specify the target region and cluster for deployment.

### Deployment Process Overview

The deployment process described in the `cloudbuild.yaml` file can be summarized as follows:

1. The `gcr.io/cloud-builders/kubectl` builder image is used for executing the deployment step.

2. The `kubectl apply` command is invoked, indicating that we want to apply changes to the cluster.

3. The `-f` flag is used to specify the Kubernetes manifest file (`kubernetes.yaml`) that defines the desired state of the deployment.

4. The `-n` flag is used to specify the target namespace for the deployment (`myapp-test` in this case).

5. The `env` field sets the necessary environment variables for the deployment, including the target region and cluster.

# Conclusion

In this blog post, we explored the cloudbuild.yaml file and the Kubernetes manifest file that work together to automate the build and deployment process of a project on Google Cloud Platform. The cloudbuild.yaml file defines the build steps, including pulling, building, and pushing Docker images, as well as cloning the environment repository and generating the manifest. The Kubernetes manifest defines the deployment, services, and other resources required to deploy and manage the application on a Kubernetes cluster.

By automating the build and deployment process with Google Cloud Build and Kubernetes, you can ensure consistent and efficient deployments of your application, reducing manual effort and enabling faster iteration cycles.

# Example for 52°North Image and Container Label Specification

The [specification for image and Container labels](https://wiki.52north.org/Documentation/ImageAndContainerLabelSpecification) specifies a minimal set of labels required for

- images
- container
- volumes

Details can be found in our wiki. Please feel invited to provide comments.

This repository provides pratical instructions how to implement this specification.

# Instructions

* **Build the image**

  Just checkout this repository and perform the following command line:

  ```
  docker build -t 52n-label-test:latest --build-arg GIT_COMMIT=$(git rev-parse -q --verify HEAD) --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") .
  ```
* **Verify the labels**

  ```
  docker inspect --format='{{range $k, $v := .ContainerConfig.Labels}} {{- printf "%s = \"%s\"\n" $k $v -}} {{end}}' 52n-label-test:latest
  ```

  The result should look like this:

  ```
  maintainer = "Jürrens, Eike Hinderk <e.h.juerrens@52north.org>"
  org.opencontainers.image.authors = "Jürrens, Eike Hinderk <e.h.juerrens@52north.org>"
  org.opencontainers.image.created = "1952-08-23T11:09:31Z"
  org.opencontainers.image.description = "Example for labelling images and container following https://wiki.52north.org/Documentation/ImageAndContainerLabelSpecification"
  org.opencontainers.image.licenses = "GPL-3.0-or-later"
  org.opencontainers.image.ref.name = "52north/label-example-1.0.0"
  org.opencontainers.image.revision = "5da05aca171acf7fd70183271e347c31d87a5f19"
  org.opencontainers.image.title = "52°North Label Example Image"
  org.opencontainers.image.url = "https://github.com/52North/label-spec.git"
  org.opencontainers.image.vendor = "52°North GmbH"
  org.opencontainers.image.version = "1.0.0"
  ```

* **Start container and add the required labels**

  Check the required labels for running containers and add them to your run command:

  ```bash
  docker run \
    --detach \
    --name 52n-label-test \
    --label org.52north.contact="e.h.juerrens+52n-label-test-on-$(hostname -f)@52north.org" \
    --label org.52north.context="local testing" \
    --label org.52north.end-of-life="$(date -d '+1 hour' -u +"%Y-%m-%dT%H:%M:%SZ")" \
    52n-label-test:latest
  ```

  For a coloured output, use [jq](https://stedolan.github.io/jq/) to show the labels of the container:

  ```
  docker inspect 52n-label-test | jq -r '.[0].Config.Labels'
  ```

  ```json
  {
    "maintainer": "Jürrens, Eike Hinderk <e.h.juerrens@52north.org>",
    "org.52north.contact": "e.h.juerrens+52n-label-test-on-makatea@52north.org",
    "org.52north.context": "local testing",
    "org.52north.end-of-life": "1952-08-23T12:54:27Z",
    "org.opencontainers.image.authors": "Jürrens, Eike Hinderk <e.h.juerrens@52north.org>",
    "org.opencontainers.image.created": "2020-12-03T11:41:15Z",
    "org.opencontainers.image.description": "Example for labelling images and container following https://wiki.52north.org/Documentation/ImageAndContainerLabelSpecification",
    "org.opencontainers.image.licenses": "GPL-3.0-or-later",
    "org.opencontainers.image.ref.name": "52north/label-example-1.0.0",
    "org.opencontainers.image.revision": "5da05aca171acf7fd70183271e347c31d87a5f19",
    "org.opencontainers.image.title": "52°North Label Example Image",
    "org.opencontainers.image.url": "https://github.com/52North/label-spec.git",
    "org.opencontainers.image.vendor": "52°North GmbH",
    "org.opencontainers.image.version": "1.0.0"
  }
  ```

  At the end, **do not** forget to stop and remove the container, or it will print dots for ever:

  ```
  docker kill 52n-label-test && docker rm 52n-label-test


  ```
* **Start container using a label file**

Instead of specifying multiple `--label` flags on the command line, you can define your labels in a file and use Docker’s `--label-file` option. This approach can improve readability and maintainability, especially when dealing with many labels.

 Create a `labels.txt` file containing your labels:

Create a file named `labels.txt` with the following content:

```
org.52north.contact=e.h.juerrens+52n-label-test-on-$(hostname -f)@52north.org
org.52north.context=local testing
org.52north.end-of-life=$(date -d '+1 hour' -u +"%Y-%m-%dT%H:%M:%SZ")
```

 Start the container using the label file:

Run the following command to start the container and apply the labels from the file:

```bash
docker run \
  --detach \
  --name 52n-label-test \
  --label-file ./labels.txt \
  52n-label-test:latest
```

 Inspect the container labels:

You can inspect the labels using `jq` for a readable output:

```bash
docker inspect 52n-label-test | jq -r '.[0].Config.Labels'
```

 Cleanup:

After testing, stop and remove the container:

```bash
docker kill 52n-label-test && docker rm 52n-label-test
```


* **Using Docker Compose**

Docker Compose can be used to simplify container management and streamline the process of applying labels to containers. By defining services and their labels in a `docker-compose.yml` file, you can easily build, run, and manage containers with consistent configurations.

Below is an example `docker-compose.yml` file for a service named `label-test` using the same labels as in the previous examples:

```yaml
version: '3.8'

services:
  label-test:
    image: 52n-label-test:latest
    container_name: 52n-label-test
    labels:
      org.52north.contact: "e.h.juerrens+52n-label-test-on-${HOSTNAME}@52north.org"
      org.52north.context: "local testing"
      org.52north.end-of-life: "${END_OF_LIFE}"
    restart: unless-stopped
```

Before running the Compose file, you can export the `END_OF_LIFE` environment variable to set the label dynamically, for example:

```bash
export END_OF_LIFE=$(date -d '+1 hour' -u +"%Y-%m-%dT%H:%M:%SZ")
```

To build and run the service in detached mode, use the following command:

```bash
docker-compose up -d
```

To verify the labels applied to the running container, you can use:

```bash
docker inspect 52n-label-test | jq -r '.[0].Config.Labels'
```

When finished, stop and remove the services and containers with:

```bash
docker-compose down
```

# Kubernetes: Labels and Selectors

## Introduction to Labels and Selectors

In Kubernetes, labels are key-value pairs attached to objects such as pods, services, and deployments. They are used to organize, select, and manage resources efficiently. Selectors enable users to filter and query resources based on their labels, facilitating operations like grouping pods for services or deployments.

## Syntax Examples

Labels are defined as key-value pairs in YAML manifests. Keys and values must be strings.

Example label definitions:

```yaml
metadata:
  labels:
    app: frontend
    environment: production
    tier: backend
```

Selectors use label keys and values to filter resources. The most common selector is `matchLabels`, which matches resources with specific label key-value pairs.

Example selector:

```yaml
selector:
  matchLabels:
    app: frontend
    environment: production
```

## Pod and Service Example

A pod with labels:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: frontend
    environment: production
spec:
  containers:
  - name: frontend
    image: nginx
```

A service selecting pods with matching labels:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
    environment: production
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## Deployment Example with matchLabels and matchExpressions

A deployment specifying pod template labels and selector:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: api
    matchExpressions:
    - key: environment
      operator: In
      values:
      - production
      - staging
  template:
    metadata:
      labels:
        app: backend
        tier: api
        environment: production
    spec:
      containers:
      - name: backend
        image: backend-image:v1
```

## Common kubectl Commands with Label Selectors

- List pods with a specific label:

  ```
  kubectl get pods -l app=frontend
  ```

- List pods matching multiple labels:

  ```
  kubectl get pods -l app=frontend,environment=production
  ```

- Delete pods with a specific label:

  ```
  kubectl delete pods -l tier=backend
  ```

- Get services selecting pods with a label:

  ```
  kubectl get svc -l app=frontend
  ```
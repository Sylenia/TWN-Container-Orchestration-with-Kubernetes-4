# Deploying a Web Application in Kubernetes from a Private Docker Registry

## Project Overview
This demo project demonstrates how to deploy a web application in a Kubernetes cluster using images from a private Docker registry. We will configure a Kubernetes Secret to securely store Docker registry credentials, integrate the Secret into the application Deployment, and deploy the application image from the private Docker registry.

---

## Technologies Used
- **Kubernetes**
- **Helm**
- **AWS Elastic Container Registry (ECR)**
- **Docker**

---

## Key Concepts

### Why Use a Private Docker Registry?
- **Security**: Private registries ensure that your application images are protected and not publicly accessible.
- **Custom Images**: Use proprietary application images or images built specifically for your project.
- **Access Control**: Manage who can pull or modify images with credentials and permissions.

### Kubernetes Secret for Private Docker Registry
A Kubernetes Secret allows you to securely store sensitive data, such as Docker registry credentials, and make it accessible to your application pods without exposing it directly in your manifests.

### ImagePullSecrets
The `imagePullSecrets` field in a Kubernetes Deployment specifies which Secret Kubernetes should use to authenticate with the private Docker registry when pulling container images.

---

## Steps to Complete the Project

### Step 1: Set Up AWS ECR (Private Docker Registry)
1. **Authenticate with AWS ECR**:
   ```bash
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
   ```
2. Verify login:
   ```bash
   docker info | grep 'Registry'
   ```
3. Push your Docker image to the ECR repository:
   ```bash
   docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<repository_name>:<tag>
   ```

---

### Step 2: Create a Docker Registry Secret

#### Option 1: Create Secret from Config File
1. **Generate Base64 Encoded Config File**:
   ```bash
   cat ~/.docker/config.json | base64
   ```
2. **Create the Secret**:
   ```bash
   kubectl create secret generic my-registry-key \
   --from-file=.dockerconfigjson=/path/to/.docker/config.json \
   --type=kubernetes.io/dockerconfigjson
   ```

#### Option 2: Create Secret with Credentials
1. Use the following command to create a Secret directly:
   ```bash
   kubectl create secret docker-registry my-registry-key \
   --docker-server=<private_repo_url> \
   --docker-username=<username> \
   --docker-password=<password>
   ```
2. Verify the created Secret:
   ```bash
   kubectl get secrets
   ```

---

### Step 3: Configure Deployment to Use the Secret

#### Deployment YAML File
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-two
  labels:
    app: my-app-two
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app-two
  template:
    metadata:
      labels:
        app: my-app-two
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: my-app-two
        image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<repository_name>:<tag>
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```
1. Save the YAML file and deploy it:
   ```bash
   kubectl apply -f deployment.yaml
   ```
2. Verify that the pods are running:
   ```bash
   kubectl get pods
   ```

---

### Step 4: Working with Minikube (Optional)
1. Access Minikube’s Docker environment:
   ```bash
   minikube ssh
   ```
2. Copy Docker config.json from Minikube to your host:
   ```bash
   minikube cp /home/docker/.docker/config.json ~/.docker/config.json
   ```
3. Reuse this `config.json` to create the Kubernetes Secret.

---

## Best Practices
1. **Secure Secrets**:
   - Use RBAC to restrict access to Secrets.
   - Avoid embedding sensitive data directly into YAML files.
2. **Use ImagePullPolicy**:
   - Set `imagePullPolicy: Always` during development to ensure the latest image is pulled.
3. **Automate Credential Management**:
   - Use tools like AWS IAM roles or CI/CD pipelines to handle registry credentials securely.
4. **Monitor and Audit**:
   - Regularly monitor access logs for unauthorized registry usage.

---

## Wrapping It Up
Deploying images from a private Docker registry to Kubernetes might sound like a guarded vault mission, but it’s all about setting up a reliable handshake between Kubernetes and your registry. Once the secrets are in place and your deployments are correctly configured, you’ve got yourself a seamless workflow for secure image deployments. Let’s keep building smarter and safer! 🚀🚀🚀

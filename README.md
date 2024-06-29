# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Dependencies
#### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster
4. `helm` - apply Helm Charts to a Kubernetes cluster

#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

# Basic Deployment
## Database setup
Set up the PostgreSQL database on Kubernetes cluster and populate it with data by running the following commands:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install sql-service bitnami/postgresql --set primary.persistence.enabled=false
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default sql-service-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
sh run_sql.sh
```

## Docker
Build and push Dockerfile for the Python application

```
docker build -t coworking:latest .
docker tag coworking:latest <ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/coworking:latest
docker push <ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/coworking:latest
```

## AWS CodeBuild Pipeline
Configure a build pipeline using AWS CodeBuild to build and push the Docker image into AWS ECR with buildspec.yml file defining building steps.
## EKS Deployment
Deploy changes to Kubernetes cluster with kubectl apply -f deployment/ command. This will make the API available within the cluster.
## API Logs
Verify the deployment with kubectl logs <POD_NAME> or the corresponding logs in AWS CloudWatch.

# Release process
1. Commit your code changes to the GitHub repository.
2. Automatically trigger the AWS CodeBuild pipeline on GitHub push action to build and push a new Docker image to AWS ECR. Image is built and pushed with an $IMAGE_TAG defined as an environment variable in the AWS CodeBuild configuration so it should be changed accordingly.
3. Update the Kubernetes deployment files with the new image tag to apply the latest changes to the EKS cluster. Use the kubectl kubectl apply -f deployment/ command.
4. Monitor the deployment and application logs in AWS CloudWatch.
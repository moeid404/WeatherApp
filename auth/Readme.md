# Authentication Service (Auth)

This directory contains the Kubernetes manifests for deploying and managing the **Authentication Service** for the WeatherApp project. The Authentication Service handles user authentication and connects to a MySQL database for storing user-related data. 

## Files Overview

- `deployment.yml` - Deploys the Authentication Service as a Kubernetes Deployment.
- `service.yml` - Exposes the Authentication Service within the cluster as a `ClusterIP` Service.
- `mysql/` - Contains Kubernetes resources to set up and manage the MySQL database, including:
  - `headless-service.yml` - Creates a headless service for MySQL, allowing stable network identity for the StatefulSet.
  - `init-job.yml` - Initializes the MySQL database with required tables and user accounts using a Kubernetes Job.
  - `statefulset.yml` - Deploys MySQL as a StatefulSet for persistent storage and reliable connections.

## Kubernetes Resources

### Authentication Service (Auth)

#### `deployment.yml`

Defines the **Authentication Service** deployment. Key configurations include:
- **Image**: `afakharany/weatherapp-auth:lab`
- **Replicas**: 1
- **Environment Variables**:
  - `DB_HOST`: Hostname of the MySQL service (set to `mysql`).
  - `DB_USER`: Username for MySQL (set to `authuser`).
  - `DB_PASSWORD`: Retrieved from `mysql-secret`, providing secure access.
  - `DB_NAME`: Database name (set to `weatherapp`).
  - `DB_PORT`: MySQL port (`3306`).
  - `SECRET_KEY`: Secret key for the service, retrieved from `mysql-secret`.

#### `service.yml`

Defines a **ClusterIP Service** for the Authentication Service:
- **Port**: 8080, exposing the Authentication Service within the Kubernetes cluster.
- **Selector**: Matches the labels in the deployment to route traffic to the Authentication Service pod.

### MySQL Database Setup

The `mysql/` directory includes resources for setting up the MySQL database.

#### `headless-service.yml`

A **headless service** for MySQL, which provides a stable network identity but no load balancing:
- **ClusterIP**: Set to `None` to create a headless service.
- **Port**: 3306 (MySQL default port).

#### `init-job.yml`

An **init-job** that runs a one-time database initialization for MySQL:
- **Image**: `mysql:5.7`, used to execute SQL commands for setup.
- **Environment Variables**: 
  - `MYSQL_ROOT_PASSWORD` and `MYSQL_AUTH_PASSWORD` are retrieved from the `mysql-secret`.
  - Creates the `weatherapp` database, an `authuser` user, and assigns necessary privileges.

#### `statefulset.yml`

Defines a **StatefulSet** for the MySQL database:
- **Replicas**: 1 (single MySQL instance for this deployment).
- **VolumeClaimTemplate**: Requests a persistent storage volume of 5Gi to ensure data durability.
- **Environment Variable**:
  - `MYSQL_ROOT_PASSWORD` is sourced from `mysql-secret` to secure the root account.

## Secrets

The `mysql-secret` Kubernetes secret is referenced in the manifests to securely manage sensitive information, such as database credentials and keys. Ensure this secret is created before deploying the services.

### Example of Creating the Secret

To create the `mysql-secret` secret, use the following command (replace the placeholders with actual values):

```bash
kubectl create secret generic mysql-secret \
  --from-literal=root-password=<root_password> \
  --from-literal=auth-password=<auth_password> \
  --from-literal=secret-key=<secret_key>
```
## Usage
- **Deploy the MySQL Resources:**

```bash
kubectl apply -f mysql/headless-service.yml
kubectl apply -f mysql/statefulset.yml
kubectl apply -f mysql/init-job.yml
```

- **Deploy the Authentication Service:**
```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

Ensure that all dependencies (like mysql-secret) are created before deploying these manifests.
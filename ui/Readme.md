# User Interface (UI) Service

This directory contains the Kubernetes manifests for deploying and managing the **User Interface (UI) Service** of the WeatherApp project. The UI service provides a front-end interface for users to interact with the application and communicates with the backend services (Authorization and Weather Client) for data retrieval.

## Files Overview

- `deployment.yml` - Deploys the UI service as a Kubernetes Deployment.
- `service.yml` - Exposes the UI service within the cluster as a `ClusterIP` Service.
- `ingress.yml` - Configures an Ingress resource to expose the UI service to external users over HTTPS.
- `tls.crt` and `tls.key` - Self-signed SSL certificate and key files for securing the Ingress.

## Kubernetes Resources

### User Interface (UI) Service

#### `deployment.yml`

Defines the **UI service** deployment. Key configurations include:
- **Image**: `afakharany/weatherapp-ui:lab`
- **Replicas**: 2, to ensure high availability of the UI.
- **Environment Variables**:
  - `AUTH_HOST` and `AUTH_PORT`: Specifies the hostname and port for the Authentication Service (`weatherapp-auth:8080`).
  - `WEATHER_HOST` and `WEATHER_PORT`: Specifies the hostname and port for the Weather Client service (`weatherapp-weather:5000`).
- **Ports**:
  - **Container Port**: 3000, where the UI service listens for incoming requests.
- **Health Checks**:
  - **Liveness Probe** and **Readiness Probe**: Check the `/health` endpoint to ensure the application is running and ready to receive traffic.

#### `service.yml`

Defines a **ClusterIP Service** for the UI service:
- **Port**: 3000, exposing the UI service within the Kubernetes cluster.
- **Selector**: Matches the labels in the deployment to route traffic to the UI service pods.

### Ingress

#### `ingress.yml`

Configures an **Ingress resource** to provide external access to the UI service over HTTPS:
- **Ingress Class**: `nginx`, specifies the ingress controller to use.
- **TLS**:
  - Uses a self-signed SSL certificate (`weatherapp-ui-tls`) for secure HTTPS access.
  - Hosts the service on `weatherapp.local` (replace with your domain if needed).
- **Rules**:
  - Routes all traffic hitting the root path (`/`) on `weatherapp.local` to the `weatherapp-ui` service on port 3000.

#### Generating a Self-Signed SSL Certificate

A self-signed SSL certificate is used to enable HTTPS for the Ingress. The certificate and key were generated with the following command:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=example.com/O=example.com"
```

This command generates:
- **`tls.crt`**: The certificate file.
- **`tls.key`**: The private key file.

### Creating the TLS Secret

To use the certificate and key with the Ingress, create a Kubernetes secret named `weatherapp-ui-tls`:

```bash
kubectl create secret tls weatherapp-ui-tls --cert=tls.crt --key=tls.key
```

## Usage

**Deploy the UI Resources**:

   ```bash
   kubectl apply -f deployment.yml
   kubectl apply -f service.yml
   kubectl apply -f ingress.yml
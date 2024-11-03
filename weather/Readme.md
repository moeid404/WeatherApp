# Weather Service

This directory contains the Kubernetes manifests for deploying and managing the **Weather Service** for the WeatherApp project. The Weather Service fetches real-time weather data from an external weather API and makes it accessible to other services within the application.

## Files Overview

- `deployment.yml` - Deploys the Weather Service as a Kubernetes Deployment.
- `service.yml` - Exposes the Weather Service within the cluster as a `ClusterIP` Service.

## Kubernetes Resources

### Weather Service

#### `deployment.yml`

Defines the **Weather Service** deployment. Key configurations include:
- **Image**: `afakharany/weatherapp-weather:lab`
- **Replicas**: 2, to ensure high availability of the Weather Service.
- **Environment Variables**:
  - `APIKEY`: The API key for accessing the external weather API, retrieved from a Kubernetes secret (`weather` secret).
- **Ports**:
  - **Container Port**: 5000, where the Weather Service listens for incoming requests.
- **Health Checks**:
  - **Liveness Probe** and **Readiness Probe**: Check the root path (`/`) to ensure the application is running and ready to receive traffic.

#### `service.yml`

Defines a **ClusterIP Service** for the Weather Service:
- **Port**: 5000, exposing the Weather Service within the Kubernetes cluster.
- **Selector**: Matches the labels in the deployment to route traffic to the Weather Service pods.

### Setting Up the API Key

The `APIKEY` environment variable for the Weather Service is retrieved from a Kubernetes secret. Before deploying the Weather Service, make sure to create this secret with the API key for the weather API.

#### Creating the API Key Secret

To create the `weather` secret with the API key, use the following command (replace `<your_api_key>` with the actual API key):

```bash
kubectl create secret generic weather --from-literal=apikey=<your_api_key>
```

## Usage

1. **Deploy the Weather Resources**:
   ```bash
   kubectl apply -f deployment.yml
   kubectl apply -f service.yml
   ```

2. **Accessing the Weather Service**:
   - The Weather Service is exposed internally within the Kubernetes cluster, allowing other services to access it on port 5000.

## Updating the Hosts File

To access the UI service externally through the `weatherapp.local` domain, you need to update your `/etc/hosts` file. This maps `weatherapp.local` to `127.0.0.1`.

1. Open the `/etc/hosts` file in a text editor (e.g., `vim` or `nano`):
   ```bash
   sudo vim /etc/hosts
   ```

2. Add the following line to the file:
   ```plaintext
   127.0.0.1 weatherapp.local
   ```

3. Save and close the file.

This change will route requests to `https://weatherapp.local` to your local Kubernetes cluster.
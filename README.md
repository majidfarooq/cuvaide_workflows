# CuvAide API: Deployment Guide

This guide provides instructions for running the CuvAide API locally and deploying it to a production environment.

## 1. Prerequisites

Before you begin, ensure you have the following tools installed on your machine:

* **Git**
* **Docker** and **Docker Compose**
* **Ruby** (version X.X.X)
* **Bundler** (`gem install bundler`)
* **Kamal** (`gem install kamal`) - Only required for local-to-prod deployment.
* **Azure CLI** (`az`) - Only required for Azure deployment.

You will also need access to the following non-public resources:

* The **"Dev keys"** and **"Prod keys"** vaults in **1Password**.
* **Read/Write access** to this GitHub repository.
* **SSH access** to the production server (for Kamal deployment).
* An **Azure account** with permissions to the `trajan` resource group and `cuvaideapistaging.azurecr.io` ACR.

---

## 2. Running Locally

This section outlines how to set up and run the API on your local machine for development.

1.  **Clone the Repository:**
    ```bash
    git clone [repository-url]
    cd cuvaide-api
    ```
2.  **Environment Variables:**
    * Create a `.env.dev` file by copying the example file: `cp .env.example .env.dev`
    * Populate the variables in `.env.dev` with values from the **"Dev keys"** vault in **1Password**.
3.  **Install Dependencies:**
    ```bash
    bundle install
    ```
4.  **Start Services (Database & Redis):**
    ```bash
    docker compose up -d
    ```
5.  **Database Setup:**
    ```bash
    bin/rails db:prepare
    ```
    This command will create the database, run any pending migrations, and seed the data if necessary.
6.  **Start the Rails Server:**
    ```bash
    bin/rails s
    ```
    The API will now be running at `http://localhost:3000`.

* For more detailed development information, please refer to the **Dev Guide**.

---

## 3. Deploying to Production

There are three ways to deploy this application to a production environment. The **GitHub Actions** method is the recommended approach.

### Method A: Deploying from GitHub Actions (Recommended)

This method uses the automated CI/CD workflow to deploy the application.

1.  Navigate to the **Actions** tab in this repository on GitHub.
2.  Click on the **Deploy to Prod** workflow from the list on the left.
3.  Click the **Run workflow** button.
4.  Select the branch you wish to deploy (e.g., `main`).
5.  Click **Run workflow** to initiate the deployment.

### Method B: Deploying from a Local Dev Machine (using Kamal)

This method requires **Kamal** to be installed and configured on your local machine, along with SSH access to the production server.

1.  **Environment Variables:**
    * Create a `.env.prod` file by copying the example file: `cp .env.example .env.prod`
    * Populate the variables in `.env.prod` with values from the **"Prod keys"** vault in **1Password**.
2.  **Run the Deployment:**
    * Set the production environment variables in your terminal session:
        ```bash
        set -a
        source .env.prod
        set +a
        ```
    * **For the first deployment to a new server**, run `kamal setup`.
    * **For all subsequent deployments**, run `kamal deploy`.

### Method C: Deploying to Azure

This method involves building and pushing a Docker image to Azure's Container Registry (ACR) and then updating the Container App.

1.  **Log in to Azure and ACR:**
    ```bash
    az login
    az acr login --name cuvaideapistaging
    ```
2.  **Build the Docker Image:**
    ```bash
    docker build . --platform linux/amd64 -t cuvaideapistaging.azurecr.io/cuvaideapistaging:latest
    ```
3.  **Push the Image to ACR:**
    ```bash
    docker push cuvaideapistaging.azurecr.io/cuvaideapistaging:latest
    ```
4.  **Update the Azure Container App:**
    ```bash
    az containerapp update \
      --name cuvaide-api-staging \
      --resource-group trajan \
      --image cuvaideapistaging.azurecr.io/cuvaideapistaging:latest
    ```
    * **Note:** If this is the **first-time deployment**, you will need to create the container app first using `az containerapp create`.

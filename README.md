# Lesson Plan: Deploying Nginx on Kubernetes using UDS CLI

**Objective:**  
By the end of this 30-minute workshop, participants will have deployed an Nginx server locally using Kubernetes and UDS CLI, following a structured deployment process.

---

**Duration:** 30 minutes

---

## Workshop Agenda

### **1. Introduction (5 minutes)**

- **Objective:** Briefly introduce the participants to the tools they will be using: Kubernetes, UDS CLI, and Nginx.
- **Topics:**
  - Overview of Kubernetes
  - Introduction to UDS CLI as a deployment tool
  - Explanation of Nginx as a lightweight web server for this exercise

### **2. Setup Environment (5 minutes)**

- **Objective:** Ensure all participants have the necessary tools installed and set up.
- **Steps:**
  - **Install UDS CLI:**
    - Command: `brew tap defenseunicorns/tap && brew install uds`
  - **Ensure Docker is Running:** Verify Docker or Colima is running.
  - **Install K3d (if not already installed):**
    - Command: `brew install k3d`

### **3. Create and Configure Nginx Zarf Package (10 minutes)**

- **Objective:** Participants will create a Zarf package for Nginx and configure it for deployment.
- **Steps:**
  - **Create Directory for Nginx Package:**
    - Command: `mkdir nginx-package && cd nginx-package`
  - **Create `zarf.yaml` File:**
    - Content:

      ```yaml
      kind: ZarfPackageConfig
      metadata:
        name: nginx
        version: 1.0.0

      components:
        - name: nginx
          required: true
          charts:
            - name: nginx
              version: 18.1.9
              namespace: nginx
              url: https://charts.bitnami.com/bitnami
              gitPath: charts/nginx
          images:
            - docker.io/bitnami/nginx:1.27.1-debian-12-r0
          actions:
            onDeploy:
              after:
                - wait:
                    cluster:
                      kind: deployment
                      name: nginx
                      namespace: nginx
                      condition: available
      ```

  - **Create Zarf Package:**
    - Command: `zarf package create --confirm`

### **4. Create UDS Bundle for Nginx Deployment (5 minutes)**

- **Objective:** Configure a UDS Bundle to deploy the Nginx package.
- **Steps:**
  - **Create UDS Bundle Configuration:**
    - Create `uds-bundle.yaml` in the same directory.
    - Content:

      ```yaml
      kind: UDSBundle
      metadata:
        name: nginx-bundle
        description: Bundle with k3d, Zarf init, UDS Core, and nginx.
        version: 1.0.0

      packages:
        - name: uds-k3d
          repository: ghcr.io/defenseunicorns/packages/uds-k3d
          ref: 0.8.0

        - name: init
          repository: oci://ghcr.io/zarf-dev/packages/init
          ref: v0.38.2

        - name: core
          repository: oci://ghcr.io/defenseunicorns/packages/uds/core
          ref: 0.25.2-upstream

        - name: nginx
          path: ./
          ref: 1.0.0
      ```

### **5. Deploy the UDS Bundle (5 minutes)**

- **Objective:** Deploy the Nginx server on a local Kubernetes cluster using UDS CLI.
- **Steps:**
  - **Create the UDS Bundle:**
    - Command: `uds create --confirm`
  - **Deploy the UDS Bundle:**
    - Command: `uds deploy uds-bundle-nginx-bundle-arm64-1.0.0.tar.zst --confirm`

### **6. Interact with the Deployed Nginx Server (3 minutes)**

- **Objective:** Verify the deployment and interact with the Nginx server.
- **Steps:**
  - **Verify Pods are Running:**
    - Command: `kubectl get pods -n nginx`
  - **Port-Forward to Access Nginx Locally:**
    - Command: `kubectl port-forward pod/nginx-<pod-name> 8080:80 -n nginx`
  - **Access Nginx in Browser:**
    - URL: `http://localhost:8080`
  - **Install k9s:**
    -   **Via Homebrew**
         `brew install derailed/k9s/k9s`

### **7. Q&A and Wrap-Up (2 minutes)**

- **Objective:** Address any questions and summarize key points.
- **Discussion Points:**
  - Importance of understanding the deployment process
  - How UDS simplifies Kubernetes deployments
  - Potential next steps: exploring additional features or more complex deployments

---

**Resources:**

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Nginx Official Site](https://www.nginx.com/)
- [UDS CLI Documentation](https://github.com/defenseunicorns/uds-cli)

This lesson plan is designed to be interactive and hands-on, with participants actively following along and deploying Nginx in real-time.

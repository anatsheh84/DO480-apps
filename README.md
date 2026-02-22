# DO480-apps

Sample applications and Kubernetes manifests used in the **DO480 – Multicluster Management with Red Hat OpenShift Platform Plus** course. The repo covers two main themes: deploying a multi-tier application with MySQL on OpenShift, and managing application deployments via **Red Hat Advanced Cluster Management (RHACM)** with images sourced from **Quay**.

---

## Repository Structure

| Directory | Key Concept |
|-----------|-------------|
| [`mysql`](./mysql) | Two-tier To-Do app (Node.js frontend + MySQL backend) deployed on OpenShift |
| [`quayacm-deployment`](./quayacm-deployment) | RHACM-managed `budget-app` deployment exercise (dev namespace) |
| [`quayacm-review`](./quayacm-review) | RHACM-managed `budget-app` review/final exercise (production namespace) |

---

## App Details

### `mysql` — Two-Tier To-Do Application

A complete two-tier application consisting of a **Node.js frontend** (`todonodejs`) and a **MySQL 8.0 backend**, deployed across separate manifests. Demonstrates multi-component deployments, environment variable injection, and persistent-ish storage using `emptyDir`.

**Components:**

| Manifest | Resource | Image | Port |
|---|---|---|---|
| `deployment.yaml` | MySQL backend | `registry.redhat.io/rhel8/mysql-80:1-152` | 3306 |
| `deployment-frontend.yaml` | Node.js frontend | `quay.io/redhattraining/todo-single:v1.0` | 8080 |
| `service.yaml` | MySQL ClusterIP Service | — | 3306 |
| `service-frontend.yaml` | Frontend ClusterIP Service | — | 8080 |
| `route.yaml` | OpenShift Route | — | exposes frontend |

**Environment variables — MySQL backend:**

| Variable | Value |
|---|---|
| `MYSQL_ROOT_PASSWORD` | `r00tpa55` |
| `MYSQL_USER` | `user1` |
| `MYSQL_PASSWORD` | `mypa55` |
| `MYSQL_DATABASE` | `items` |

**Environment variables — Node.js frontend:**

| Variable | Value |
|---|---|
| `MYSQL_ENV_MYSQL_DATABASE` | `items` |
| `MYSQL_ENV_MYSQL_USER` | `user1` |
| `MYSQL_ENV_MYSQL_PASSWORD` | `mypa55` |
| `APP_PORT` | `8080` |

**Deploy:**
```bash
oc new-project mysql
oc apply -f mysql/deployment.yaml
oc apply -f mysql/service.yaml
oc apply -f mysql/deployment-frontend.yaml
oc apply -f mysql/service-frontend.yaml
oc apply -f mysql/route.yaml
```

> **Note:** MySQL uses `emptyDir` for storage — data does not persist across pod restarts. This is intentional for course lab purposes.

---

### `quayacm-deployment` — RHACM Budget App (Development)

OpenShift/RHACM manifests for deploying a `budget-app` into the **`budget-app-dev`** namespace as part of an ACM-managed application deployment exercise. The image placeholder `IMAGE_NAME` is meant to be replaced with an image pulled from a **Quay** registry as part of the lab workflow.

| Manifest | Kind | Description |
|---|---|---|
| `application.yaml` | `Application` (ACM) | ACM Application object targeting the `budget-app-dev` namespace |
| `deployment.yaml` | `Deployment` | `budget-app` deployment — replace `IMAGE_NAME` with your Quay image |
| `service.yaml` | `Service` | ClusterIP service on port 8080 |
| `route.yaml` | `Route` | OpenShift Route to expose the app |

**Deploy:**
```bash
# Replace IMAGE_NAME with your actual Quay image first
oc new-project budget-app-dev
oc apply -f quayacm-deployment/
```

---

### `quayacm-review` — RHACM Budget App (Review / Production)

Similar to `quayacm-deployment` but targets the **`budget-app`** namespace. Used in the final course review exercise to verify end-to-end ACM application lifecycle management with Quay as the image registry.

| Manifest | Kind | Description |
|---|---|---|
| `application.yaml` | `Application` (ACM) | ACM Application object targeting the `budget-app` namespace |
| `deployment.yaml` | `Deployment` | `budget-app` deployment — replace `IMAGE_NAME` with your Quay image |
| `service.yaml` | `Service` | ClusterIP service on port 8080 |

**Deploy:**
```bash
# Replace IMAGE_NAME with your actual Quay image first
oc new-project budget-app
oc apply -f quayacm-review/
```

---

## Prerequisites

- Red Hat OpenShift cluster managed by **Red Hat Advanced Cluster Management (RHACM)**
- Access to a **Quay** registry with the relevant `budget-app` image (for the `quayacm-*` exercises)
- `oc` CLI installed and logged in
- Access to `registry.redhat.io` (pull secret configured) for the MySQL image

---

## License

This project is licensed under the [Apache 2.0 License](./LICENSE).

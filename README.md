---

# Helm Chart Versioning with GitHub Pages & Deployment via Argo CD

This guide demonstrates how to:

* Package **multiple versions of a Helm chart**
* Host them on **GitHub Pages as a Helm repository**
* Deploy the chart using **Argo CD**

---

## Prerequisites

* Kubernetes cluster
* `kubectl` configured
* `helm` installed
* `git` installed
* GitHub repository with **GitHub Pages enabled**
* Helm chart directory (example: `appnginx/`)

---

## Repository Structure

```
.
├── appnginx/          # Helm chart source
├── packages/          # Packaged chart versions (tgz)
└── README.md
```

---

## Step 1: Create a Release Branch

Create and switch to a dedicated branch for Helm releases:

```bash
git checkout -b helm-releases
```

---

## Step 2: Lint the Helm Chart

Validate the Helm chart before packaging:

```bash
helm lint appnginx/
```

---

## Step 3: Package Initial Chart Version

Create a directory to store packaged charts:

```bash
mkdir packages
```

Package the Helm chart:

```bash
helm package appnginx -d packages/
```

Generate the Helm repository index:

```bash
helm repo index packages --url https://shehbab-kakkar.github.io/appnginx/packages
```

Commit and push the first release:

```bash
git add .
git commit -m "Package release 0.1.1"
git push --set-upstream origin helm-releases
```

---

## Step 4: Add More Chart Revisions (Multiple Versions)

Each new version requires updating `Chart.yaml` inside `appnginx/`
(example: `version: 0.1.2`)

Re-package the chart:

```bash
helm package appnginx -d packages/
```

Merge the new version into the existing Helm index:

```bash
helm repo index packages --merge packages/index.yaml
```

Commit and push the new release:

```bash
git add .
git commit -m "Package release 0.1.2"
git push origin helm-releases
```

✅ This preserves **older versions** while adding new ones.

---

## Step 5: Enable GitHub Pages

In your GitHub repository:

1. Go to **Settings → Pages**
2. Select:

   * Branch: `helm-releases`
   * Folder: `/root` (or `/packages` if configured)
3. Save

Your Helm repo URL becomes:

```
https://shehbab-kakkar.github.io/appnginx/packages
```

---

## Step 6: Install Argo CD

Apply Argo CD manifests:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Change Argo CD service type to NodePort:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

Retrieve initial admin password:

```bash
kubectl get secret -n argocd argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d && echo
```

Port-forward Argo CD UI:

```bash
kubectl port-forward -n argocd svc/argocd-server 8080:443 --address=0.0.0.0 &
```

Access UI at:

```
https://localhost:8080
```

---

## Step 7: Deploy Helm Chart Using Argo CD

In Argo CD UI:

* **Repository URL**

  ```
  https://shehbab-kakkar.github.io/appnginx
  ```

* **Chart**

  ```
  appnginx
  ```

* **Chart Version**

  ```
  0.1.1 or 0.1.2
  ```

* **Helm Repository URL**

  ```
  https://shehbab-kakkar.github.io/appnginx/packages
  ```

Argo CD will now:

* Track Helm chart versions
* Allow rollbacks
* Deploy selected versions declaratively

---

## Summary

✔ Helm chart packaged with multiple versions
✔ Hosted as a Helm repo on GitHub Pages
✔ Versioned deployments via Argo CD
✔ Easy rollback and GitOps workflow

---

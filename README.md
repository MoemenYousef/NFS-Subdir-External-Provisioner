### ✅ NFS Subdir External Provisioner with Kubernetes

Kubernetes supports dynamic volume provisioning—automatically creating a PersistentVolume (PV) when a PersistentVolumeClaim (PVC) is created—but:

- Default provisioners (like AWS EBS, GCE PD) are **block storage** and do **not** support `ReadWriteMany`.
- NFS supports `ReadWriteMany`, but requires **manual PV setup**, which includes:
  - Manually creating each PV.
  - Ensuring each PV has a unique path on the NFS server.
  - Manually cleaning up after deletion.

This is **labor-intensive** and **error-prone**, especially in dynamic or multi-tenant environments.

---

### 🚀 What the NFS Subdir External Provisioner Solves

The **NFS Subdir External Provisioner** allows **dynamic NFS storage provisioning**, automating the creation and cleanup of NFS-backed PersistentVolumes.

- ✅ Creates NFS-backed PVs on demand
- ✅ Automatically creates subdirectories for each PVC
- ✅ Supports `ReadWriteMany` access mode

---

### 🧰 Prerequisites

- A running **Kubernetes cluster**
- An **existing NFS server** that is accessible from all Kubernetes worker nodes

---

### 📦 Installation Steps

#### 1️⃣ Install NFS client on worker nodes

For Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y nfs-common
```

#### 2️⃣ Prepare NFS server directory

Create a directory that will be used by the provisioner:

```bash
sudo mkdir -p /mnt/data/k8s/subdir_external_provisioner
sudo chown -R nobody:nogroup /mnt/data/k8s/subdir_external_provisioner
```

#### 3️⃣ Install the Provisioner with Helm

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

helm install truenas-nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --namespace nfs-provisioner --create-namespace \
  --set nfs.server=<NFS_SERVER_IP> \
  --set nfs.path=/mnt/data/k8s/subdir_external_provisioner \
  --set storageClass.defaultClass=true \
  --set storageClass.name=truenas-nfs
```

#### ✅ Verify Installation

```bash
kubectl get pods -n nfs-provisioner
kubectl get storageclass
```

---

### 📄 Example: Create a PersistentVolumeClaim

`pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: truenas-nfs
  resources:
    requests:
      storage: 1Gi
```

Apply it:

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

You should see the PVC status as `Bound` shortly after.

---

### ✅ Summary

The NFS Subdir External Provisioner:

- Makes NFS usable as a dynamic storage backend
- Great for workloads needing `ReadWriteMany`
- Saves time and reduces error-prone manual configuration

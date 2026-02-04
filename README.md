# Kubernetes Volumes Practice (HostPath, NFS, AWS EFS)

This repository contains hands-on practice and screenshots for:

* HostPath Volume
* NFS Volume
* AWS EFS (Elastic File System) with Kubernetes



---

## 1️⃣ HostPath Volume

HostPath allows mounting a directory from the Node into a Pod.

### Example YAML

```yaml
volumes:
- name: host-volume
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

### Screenshots

![HostPath Step 1](images/screenshot1.png)
![HostPath Step 2](images/screenshot2.png)

---

## 2️⃣ NFS Volume

NFS allows multiple pods/nodes to share the same storage.

### NFS Server Setup

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
sudo mkdir -p /srv/nfs/kubedata
sudo chown nobody:nogroup /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
```

Edit exports:

```bash
sudo nano /etc/exports
/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)
```

Restart NFS:

```bash
sudo exportfs -rav
sudo systemctl restart nfs-kernel-server
```

### Pod Using NFS

```yaml
volumes:
- name: nfs-volume
  nfs:
    server: <NFS_SERVER_IP>
    path: /srv/nfs/kubedata
```

### Screenshots

![NFS Step 1](images/screenshot3.png)
![NFS Step 2](images/screenshot4.png)

---

## 3️⃣ AWS EFS with Kubernetes

AWS EFS is a managed NFS service that can be mounted across multiple EC2 instances and Kubernetes nodes.

### EFS Mount Command (Manual)

```bash
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
fs-xxxxxxxx.efs.ap-south-1.amazonaws.com:/ /home/ubuntu/efs
```

### Unmount Command (Correct)

```bash
sudo umount /home/ubuntu/efs
```

> ❌ Do NOT use mount options with umount

### EFS in Kubernetes (PVC + StorageClass)

#### StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
```

#### PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```

### Screenshots

![EFS Step 1](images/screenshot5.png)
![EFS Step 2](images/screenshot6.png)
![EFS Step 3](images/screenshot7.png)
![EFS Step 4](images/screenshot8.png)

---

## 🧠 Common Errors & Fixes

### Error: umount: invalid option -- 'o'

❌ Wrong:

```bash
sudo umount -t nfs4 -o nfsvers=4.1 ... /home/ubuntu/efs
```

✅ Correct:

```bash
sudo umount /home/ubuntu/efs
```

---

## ✅ Learning Outcomes

* Understood HostPath volumes
* Configured NFS server and client
* Integrated AWS EFS with Kubernetes
* Used PVC & StorageClass
* Debugged mount and umount issues

---

## 📌 Author

**Rajiv Nakhawa**
DevOps | AWS | Kubernetes | Linux

---

> ⚠️ Note: Rename screenshot files according to your repo and place them inside `images/` folder.

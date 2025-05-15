SIT737 9.1P – Comprehensive Guide**

Purpose
This guide walks through integrating MongoDB into your containerized Node.js microservice on Kubernetes, using ConfigMaps, Secrets, PersistentVolumes, StatefulSets, and Services to achieve a reliable, scalable database layer.
---
## Technology Stack

* **Node.js**: JavaScript backend service
* **MongoDB (v4.0.8)**: NoSQL database
* **Kubernetes**: Orchestration platform
* **Docker**: Container runtime
* **kubectl**: Kubernetes CLI
* **Minikube**: Local Kubernetes cluster
* **Git & GitHub**: Version control
---
## Files & Roles

| Filename                       | Purpose                                              |
| ------------------------------ | ---------------------------------------------------- |
| `createStatefulSet.yaml`       | Defines three MongoDB replica pods via a StatefulSet |
| `createHeadlessService.yaml`   | Headless Service for stable pod DNS                  |
| `createConfigMap.yaml`         | Replica set name and MongoDB config                  |
| `createMongoDBSecret.yaml`     | Encrypted Mongo credentials                          |
| `hostpath-storage.yaml`        | hostPath StorageClass for PVCs                       |
| `replicasetcommand.txt`        | `rs.initiate` commands for replica initialization    |
| `deployment.yaml`              | Node.js microservice Deployment                      |
| `app.js`, `package.json` files | Node.js application code and dependencies            |

---

## Deployment Steps & Commands

Execute these commands in order in **Command Prompt** (open a new window after updating PATH):

1. **Start Minikube**

   ```bat
   minikube start --driver=docker
   ```
2. **Create Headless Service**

   ```bat
   kubectl apply -f createHeadlessService.yaml
   ```
3. **Create MongoDB Secret**

   ```bat
   kubectl apply -f createMongoDBSecret.yaml
   ```
4. **Apply ConfigMap**

   ```bat
   kubectl apply -f createConfigMap.yaml
   ```
5. **Deploy StatefulSet**

   ```bat
   kubectl apply -f createStatefulSet.yaml
   ```
6. **Set up StorageClass**

   ```bat
   kubectl apply -f hostpath-storage.yaml
   ```
7. **Deploy Node.js App**

   ```bat
   kubectl apply -f deployment.yaml
   ```

---

## Verifying Pods

Check which pods are running:

```bat
kubectl get pods
```

You should see:

```
NAME                     READY   STATUS         RESTARTS   AGE
abc-0                    1/1     Running        0          X
abc-1                    1/1     Running        0          X
abc-2                    1/1     Running        0          X
myapp-<id>               1/1     Running        0          X
```

---

## Initializing the Replica Set

Once `abc-0` is **Running**, open the Mongo shell:

```bat
kubectl exec -it abc-0 -- mongo
```

Then run inside the shell:

```js
rs.initiate({
  _id: "rs0", members: [
    { _id: 0, host: "abc-0.abc.default.svc.cluster.local:27017" },
    { _id: 1, host: "abc-1.abc.default.svc.cluster.local:27017" },
    { _id: 2, host: "abc-2.abc.default.svc.cluster.local:27017" }
  ]
});
```

---

## App Connection String

Use this URI in your Node.js code (replace `<password>` with your Secret’s value):

```
mongodb://admin1:<password>@
  abc-0.abc.default.svc.cluster.local:27017,
  abc-1.abc.default.svc.cluster.local:27017,
  abc-2.abc.default.svc.cluster.local:27017
/?replicaSet=rs0
```

---

## Testing & Status

Check replica status and perform CRUD:

```bat
kubectl exec -it abc-0 -- mongo --eval "rs.status()"
```

```js
use testdb
db.users.insert({ name: "George" })
db.users.find()
```

---

## Cleanup Commands

```bat
kubectl delete -f deployment.yaml
kubectl delete -f createStatefulSet.yaml
kubectl delete -f createHeadlessService.yaml
kubectl delete -f createMongoDBSecret.yaml
kubectl delete -f createConfigMap.yaml
kubectl delete pvc -l app=mongo
minikube delete
```

---

## Git Setup & Submission

Initialize, commit, and push:

```bat
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Mita22-1/sit323-737-2024-t1-prac7p.git
git push -u origin main
```

---

**Student:** Mitali Kaur
**Course:** SIT737 – Cloud-Native Application Development
**Repo:** [https://github.com/Mita22-1/sit323-737-2024-t1-prac7p](https://github.com/Mita22-1/sit323-737-2024-t1-prac7p)

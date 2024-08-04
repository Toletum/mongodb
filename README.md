# Install mongodb in k8s with replicaset

## Create replicaset key file
```bash
openssl rand -base64 756   > mongo-keyfile
chmod 400 mongo-keyfile
```

```bash
kubectl create secret generic mongo-key --from-file=mongo-keyfile
```

## User/Pass admin
```bash
kubectl apply -f mongodb-secret.yaml
```

## Start pods
```bash
kubectl apply -f mongodb-statefulset.yaml
```

## Active replicaset
```bash
kubectl exec -it mongodb-0 -- mongosh -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD
```

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongodb-0.mongodb-headless:27017" },
    { _id: 1, host: "mongodb-1.mongodb-headless:27017" },
    { _id: 2, host: "mongodb-2.mongodb-headless:27017" }
  ]
})

use admin
db.createUser({
  user: "dbadmin",
  pwd: "dbpassword",
  roles: [ { role: "root", db: "admin" } ]
})
```

```bash
kubectl exec -it mongodb-0 -- mongosh "mongodb://dbadmin:dbpassword@localhost:27017/admin?replicaSet=rs0
```

## ReplicaSet status
```javascript
rs.status()
```

## Port-forward
```bash
kubectl port-forward svc/mongodb-headless 27017:27017
```


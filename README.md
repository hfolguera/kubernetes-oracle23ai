# kubernetes-oracle23ai

This repository helps you to deploy an Oracle Database 23ai free version in your Kubernetes cluster.

## Introduction
Oracle Database 23ai Free offers the ability to experience Oracle Database, which businesses throughout the world rely on for their mission-critical workloads. The resource limits for Oracle Database Free are up to 2 CPUs for foreground processes, 2 GB of RAM and 12 GB of user data on disk. It is packaged for ease of use and simple download.

You can find more information at: https://www.oracle.com/database/free/

## Get Started
You can find a ready to use Docker image for Oracle 23ai free in official [Oracle Container Registry](https://container-registry.oracle.com/)

Simply download your required image with one of the following commands:
```
docker pull container-registry.oracle.com/database/free:latest
docker pull container-registry.oracle.com/database/free:23.4.0.0
docker pull container-registry.oracle.com/database/free:23.4.0.0-lite
```

Check the latest available versions at: [https://container-registry.oracle.com/ords/ocr/ba/database](https://container-registry.oracle.com/ords/ocr/ba/database)

### Quick Docker deployment
Long-story short... deploy an Oracle 23ai container in Docker with...

```
podman run --name <container name> \
-P | -p <host port>:1521 \
-e ORACLE_PWD=<your database passwords> \
-e ORACLE_CHARACTERSET=<your character set> \
-e ENABLE_ARCHIVELOG=true \
-e ENABLE_FORCE_LOGGING=true \
-v [<host mount point>:]/opt/oracle/oradata \
container-registry.oracle.com/database/free:latest
```

## Kubernetes deployment
Now you are familiar with the Docker image, deploy it in a Kubernetes cluster to benefit from all its features. Execute the following steps to successfully deploy it.

### Namespace creation
All Oracle23ai resources will be organized under the `oracle` namespace:
```
kubectl apply -f oracle23ai-namespace.yaml
```

### Persistent Volumes
In order to store your database files, create a Persistent Volume Claim. In my case, I use an nfs-client storageclass that automatically creates a volume from my homelab NAS.
If you are interested in the NFS configuration check my [deploy-kubernetes-cluster](https://github.com/hfolguera/deploy-kubernetes-cluster?tab=readme-ov-file#configure-storage-provider) post installation steps.

Create the Persistent Volume Claim (PVC) with the following command:
```
kubectl apply -f oracle23ai-pvc.yaml
```

### Secrets
To securely store your Database passwords, manually create a secret with the following command:
```
kubectl create secret generic oracle-pass --from-literal=password=welcome1 -n oracle
```

This password will be used for your `SYS, SYSTEM and PDBADMIN` users.

### Deployment
Next, create a kubernetes deployment to create the database pod:
```
kubectl apply -f oracle23ai-deployment.yaml
```

Feel free to edit the file and modify for any of your requirements.
Note that deployment is using the `latest` tag. This is for test purposes since this tag will always exist. Consider using a fixed image tag for your environment as a kubernetes best practice.

### Service
Finally, create a service in order to be able to connect to the Database from external clients:
```
kubectl apply -f oracle23ai-service.yaml
```

Again, feel free to adapt the service resource based on your needs. In my case, I'm using a LoadBalancer type based on `metallb` configuration.

### Validation
You can validate you have successfully deployed your Oracle23ai environment by checking the pod's logs:
```
kubectl get all -n oracle
```
Check the overall status of the resources.

Get the pod's name and execute the following command:
```
kubectl logs pod/oracle23ai-6ff9998fc4-xxxxx -n oracle
```

If everything has gone as expected, you should be able to see the following in your log:
```
#########################
DATABASE IS READY TO USE!
#########################
```

Now, you can proceed to connect to your Database using any of your favourite database clients (sqlplus, SQLDeveloper, VSCode extension, sqlcli, DBeaver...)

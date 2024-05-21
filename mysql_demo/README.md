# Setup

## Prerequisites

### Installed minikube which is currently active context for kubectl
### Installed helm `brew install helm`
### Installed [KEDA with helm](https://keda.sh/docs/2.14/deploy/#helm)

## Create the namespace for mysql and app

```
kubectl create ns mysql
kubectl create ns app
```

## Create MySQL

```
kubectl apply -f mysql-secret.yaml 
kubectl apply -f mysql-pod.yaml   
kubectl apply -f mysql-service.yaml
```

## Create the application to be monitored

This is the busyboxx app in the app namespace.

```
kubectl apply -f busybox.yaml
```

## Install KEDA 

```
kubectl apply -f busybox.yaml
```

## Connect the DB pod and create the Database

```
kubectl exec k8s-mysql -it --namespace mysql -- bash
```

```
mysql --user=root --password=$MYSQL_ROOT_PASSWORD

```

```
create database test;
use test;
CREATE TABLE `test` (`num` int(9));
INSERT into test values (5);
```

## Debug the KEDA operator

```
kubectl get pods -n keda
kubectl logs -n keda -f {keda-pod-name} -c keda-operator
```

# Explore results

## Example

Min replicas - 0
Max replicas - 5
activationQueryValue - 10
queryValue - 15

The table gives the scaling results, ordered by time.

| Value in DB | Number of replicas |
|-------------|--------------------|
| 5           | 0                  |
| 10          | 0                  |
| 11          | 1                  |
| 15          | 1                  |
| 16          | 1                  |
| 20          | 2                  |
| 31          | 2                  |
| 45          | 3                  |
| 60          | 4                  |

## Change the values in DB

```
kubectl exec k8s-mysql -it --namespace mysql -- bash
mysql --user=root --password=$MYSQL_ROOT_PASSWORD
create database test;
use test;
update test set num=<put-a-value-here>;
```

These values will reflect the HPA and will deactivate the deployment eventually.

## Monitor the automatically created HPA

```
kubectl get hpa keda-hpa-mysql-scaledobject -n app --watch
```

Example:

```
NAME                          REFERENCE            TARGETS           MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-mysql-scaledobject   Deployment/busybox   12250m/15 (avg)   1         5         4          4h16m
```

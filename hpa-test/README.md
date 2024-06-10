An experiment how the HPA will behave when we have a missing metrics.

```
kubectl apply -f busybox.yaml
deployment.apps/busybox created


kubect get pods --namespace=hackdays

NAME                       READY   STATUS    RESTARTS   AGE
busybox-77b7dfbb8b-8s6cv   1/1     Running   0          32s
busybox-77b7dfbb8b-scblc   1/1     Running   0          32s
```

The targets are not calculated:

```
kubectl get hpa --namespace=hackdays

NAME          REFERENCE            TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
busybox-hpa   Deployment/busybox   <unknown>/25 (avg)   2         10        2          60m
```

More details:

```
➜  hpa-test git:(main) ✗ kubectl describe hpa --namespace=hackdays
Name:                                             busybox-hpa
Namespace:                                        hackdays
Labels:                                           <none>
Annotations:                                      <none>
CreationTimestamp:                                Mon, 10 Jun 2024 15:12:19 +0300
Reference:                                        Deployment/busybox
Metrics:                                          ( current / target )
  "not-existant-metrics" (target average value):  <unknown> / 25
Min replicas:                                     2
Max replicas:                                     10
Behavior:
  Scale Up:
    Stabilization Window: 60 seconds
    Select Policy: Max
    Policies:
      - Type: Pods     Value: 4    Period: 15 seconds
      - Type: Percent  Value: 100  Period: 15 seconds
  Scale Down:
    Stabilization Window: 300 seconds
    Select Policy: Max
    Policies:
      - Type: Percent  Value: 100  Period: 15 seconds
Deployment pods:       2 current / 2 desired
Conditions:
  Type            Status  Reason                   Message
  ----            ------  ------                   -------
  AbleToScale     True    ReadyForNewScale         recommended size matches current size
  ScalingActive   False   FailedGetExternalMetric  the HPA was unable to compute the replica count: unable to get external metric hackdays/not-existant-metrics/&LabelSelector{MatchLabels:map[string]string{...}: unable to fetch metrics from external metrics API: the server could not find the metric not-existant-metrics for 
  ScalingLimited  False   DesiredWithinRange       the desired count is within the acceptable range
Events:
  Type     Reason                   Age                    From                       Message
  ----     ------                   ----                   ----                       -------
  Warning  FailedGetExternalMetric  3m10s (x241 over 63m)  horizontal-pod-autoscaler  unable to get external metric hackdays/not-existant-metrics/&LabelSelector{MatchLabels:map[string]string{....}: unable to fetch metrics from external metrics API: the server could not find the metric not-existant-metrics for

```
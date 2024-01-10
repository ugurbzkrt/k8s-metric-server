wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.0/components.yaml


kubectl apply -f components.yaml


kubectl get pods -n kube-system | grep metrics-server

kubectl top pod <pod-name> --namespace=<namespace>
kubectl describe hpa <hpa-name> --namespace=<namespace>

-----------

## Install Metric Server 

- First Delete the existing Metric Server if any.

```bash
$ kubectl delete -n kube-system deployments.apps metrics-server
```

- Get the Metric Server form [GitHub](https://github.com/kubernetes-sigs/metrics-server/releases/tag/v0.5.0).

```bash
$ wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
```

- Edit the file `components.yaml`. You will select the `Deployment` part in the file. Add the below line to `containers.args field under the deployment object`.

```yaml
        - --kubelet-insecure-tls
``` 
(We have already done for this lesson)

```yaml
apiVersion: apps/v1
kind: Deployment
......
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
......	
```

- Add `metrics-server` to your Kubernetes instance.

```bash
$ kubectl apply -f components.yaml
```
- Wait 1-2 minute or so.

- Verify the existace of `metrics-server` run by below command

```bash
$ kubectl -n kube-system get pods
```

- Verify `metrics-server` can access resources of the pods and nodes.

```bash
$ kubectl top pods
```
- Look at the the values under TARGETS. The values are changed from `<unknown>/50%` to `1%/50%` & `2%/50%`, means the HPA can now idendify the current use of CPU.

- If it is still `<unknown>/50%`, check the `spec.template.spec.containers.resources.request` field of deployment.yaml files. It is required to define this field. Otherwise, the autoscaler will not take any action for that metric. 

> For per-pod resource metrics (like CPU), the controller fetches the metrics from the resource metrics API for each Pod targeted by the HorizontalPodAutoscaler. Then, if a target utilization value is set, the controller calculates the utilization value as a percentage of the equivalent resource request on the containers in each Pod.
 
> Please note that if some of the Pod's containers do not have the relevant resource request set, CPU utilization for the Pod will not be defined and the autoscaler will not take any action for that metric.


--------------


Logs:

kubectl logs -f oobeya-addons-547c4f9f47-2tx8r

 - Pod'un live log'u.

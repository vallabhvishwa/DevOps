# kubectl Cheatsheet

> See [Kubernetes](../Kubernetes/DevOps-04-Kubernetes.md) | [Troubleshooting](../../Troubleshooting/Kubernetes/02-Kubernetes-Troubleshooting.md)

## Context & info

```bash
kubectl config current-context
kubectl config use-context my-aks
kubectl cluster-info
kubectl get nodes -o wide
kubectl api-resources
```

## Workloads

```bash
kubectl get pods -A
kubectl get deploy,svc,ingress -n myns
kubectl describe pod POD -n NS
kubectl logs POD -n NS -f
kubectl logs POD -n NS --previous
kubectl exec -it POD -n NS -- sh
```

## Apply & delete

```bash
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl delete pod POD --grace-period=0 --force   # emergency
```

## Rollouts

```bash
kubectl rollout status deploy/APP -n NS
kubectl rollout history deploy/APP -n NS
kubectl rollout undo deploy/APP -n NS
kubectl set image deploy/APP container=img:tag -n NS
```

## Debug

```bash
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
kubectl get events -n NS --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods -n NS
kubectl auth can-i create pods -n NS
```

## Port forward

```bash
kubectl port-forward svc/myapp 8080:80 -n NS
```

## Common pod states

| State | First check |
|-------|-------------|
| Pending | `kubectl describe pod` events |
| ImagePullBackOff | image name, pull secrets |
| CrashLoopBackOff | `kubectl logs --previous` |
| Not Ready | probe failures in describe |

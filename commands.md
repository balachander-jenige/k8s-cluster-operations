Kubernetes common kubectl commands — simple and easy to revise

- Create or update objects: `kubectl apply -f <file>` — create or patch from YAML
- Create object (one-time): `kubectl create -f <file>` — create from a definition
- Replace object: `kubectl replace -f <file>` — replace the full object
- Delete object: `kubectl delete <resource> <name>` or `kubectl delete -f <file>`
- List resources: `kubectl get <resource>` (e.g., `pods`, `deployments`, `replicasets`, `services`)
- List all namespaces: `kubectl get pods --all-namespaces`
- List by namespace: `kubectl get pods --namespace=<namespace>`
- Create namespace: `kubectl create namespace <name>`
- Scale from file or resource: `kubectl scale --replicas=<n> -f <file>` or `kubectl scale deployment/<name> --replicas=<n>`
- Describe resource details: `kubectl describe <resource> <name>`
- View logs: `kubectl logs <pod>` (add `-c <container>` if needed)
- Exec into a pod: `kubectl exec -it <pod> -- /bin/sh` (or `/bin/bash`)
- Port-forward: `kubectl port-forward <pod|svc> <local>:<remote>`
- Rollout status: `kubectl rollout status deployment/<name>`
- Restart rollout: `kubectl rollout restart deployment/<name>`
- View events: `kubectl get events`

Quick tips
- Use `kubectl apply -f` when you edit YAML often (idempotent).
- Use `kubectl get <resource> -o wide` for more info.
- Add `--namespace=<ns>` to commands to target a namespace.

Examples
- `kubectl apply -f deployment.yaml`
- `kubectl get pods --namespace=dev`
- `kubectl scale deployment/my-app --replicas=3`
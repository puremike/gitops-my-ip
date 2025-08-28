# Kubernetes / EKS Setup & Management

## Sealed Secrets (Bitnami + kubeseal)

1. Add bitnami repo: ```helm repo add bitnami https://charts.bitnami.com/bitnami```
   
2. Update Helm repo: ```helm repo update```
   
3. Install SealedSecret: ```helm install sealed-secrets-controller bitnami/sealed-secrets \
  --namespace kube-system \
  --set service.type=ClusterIP \
  --create-namespace```

4. Seal a secret: ```kubeseal --controller-name=sealed-secrets-controller \
  --controller-namespace=kube-system \
  -n namespace-name < path/rendered-secret.yaml > path/sealed-secret.yaml```

## ArgoCD Installation & Management

1. Create a argocd namespace: ```kubectl create namespace argocd```
   
2. Install ArgoCD: ```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```
   
3. Expose ArgoCD Server (LoadBalancer): ```kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'```
   
4. Get Services: ```kubectl get svc -n argocd```
   
5. GEt argocd default password: ```kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d; echo```
   
6. Login to argocd: ```argocd login localhost:8080 \
  --username admin \
  --password "$(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)" \
  --insecure```

7. Get ArgoCD applications: ```kubectl get applications -n argocd```
8. Sync ArgoCD app: ```argocd app sync <app-name>```


## Storage Management (EBS / PVC)

1. Set Default StorageClass: ```kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'```

2. Install EBS CSI Driver with eksctl: ```eksctl create addon \
    --name aws-ebs-csi-driver \
    --cluster your-cluster-name \
    --service-account-role-arn arn:aws:iam::<aws_account_id>:role/AmazonEKS_EBS_CSI_DriverRole \
    --force```

3. Install EBS CSI Driver with AWS: ```aws eks create-addon \
    --cluster-name your-cluster-name \
    --addon-name aws-ebs-csi-driver \
    --service-account-role-arn arn:aws:iam::<aws_account_id>:role/AmazonEKS_EBS_CSI_DriverRole \
    --resolve-conflicts OVERWRITE \
    --region your-region-code```

4. Pod Identity & Role Association - Install the addon: ```eksctl create addon --cluster=<cluster-name> --name=eks-pod-identity-agent```

5. Pod Identity & Role Association - Associate: ```eksctl create podidentityassociation \
  --cluster <cluster-name> \
  --namespace kube-system \
  --service-account-name ebs-csi-controller-sa \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/AmazonEKS_EBS_CSI_DriverRole```

6. Get Pod Identity: ```eksctl get podidentityassociation --cluster <cluster-name>```

7. Verify: ```kubectl get pods -n kube-system | grep ebs-csi``` | ```kubectl logs -n kube-system -l app=ebs-csi-controller -c ebs-plugin```


## Node Management
1. Scale Nodegroup: ```eksctl scale nodegroup --cluster=<clusterName> --name=<nodegroupName> --nodes=<desiredCount>```

2. Create Nodegroup: ```eksctl create nodegroup \
  --cluster my-cluster \
  --name my-second-group \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 15 \
  --node-type m5.large```

3. Scale with AWS: ```aws eks update-nodegroup-config \
  --cluster-name <cluster-name> \
  --nodegroup-name <nodegroup-name> \
  --scaling-config minSize=1,maxSize=2,desiredSize=1```

4. Get Nodegroup info: ```eksctl get nodegroup --cluster <cluster-name> --region us-east-1 --name <nodegroup-name>```


## Taints & Node Affinity
1. Taint Node: ```kubectl taint nodes <node-name> dedicated=postgres:NoSchedule```

2. Remove Taint: ```kubectl taint nodes <node-name> dedicated=postgres:NoSchedule-```

3. List Tainted Nodes: ```kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints --no-headers | grep -v "<none>"```

4. Pod toleration example: ```
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - postgres```

## Restart / Delete

1. Restart deploy: ```kubectl rollout restart deploy <service_name> -n <namespace_name>```

## Monitoring: kube-prometheus-stack

1. Install Helm Chart: ```helm repo add prometheus-community https://prometheus-community.github.io/helm-charts```
2. Update Helm repo: ```helm repo update```
3. Create namespace: ```kubectl create namespace monitoring```
4. Install prom: ```helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring```
5. Get Grafana password: ```kubectl get secret kube-prometheus-stack-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 --decode```
6. Apply: ```kubectl apply -f <path-to-rule-manifest>```
7. Get Prom Details: ```kubectl get prometheus -n monitoring -o yaml```


## Popular Prometheus Queries

### Core System Metrics
1. http_requests_total
2. http_requests_total{status="500"}
3. rate(http_requests_total[5m])
4. (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) * 100
5. (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) /
6. node_memory_MemTotal_bytes * 100
7. node_filesystem_avail_bytes / node_filesystem_size_bytes * 100

### Kubernetes Metrics
1. increase(kube_pod_container_status_restarts_total[5m])
2. kube_deployment_status_replicas_available{deployment="your-deployment"}
3. kube_node_status_condition{condition="Ready", status="true"}
4. rate(container_cpu_usage_seconds_total[5m])
5. container_memory_usage_bytes

### Alerting Essentials
1. sum(rate(container_cpu_usage_seconds_total[5m])) > 0.8
2. kube_deployment_status_replicas_available{deployment="my-app"} == 0
3. increase(http_requests_total{status=~"5.."}[5m]) > 100
4. histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

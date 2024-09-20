1 - Clonar el repositorio de git
2 - Modificar el archivo deploy/charts/crate-operator/values.yaml ->
CRATEDB_OPERATOR_DEBUG_VOLUME_SIZE: "5GiB"
CRATEDB_OPERATOR_DEBUG_VOLUME_STORAGE_CLASS: "db-storage-retain"
Modificar Recursos
resources:
  limits:
    cpu: 500m
    memory: 256Mi
  requests:
    cpu: 250m
    memory: 256Mi

Agregar Afinidad
affinity: 
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-role.kubernetes.io/storage-crate
              operator: In
              values:
                - storage-crate

helm dependency build ./charts/crate-operator
helm install crate-operator ./charts/crate-operator -n st-sql-crate --create-namespace
kubectl get namespaces
kubectl get pod -n st-sql-crate
kubectl get pvc -n st-sql-crate
kubectl describe pod crate-operator-854b46bc-gnwlw -n st-sql-crate
kubectl describe node ip-10-138-0-66.ec2.internal


Para desinstalar
helm uninstall crate-operator -n st-sql-crate
kubectl delete namespace st-sql-crate
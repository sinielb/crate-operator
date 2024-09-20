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

3 - Recrear dependencias (Interno)
helm dependency build ./charts/crate-operator

4 - Desplegar el Operador
helm install crate-operator ./charts/crate-operator -n st-sql-crate --create-namespace

5 - Confirmar 
kubectl get namespaces
kubectl get pod -n st-sql-crate
kubectl get pvc -n st-sql-crate
kubectl describe pod crate-operator-64bfcb84-pw7w4 -n st-sql-crate
kubectl describe node ip-10-138-0-66.ec2.internal

Para desinstalar
helm uninstall crate-operator -n st-sql-crate
kubectl delete namespace st-sql-crate

6 - Crear Cluster de Crate
Modificar el manifiesto cluster-crate.yaml
  name: crate-cluster
  namespace: st-distributed
spec:
  cluster:
    imageRegistry: crate
    name: crate-dev
    version: 5.7.3
  nodes:
    data:
    - name: crate-cluster
      replicas: 3
      resources:
        limits:
          cpu: 2
          memory: 4Gi
        disk:
          count: 1
          size: 200GiB
          storageClass: gp2
        heapRatio: 0.25
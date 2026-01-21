# OpenShift Web Console – Administrator View Resources

> Interview-Ready Documentation | Senior OpenShift Administrator

---

## Table of Contents

### Overview
- [Overview](#overview-1)
- [Dashboards](#dashboards)

### Compute
- [Nodes](#nodes)
- [Machines](#machines)
- [MachineSets](#machinesets)
- [MachineConfigs](#machineconfigs)

### Workloads
- [Pods](#pods)
- [Deployments](#deployments)
- [DeploymentConfigs](#deploymentconfigs)
- [ReplicaSets](#replicasets)
- [ReplicationControllers](#replicationcontrollers)
- [StatefulSets](#statefulsets)
- [DaemonSets](#daemonsets)
- [Jobs](#jobs)
- [CronJobs](#cronjobs)
- [PodDisruptionBudgets](#poddisruptionbudgets)

### Networking
- [Services](#services)
- [Routes](#routes)
- [Ingresses](#ingresses)
- [NetworkPolicies](#networkpolicies)
- [Endpoints](#endpoints)

### Storage
- [PersistentVolumes](#persistentvolumes)
- [PersistentVolumeClaims](#persistentvolumeclaims)
- [StorageClasses](#storageclasses)
- [VolumeSnapshots](#volumesnapshots)

### Builds
- [BuildConfigs](#buildconfigs)
- [Builds](#builds)

### Images
- [ImageStreams](#imagestreams)
- [ImageStreamTags](#imagestreamtags)
- [ImageStreamImages](#imagestreamimages)

### User Management
- [Users](#users)
- [Groups](#groups)
- [Roles](#roles)
- [RoleBindings](#rolebindings)
- [ClusterRoles](#clusterroles)
- [ClusterRoleBindings](#clusterrolebindings)
- [ServiceAccounts](#serviceaccounts)

### Administration
- [Projects](#projects)
- [Namespaces](#namespaces)
- [LimitRanges](#limitranges)
- [ResourceQuotas](#resourcequotas)
- [Events](#events)

### Operators
- [Installed Operators](#installed-operators)
- [OperatorHub](#operatorhub)
- [Subscriptions](#subscriptions)
- [CatalogSources](#catalogsources)
- [ClusterServiceVersions](#clusterserviceversions)

### Custom Resources
- [CustomResourceDefinitions](#customresourcedefinitions)

### Cluster Configuration
- [Authentication](#authentication)
- [OAuth](#oauth)
- [FeatureGates](#featuregates)
- [Schedulers](#schedulers)
- [Proxies](#proxies)

### Quick Reference
- [Quick Reference Commands](#quick-reference-commands)

---

# Overview

## Overview

### Definition
- Central dashboard showing cluster health, resource utilization, and alerts at a glance
- Entry point for administrators to assess cluster state quickly

### Why It Is Used
- Provides immediate visibility into cluster health, capacity, and critical alerts for rapid triage.

### Use Cases
- Quick health check before deployments
- Identifying resource pressure across nodes

### Important CLI Commands
```bash
oc get clusterversion
oc describe clusterversion version
oc get clusteroperators
oc adm top nodes
```

### Sample YAML
```yaml
# No direct YAML — accessed via Web Console or CLI status commands
```

---

## Dashboards

### Definition
- Pre-built and customizable views for monitoring cluster metrics, workloads, and infrastructure
- Backed by Prometheus and Grafana for visualization

### Why It Is Used
- Enables proactive monitoring and capacity planning through visual metric representation.

### Use Cases
- Monitoring CPU/memory trends over time
- Tracking pod restart rates and error patterns

### Important CLI Commands
```bash
oc get pods -n openshift-monitoring
oc logs -n openshift-monitoring prometheus-k8s-0
oc get servicemonitors -A
```

### Sample YAML
```yaml
# Dashboards are managed via ConfigMaps in openshift-monitoring namespace
apiVersion: v1
kind: ConfigMap
metadata:
  name: custom-dashboard
  namespace: openshift-config-managed
data:
  dashboard.json: |
    { "title": "Custom Metrics" }
```

---

# Compute

## Nodes

### Definition
- Worker or control-plane machines that run containerized workloads
- Fundamental compute units managed by OpenShift scheduler

### Why It Is Used
- Nodes provide the actual compute, memory, and storage capacity for running pods.

### Use Cases
- Scaling cluster capacity by adding nodes
- Cordoning nodes for maintenance

### Important CLI Commands
```bash
oc get nodes
oc describe node <node-name>
oc adm cordon <node-name>
oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data
oc adm uncordon <node-name>
oc label node <node-name> node-role.kubernetes.io/infra=
```

### Sample YAML
```yaml
# Node labels applied via oc label or MachineSet
apiVersion: v1
kind: Node
metadata:
  name: worker-0
  labels:
    node-role.kubernetes.io/worker: ""
    topology.kubernetes.io/zone: us-east-1a
```

---

## Machines

### Definition
- Represents a single machine instance provisioned by the Machine API
- Maps 1:1 with underlying infrastructure VMs or bare-metal servers

### Why It Is Used
- Provides infrastructure-level abstraction for automated node provisioning and lifecycle management.

### Use Cases
- Investigating node provisioning failures
- Correlating node issues with underlying machine state

### Important CLI Commands
```bash
oc get machines -n openshift-machine-api
oc describe machine <machine-name> -n openshift-machine-api
oc delete machine <machine-name> -n openshift-machine-api
oc get machines -n openshift-machine-api -o wide
```

### Sample YAML
```yaml
apiVersion: machine.openshift.io/v1beta1
kind: Machine
metadata:
  name: worker-us-east-1a-abc123
  namespace: openshift-machine-api
spec:
  providerSpec:
    value:
      instanceType: m5.xlarge
      ami:
        id: ami-0123456789abcdef0
```

---

## MachineSets

### Definition
- Controller that maintains a desired count of Machines
- Similar to ReplicaSet but for infrastructure nodes

### Why It Is Used
- Enables horizontal scaling of worker nodes with consistent configuration across availability zones.

### Use Cases
- Scaling worker pool from 3 to 6 nodes
- Creating dedicated node pools for specific workloads (infra, GPU)

### Important CLI Commands
```bash
oc get machinesets -n openshift-machine-api
oc describe machineset <machineset-name> -n openshift-machine-api
oc scale machineset <machineset-name> --replicas=5 -n openshift-machine-api
oc edit machineset <machineset-name> -n openshift-machine-api
```

### Sample YAML
```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  name: worker-us-east-1a
  namespace: openshift-machine-api
spec:
  replicas: 3
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-machineset: worker-us-east-1a
  template:
    metadata:
      labels:
        machine.openshift.io/cluster-api-machineset: worker-us-east-1a
    spec:
      providerSpec:
        value:
          instanceType: m5.xlarge
```

---

## MachineConfigs

### Definition
- Defines OS-level configuration applied to nodes via Machine Config Operator (MCO)
- Controls systemd units, files, kernel arguments, and container runtime settings

### Why It Is Used
- Ensures consistent, declarative OS configuration across all nodes in a MachineConfigPool.

### Use Cases
- Adding custom CA certificates to nodes
- Configuring kernel parameters (sysctl)
- Setting up chrony for NTP

### Important CLI Commands
```bash
oc get machineconfigs
oc describe machineconfig <mc-name>
oc get machineconfigpools
oc describe machineconfigpool worker
oc get nodes -l node-role.kubernetes.io/worker -o wide
```

### Sample YAML
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-custom-sysctl
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
        - path: /etc/sysctl.d/99-custom.conf
          mode: 0644
          contents:
            source: data:,vm.max_map_count%3D262144
```

---

# Workloads

## Pods

### Definition
- Smallest deployable unit containing one or more containers
- Ephemeral by design; managed by higher-level controllers

### Why It Is Used
- Pods execute application containers with shared networking and storage context.

### Use Cases
- Debugging container crashes via logs
- Exec into pod for troubleshooting

### Important CLI Commands
```bash
oc get pods -n <namespace>
oc describe pod <pod-name> -n <namespace>
oc logs <pod-name> -c <container-name> -n <namespace>
oc logs <pod-name> --previous -n <namespace>
oc exec -it <pod-name> -n <namespace> -- /bin/sh
oc delete pod <pod-name> -n <namespace>
oc get pods -o wide -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
  namespace: myapp
spec:
  containers:
    - name: app
      image: registry.redhat.io/ubi8/ubi:latest
      command: ["sleep", "infinity"]
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "200m"
```

---

## Deployments

### Definition
- Kubernetes-native controller for declarative pod management with rolling updates
- Manages ReplicaSets to ensure desired pod count

### Why It Is Used
- Standard method for deploying stateless applications with zero-downtime updates.

### Use Cases
- Rolling out new application versions
- Scaling application replicas
- Rollback to previous revision

### Important CLI Commands
```bash
oc get deployments -n <namespace>
oc describe deployment <name> -n <namespace>
oc rollout status deployment/<name> -n <namespace>
oc rollout history deployment/<name> -n <namespace>
oc rollout undo deployment/<name> -n <namespace>
oc scale deployment <name> --replicas=5 -n <namespace>
oc set image deployment/<name> container=image:tag -n <namespace>
```

### Sample YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:v1.2.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

---

## DeploymentConfigs

### Definition
- OpenShift-specific deployment controller with integrated triggers and lifecycle hooks
- Predates Kubernetes Deployments; still used for ImageStream triggers

### Why It Is Used
- Enables automatic deployments triggered by ImageStream changes or ConfigMap updates.

### Use Cases
- Auto-deploy when new image is pushed to ImageStream
- Running pre/post deployment hooks

### Important CLI Commands
```bash
oc get dc -n <namespace>
oc describe dc <name> -n <namespace>
oc rollout latest dc/<name> -n <namespace>
oc rollout status dc/<name> -n <namespace>
oc rollout cancel dc/<name> -n <namespace>
oc rollout undo dc/<name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    app: myapp
  strategy:
    type: Rolling
  triggers:
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - myapp
        from:
          kind: ImageStreamTag
          name: myapp:latest
    - type: ConfigChange
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 8080
```

---

## ReplicaSets

### Definition
- Ensures a specified number of pod replicas are running at all times
- Managed automatically by Deployments; rarely created directly

### Why It Is Used
- Provides self-healing capability by replacing failed pods automatically.

### Use Cases
- Investigating stuck rollouts
- Understanding deployment revision history

### Important CLI Commands
```bash
oc get replicasets -n <namespace>
oc describe replicaset <name> -n <namespace>
oc get rs -n <namespace> -o wide
oc delete replicaset <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-abc123
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:v1.0.0
```

---

## ReplicationControllers

### Definition
- Legacy pod replication controller predating ReplicaSets
- Used internally by DeploymentConfigs

### Why It Is Used
- Maintains backward compatibility with DeploymentConfig-based workloads.

### Use Cases
- Troubleshooting DeploymentConfig rollout issues
- Migrating legacy workloads to Deployments

### Important CLI Commands
```bash
oc get rc -n <namespace>
oc describe rc <name> -n <namespace>
oc scale rc <name> --replicas=5 -n <namespace>
oc delete rc <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-1
  namespace: production
spec:
  replicas: 3
  selector:
    app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:v1.0.0
```

---

## StatefulSets

### Definition
- Controller for stateful applications requiring stable network identity and persistent storage
- Pods are created sequentially with predictable names (app-0, app-1, app-2)

### Why It Is Used
- Essential for databases, message queues, and clustered applications needing ordered deployment.

### Use Cases
- Running PostgreSQL/MySQL clusters
- Deploying Kafka or Elasticsearch

### Important CLI Commands
```bash
oc get statefulsets -n <namespace>
oc describe statefulset <name> -n <namespace>
oc scale statefulset <name> --replicas=5 -n <namespace>
oc rollout status statefulset/<name> -n <namespace>
oc delete pod <name>-0 -n <namespace>
```

### Sample YAML
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
  namespace: database
spec:
  serviceName: postgresql
  replicas: 3
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
        - name: postgresql
          image: registry.redhat.io/rhel8/postgresql-13:latest
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/pgsql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp3-csi
        resources:
          requests:
            storage: 10Gi
```

---

## DaemonSets

### Definition
- Ensures a pod runs on all (or selected) nodes in the cluster
- Automatically schedules pods on new nodes as they join

### Why It Is Used
- Ideal for node-level services like log collectors, monitoring agents, and network plugins.

### Use Cases
- Deploying Fluentd/Fluent Bit for log aggregation
- Running node monitoring agents (node-exporter)

### Important CLI Commands
```bash
oc get daemonsets -n <namespace>
oc describe daemonset <name> -n <namespace>
oc rollout status daemonset/<name> -n <namespace>
oc get pods -l app=<daemonset-label> -o wide -n <namespace>
```

### Sample YAML
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: openshift-logging
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      containers:
        - name: fluentd
          image: registry.redhat.io/openshift-logging/fluentd-rhel8:latest
          volumeMounts:
            - name: varlog
              mountPath: /var/log
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
```

---

## Jobs

### Definition
- Runs a pod to completion for batch processing tasks
- Ensures successful completion with configurable retry policy

### Why It Is Used
- Executes one-off or batch tasks that must run to completion.

### Use Cases
- Database migrations
- Batch data processing
- Backup operations

### Important CLI Commands
```bash
oc get jobs -n <namespace>
oc describe job <name> -n <namespace>
oc logs job/<name> -n <namespace>
oc delete job <name> -n <namespace>
oc create job <name> --image=<image> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  namespace: production
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 600
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: myregistry/db-migrate:v1.0.0
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
```

---

## CronJobs

### Definition
- Schedules Jobs to run periodically based on cron expressions
- Creates Job objects at scheduled times

### Why It Is Used
- Automates recurring tasks like backups, report generation, and cleanup operations.

### Use Cases
- Nightly database backups
- Periodic cache cleanup
- Scheduled report generation

### Important CLI Commands
```bash
oc get cronjobs -n <namespace>
oc describe cronjob <name> -n <namespace>
oc get jobs -l job-name=<cronjob-name> -n <namespace>
oc create job --from=cronjob/<name> <manual-job-name> -n <namespace>
oc delete cronjob <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
  namespace: production
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: myregistry/backup-tool:latest
              env:
                - name: BACKUP_TARGET
                  value: "s3://my-bucket/backups"
```

---

## PodDisruptionBudgets

### Definition
- Limits voluntary disruptions to ensure minimum pod availability during maintenance
- Protects application availability during node drains and upgrades

### Why It Is Used
- Prevents service outages during cluster maintenance by enforcing minimum available replicas.

### Use Cases
- Protecting critical services during node upgrades
- Ensuring HA during voluntary pod evictions

### Important CLI Commands
```bash
oc get pdb -n <namespace>
oc describe pdb <name> -n <namespace>
oc get pdb -n <namespace> -o wide
oc delete pdb <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
---
# Alternative: maxUnavailable
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb-percent
  namespace: production
spec:
  maxUnavailable: 25%
  selector:
    matchLabels:
      app: myapp
```

---

# Networking

## Services

### Definition
- Stable network endpoint that load balances traffic to a set of pods
- Provides service discovery via DNS within the cluster

### Why It Is Used
- Abstracts pod IPs and provides reliable internal/external access to applications.

### Use Cases
- Internal service-to-service communication
- Exposing applications via LoadBalancer or NodePort

### Important CLI Commands
```bash
oc get services -n <namespace>
oc describe service <name> -n <namespace>
oc expose deployment <name> --port=8080 -n <namespace>
oc get endpoints <service-name> -n <namespace>
oc delete service <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
---
# Headless Service for StatefulSets
apiVersion: v1
kind: Service
metadata:
  name: postgresql-headless
  namespace: database
spec:
  clusterIP: None
  selector:
    app: postgresql
  ports:
    - port: 5432
```

---

## Routes

### Definition
- OpenShift-native resource for exposing services externally via HTTP/HTTPS
- Managed by the OpenShift Router (HAProxy-based)

### Why It Is Used
- Provides external access with TLS termination, path-based routing, and traffic splitting.

### Use Cases
- Exposing web applications externally
- Blue-green deployments with traffic splitting
- TLS termination at edge

### Important CLI Commands
```bash
oc get routes -n <namespace>
oc describe route <name> -n <namespace>
oc expose service <service-name> -n <namespace>
oc create route edge <name> --service=<service> --cert=tls.crt --key=tls.key -n <namespace>
oc get route <name> -o jsonpath='{.spec.host}' -n <namespace>
```

### Sample YAML
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp
  namespace: production
spec:
  host: myapp.apps.cluster.example.com
  to:
    kind: Service
    name: myapp
    weight: 100
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
---
# A/B Traffic Split
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-ab
  namespace: production
spec:
  to:
    kind: Service
    name: myapp-v1
    weight: 90
  alternateBackends:
    - kind: Service
      name: myapp-v2
      weight: 10
```

---

## Ingresses

### Definition
- Kubernetes-native resource for HTTP/HTTPS routing to services
- Processed by OpenShift Ingress Controller to create Routes

### Why It Is Used
- Provides cross-platform compatibility for external access configuration.

### Use Cases
- Multi-path routing to different services
- Migrating workloads from other Kubernetes distributions

### Important CLI Commands
```bash
oc get ingress -n <namespace>
oc describe ingress <name> -n <namespace>
oc get ingress <name> -o yaml -n <namespace>
oc delete ingress <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  namespace: production
  annotations:
    route.openshift.io/termination: edge
spec:
  rules:
    - host: myapp.apps.cluster.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

---

## NetworkPolicies

### Definition
- Controls pod-to-pod and external traffic using label selectors
- Default-deny model when policies are applied to a namespace

### Why It Is Used
- Implements microsegmentation and zero-trust networking for security compliance.

### Use Cases
- Isolating namespaces from each other
- Restricting database access to specific application pods

### Important CLI Commands
```bash
oc get networkpolicies -n <namespace>
oc describe networkpolicy <name> -n <namespace>
oc get networkpolicy <name> -o yaml -n <namespace>
oc delete networkpolicy <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## Endpoints

### Definition
- Lists IP addresses and ports of pods backing a Service
- Automatically managed by endpoint controller based on Service selector

### Why It Is Used
- Enables debugging of service connectivity and understanding pod-to-service mapping.

### Use Cases
- Verifying pods are correctly registered with a Service
- Debugging service connectivity issues

### Important CLI Commands
```bash
oc get endpoints -n <namespace>
oc describe endpoints <service-name> -n <namespace>
oc get endpoints <service-name> -o yaml -n <namespace>
```

### Sample YAML
```yaml
# Manually created Endpoints for external service
apiVersion: v1
kind: Endpoints
metadata:
  name: external-database
  namespace: production
subsets:
  - addresses:
      - ip: 10.0.1.100
      - ip: 10.0.1.101
    ports:
      - port: 5432
        protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: external-database
  namespace: production
spec:
  ports:
    - port: 5432
```

---

# Storage

## PersistentVolumes

### Definition
- Cluster-wide storage resource provisioned by admin or dynamically via StorageClass
- Lifecycle independent of pods; can be retained or recycled

### Why It Is Used
- Provides durable storage that persists beyond pod lifecycle for stateful workloads.

### Use Cases
- Provisioning storage for databases
- Managing storage lifecycle (retain, delete)

### Important CLI Commands
```bash
oc get pv
oc describe pv <pv-name>
oc get pv -o wide
oc delete pv <pv-name>
oc patch pv <pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

### Sample YAML
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-data
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    server: nfs.example.com
    path: /exports/data
```

---

## PersistentVolumeClaims

### Definition
- Request for storage by a user/application specifying size and access mode
- Binds to available PV or triggers dynamic provisioning

### Why It Is Used
- Allows applications to request storage without knowing underlying infrastructure details.

### Use Cases
- Requesting storage for application data
- Expanding volume size (if StorageClass supports)

### Important CLI Commands
```bash
oc get pvc -n <namespace>
oc describe pvc <name> -n <namespace>
oc get pvc -o wide -n <namespace>
oc delete pvc <name> -n <namespace>
oc patch pvc <name> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}' -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-data
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3-csi
  resources:
    requests:
      storage: 10Gi
```

---

## StorageClasses

### Definition
- Defines storage provisioner and parameters for dynamic PV creation
- Enables self-service storage provisioning for developers

### Why It Is Used
- Automates storage provisioning and standardizes storage tiers across the cluster.

### Use Cases
- Defining storage tiers (fast SSD, standard HDD)
- Enabling volume expansion and snapshot support

### Important CLI Commands
```bash
oc get storageclasses
oc describe storageclass <name>
oc get sc -o wide
oc patch storageclass <name> -p '{"allowVolumeExpansion":true}'
oc annotate storageclass <name> storageclass.kubernetes.io/is-default-class=true
```

### Sample YAML
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-csi
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

---

## VolumeSnapshots

### Definition
- Point-in-time copy of a PersistentVolumeClaim for backup or cloning
- Requires CSI driver with snapshot support

### Why It Is Used
- Enables backup, disaster recovery, and rapid provisioning from existing data.

### Use Cases
- Creating database backups before upgrades
- Cloning production data for testing

### Important CLI Commands
```bash
oc get volumesnapshots -n <namespace>
oc describe volumesnapshot <name> -n <namespace>
oc get volumesnapshotclasses
oc delete volumesnapshot <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: myapp-data-snapshot
  namespace: production
spec:
  volumeSnapshotClassName: csi-aws-vsc
  source:
    persistentVolumeClaimName: myapp-data
---
# Restore from snapshot
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-data-restored
  namespace: production
spec:
  storageClassName: gp3-csi
  dataSource:
    name: myapp-data-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

---

# Builds

## BuildConfigs

### Definition
- Defines how to build container images from source code in OpenShift
- Supports Source-to-Image (S2I), Docker, and Pipeline strategies

### Why It Is Used
- Automates CI/CD by building images directly within OpenShift from Git repositories.

### Use Cases
- Building applications from Git source
- Triggering builds on code push via webhooks

### Important CLI Commands
```bash
oc get buildconfigs -n <namespace>
oc describe buildconfig <name> -n <namespace>
oc start-build <name> -n <namespace>
oc start-build <name> --from-dir=. -n <namespace>
oc logs -f bc/<name> -n <namespace>
oc cancel-build <build-name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: myapp
  namespace: development
spec:
  source:
    type: Git
    git:
      uri: https://github.com/myorg/myapp.git
      ref: main
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        namespace: openshift
        name: nodejs:18-ubi8
  output:
    to:
      kind: ImageStreamTag
      name: myapp:latest
  triggers:
    - type: GitHub
      github:
        secret: github-webhook-secret
    - type: ImageChange
    - type: ConfigChange
```

---

## Builds

### Definition
- Instance of a BuildConfig execution that produces a container image
- Tracks build status, logs, and output artifacts

### Why It Is Used
- Represents the actual build execution with logs and status for troubleshooting.

### Use Cases
- Monitoring build progress
- Debugging failed builds

### Important CLI Commands
```bash
oc get builds -n <namespace>
oc describe build <name> -n <namespace>
oc logs build/<name> -n <namespace>
oc get builds -n <namespace> --sort-by=.metadata.creationTimestamp
oc delete build <name> -n <namespace>
```

### Sample YAML
```yaml
# Builds are typically created automatically by BuildConfigs
# Manual build creation is rare
apiVersion: build.openshift.io/v1
kind: Build
metadata:
  name: myapp-1
  namespace: development
spec:
  source:
    type: Git
    git:
      uri: https://github.com/myorg/myapp.git
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: nodejs:18-ubi8
        namespace: openshift
  output:
    to:
      kind: ImageStreamTag
      name: myapp:latest
```

---

# Images

## ImageStreams

### Definition
- Virtual repository that tracks image references and triggers deployments on updates
- Abstracts external registries and enables image promotion workflows

### Why It Is Used
- Provides image lifecycle management and automatic deployment triggers on image changes.

### Use Cases
- Tracking images across environments (dev→staging→prod)
- Triggering deployments when new images are pushed

### Important CLI Commands
```bash
oc get imagestreams -n <namespace>
oc describe imagestream <name> -n <namespace>
oc import-image <name>:tag --from=registry/image:tag --confirm -n <namespace>
oc tag <source> <destination> -n <namespace>
oc get is <name> -o jsonpath='{.status.dockerImageRepository}' -n <namespace>
```

### Sample YAML
```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
  namespace: development
spec:
  lookupPolicy:
    local: true
  tags:
    - name: latest
      from:
        kind: DockerImage
        name: quay.io/myorg/myapp:latest
      importPolicy:
        scheduled: true
      referencePolicy:
        type: Local
```

---

## ImageStreamTags

### Definition
- Specific tagged reference within an ImageStream pointing to an image SHA
- Immutable pointer to a specific image version

### Why It Is Used
- Enables version control and promotion of specific image versions across environments.

### Use Cases
- Promoting specific builds to production
- Rolling back to known-good image versions

### Important CLI Commands
```bash
oc get imagestreamtags -n <namespace>
oc describe imagestreamtag <imagestream>:<tag> -n <namespace>
oc tag <source-is>:<tag> <dest-is>:<tag> -n <namespace>
oc delete imagestreamtag <imagestream>:<tag> -n <namespace>
```

### Sample YAML
```yaml
# ImageStreamTags are typically created via oc tag command
# Direct creation example:
apiVersion: image.openshift.io/v1
kind: ImageStreamTag
metadata:
  name: myapp:v1.2.0
  namespace: production
tag:
  from:
    kind: DockerImage
    name: quay.io/myorg/myapp@sha256:abc123...
  referencePolicy:
    type: Local
```

---

## ImageStreamImages

### Definition
- Immutable reference to a specific image by SHA digest within an ImageStream
- Guarantees exact image content regardless of tag changes

### Why It Is Used
- Provides immutable image references for audit trails and reproducible deployments.

### Use Cases
- Referencing exact image for compliance
- Debugging which exact image version was deployed

### Important CLI Commands
```bash
oc get imagestreamimages -n <namespace>
oc describe imagestreamimage <imagestream>@<sha> -n <namespace>
oc get is <imagestream> -o jsonpath='{.status.tags[*].items[0].image}' -n <namespace>
```

### Sample YAML
```yaml
# ImageStreamImages are system-managed; no direct creation
# Reference in DeploymentConfig:
spec:
  containers:
    - name: myapp
      image: image-registry.openshift-image-registry.svc:5000/production/myapp@sha256:abc123def456...
```

---

# User Management

## Users

### Definition
- Represents an authenticated identity in OpenShift
- Created automatically upon first login via configured identity provider

### Why It Is Used
- Tracks authenticated users for RBAC assignments and audit logging.

### Use Cases
- Managing user access permissions
- Auditing user activities

### Important CLI Commands
```bash
oc get users
oc describe user <username>
oc delete user <username>
oc get identity
oc adm policy add-cluster-role-to-user cluster-admin <username>
```

### Sample YAML
```yaml
# Users are typically created by identity providers
# Manual creation (rarely needed):
apiVersion: user.openshift.io/v1
kind: User
metadata:
  name: developer1
identities:
  - htpasswd:developer1
```

---

## Groups

### Definition
- Collection of users for simplified RBAC management
- Can be synced from external identity providers (LDAP, Active Directory)

### Why It Is Used
- Enables role assignment to multiple users simultaneously for efficient access management.

### Use Cases
- Granting project access to development teams
- Syncing groups from corporate LDAP

### Important CLI Commands
```bash
oc get groups
oc describe group <group-name>
oc adm groups new <group-name> <user1> <user2>
oc adm groups add-users <group-name> <username>
oc adm groups remove-users <group-name> <username>
oc adm policy add-role-to-group edit <group-name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: user.openshift.io/v1
kind: Group
metadata:
  name: developers
users:
  - developer1
  - developer2
  - developer3
```

---

## Roles

### Definition
- Namespace-scoped set of permissions (verbs on resources)
- Defines what actions can be performed within a specific namespace

### Why It Is Used
- Implements principle of least privilege within a namespace scope.

### Use Cases
- Creating custom application-specific roles
- Granting deployment-only access to CI/CD service accounts

### Important CLI Commands
```bash
oc get roles -n <namespace>
oc describe role <name> -n <namespace>
oc create role <name> --verb=get,list,watch --resource=pods -n <namespace>
oc delete role <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create"]
```

---

## RoleBindings

### Definition
- Grants Role permissions to users, groups, or service accounts within a namespace
- Links subjects (who) to roles (what permissions)

### Why It Is Used
- Assigns namespace-scoped permissions to specific identities.

### Use Cases
- Granting developers edit access to their namespace
- Binding CI/CD service account to deployment role

### Important CLI Commands
```bash
oc get rolebindings -n <namespace>
oc describe rolebinding <name> -n <namespace>
oc adm policy add-role-to-user <role> <user> -n <namespace>
oc adm policy add-role-to-group <role> <group> -n <namespace>
oc delete rolebinding <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-edit
  namespace: production
subjects:
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: cicd-deployer
    namespace: cicd
roleRef:
  kind: Role
  name: edit
  apiGroup: rbac.authorization.k8s.io
```

---

## ClusterRoles

### Definition
- Cluster-wide set of permissions applicable across all namespaces
- Includes permissions for cluster-scoped resources (nodes, PVs)

### Why It Is Used
- Defines permissions for cluster administration and cross-namespace access.

### Use Cases
- Creating cluster-admin level custom roles
- Defining roles for operators requiring cluster-wide access

### Important CLI Commands
```bash
oc get clusterroles
oc describe clusterrole <name>
oc create clusterrole <name> --verb=get,list --resource=nodes
oc get clusterrole <name> -o yaml
```

### Sample YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["nodes"]
    verbs: ["get", "list"]
```

---

## ClusterRoleBindings

### Definition
- Grants ClusterRole permissions cluster-wide to subjects
- Provides access to all namespaces and cluster-scoped resources

### Why It Is Used
- Assigns cluster-wide administrative permissions to users or service accounts.

### Use Cases
- Granting cluster-admin access
- Binding monitoring service account to view all namespaces

### Important CLI Commands
```bash
oc get clusterrolebindings
oc describe clusterrolebinding <name>
oc adm policy add-cluster-role-to-user <clusterrole> <user>
oc adm policy add-cluster-role-to-group <clusterrole> <group>
oc delete clusterrolebinding <name>
```

### Sample YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-monitoring
subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: openshift-monitoring
  - kind: Group
    name: cluster-admins
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-monitoring-view
  apiGroup: rbac.authorization.k8s.io
```

---

## ServiceAccounts

### Definition
- Identity for pods and automated processes within a namespace
- Automatically mounted token provides API authentication for workloads

### Why It Is Used
- Enables pods to authenticate with Kubernetes API and external services securely.

### Use Cases
- Running CI/CD pipelines with specific permissions
- Granting pods access to pull images from private registries

### Important CLI Commands
```bash
oc get serviceaccounts -n <namespace>
oc describe serviceaccount <name> -n <namespace>
oc create serviceaccount <name> -n <namespace>
oc adm policy add-scc-to-user anyuid -z <sa-name> -n <namespace>
oc adm policy add-role-to-user edit -z <sa-name> -n <namespace>
oc create token <sa-name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-deployer
  namespace: production
secrets:
  - name: cicd-deployer-dockercfg
imagePullSecrets:
  - name: private-registry-secret
---
# Link to image pull secret
apiVersion: v1
kind: Secret
metadata:
  name: private-registry-secret
  namespace: production
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-config>
```

---

# Administration

## Projects

### Definition
- OpenShift wrapper around Kubernetes namespace with additional metadata
- Provides multi-tenancy with network isolation and resource quotas

### Why It Is Used
- Enables team-based resource isolation with self-service provisioning capabilities.

### Use Cases
- Creating isolated environments for teams
- Applying default quotas and network policies

### Important CLI Commands
```bash
oc get projects
oc describe project <name>
oc new-project <name> --display-name="My Project" --description="Description"
oc project <name>
oc delete project <name>
oc adm new-project <name> --node-selector="env=production"
```

### Sample YAML
```yaml
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  name: myteam-production
  annotations:
    openshift.io/display-name: "My Team Production"
    openshift.io/description: "Production environment for My Team"
    openshift.io/requester: admin
    openshift.io/node-selector: "env=production"
```

---

## Namespaces

### Definition
- Kubernetes-native resource for logical cluster partitioning
- Provides scope for names and a target for resource quotas

### Why It Is Used
- Provides resource isolation and access control boundaries within the cluster.

### Use Cases
- Creating infrastructure namespaces not requiring Project features
- Managing operator namespaces

### Important CLI Commands
```bash
oc get namespaces
oc describe namespace <name>
oc create namespace <name>
oc label namespace <name> environment=production
oc delete namespace <name>
oc get all -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: infrastructure
  labels:
    environment: production
    team: platform
  annotations:
    scheduler.alpha.kubernetes.io/node-selector: "node-role.kubernetes.io/infra="
```

---

## LimitRanges

### Definition
- Defines default and maximum resource limits for containers in a namespace
- Enforces resource constraints at pod admission time

### Why It Is Used
- Prevents resource abuse and ensures fair resource allocation across workloads.

### Use Cases
- Setting default CPU/memory limits for all pods
- Preventing unbounded resource requests

### Important CLI Commands
```bash
oc get limitranges -n <namespace>
oc describe limitrange <name> -n <namespace>
oc create -f limitrange.yaml -n <namespace>
oc delete limitrange <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: production
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "4Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
    - type: PersistentVolumeClaim
      max:
        storage: "50Gi"
      min:
        storage: "1Gi"
```

---

## ResourceQuotas

### Definition
- Limits total resource consumption within a namespace
- Enforces hard limits on CPU, memory, storage, and object counts

### Why It Is Used
- Ensures fair cluster resource sharing and prevents namespace resource exhaustion.

### Use Cases
- Limiting total CPU/memory per namespace
- Controlling number of pods, services, or PVCs

### Important CLI Commands
```bash
oc get resourcequotas -n <namespace>
oc describe resourcequota <name> -n <namespace>
oc create quota <name> --hard=cpu=4,memory=8Gi,pods=20 -n <namespace>
oc delete resourcequota <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "50"
    persistentvolumeclaims: "10"
    requests.storage: "100Gi"
    services.loadbalancers: "2"
```

---

## Events

### Definition
- Cluster-generated records of state changes and errors
- Short-lived (default 1 hour retention) diagnostic information

### Why It Is Used
- Provides real-time visibility into resource state changes and error conditions.

### Use Cases
- Debugging pod scheduling failures
- Investigating image pull errors

### Important CLI Commands
```bash
oc get events -n <namespace>
oc get events --sort-by=.lastTimestamp -n <namespace>
oc get events --field-selector type=Warning -n <namespace>
oc get events --field-selector involvedObject.name=<pod-name> -n <namespace>
oc get events -A --sort-by=.lastTimestamp
```

### Sample YAML
```yaml
# Events are system-generated; no manual creation
# Viewing events via API:
# GET /api/v1/namespaces/{namespace}/events
```

---

# Operators

## Installed Operators

### Definition
- Operators deployed and running in the cluster via OLM
- Provides application lifecycle management automation

### Why It Is Used
- Automates complex application deployment, scaling, and management tasks.

### Use Cases
- Managing database operators (PostgreSQL, MongoDB)
- Deploying monitoring stack (Prometheus, Grafana)

### Important CLI Commands
```bash
oc get csv -n <namespace>
oc get csv -A
oc describe csv <operator-name> -n <namespace>
oc get installplans -n <namespace>
oc get pods -n <operator-namespace>
```

### Sample YAML
```yaml
# Operators are installed via Subscription
# CSV is created automatically by OLM
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: postgresql-operator.v5.4.0
  namespace: openshift-operators
spec:
  displayName: PostgreSQL Operator
  install:
    strategy: deployment
```

---

## OperatorHub

### Definition
- Marketplace for discovering and installing Operators from multiple sources
- Aggregates Red Hat, certified, community, and custom catalogs

### Why It Is Used
- Provides centralized discovery and installation of cluster capabilities.

### Use Cases
- Discovering available operators for specific use cases
- Installing certified third-party operators

### Important CLI Commands
```bash
oc get catalogsources -n openshift-marketplace
oc get packagemanifests
oc describe packagemanifest <operator-name>
oc get packagemanifests | grep -i <search-term>
```

### Sample YAML
```yaml
# OperatorHub is accessed via Console or packagemanifests API
# Filter example for available channels:
oc get packagemanifest elasticsearch-operator -o jsonpath='{.status.channels[*].name}'
```

---

## Subscriptions

### Definition
- Declares intent to install and keep an Operator updated
- Specifies source catalog, channel, and approval strategy

### Why It Is Used
- Enables declarative operator installation with controlled update policies.

### Use Cases
- Installing operators with automatic updates
- Pinning operator to specific version channel

### Important CLI Commands
```bash
oc get subscriptions -n <namespace>
oc describe subscription <name> -n <namespace>
oc get subscription <name> -o yaml -n <namespace>
oc delete subscription <name> -n <namespace>
```

### Sample YAML
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: elasticsearch-operator
  namespace: openshift-operators-redhat
spec:
  channel: stable-5.8
  name: elasticsearch-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: Manual
  config:
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
```

---

## CatalogSources

### Definition
- Defines a repository of operators available for installation
- Points to container image containing operator metadata

### Why It Is Used
- Enables custom operator catalogs for air-gapped environments or internal operators.

### Use Cases
- Adding custom operator catalogs
- Mirroring Red Hat catalogs for disconnected clusters

### Important CLI Commands
```bash
oc get catalogsources -n openshift-marketplace
oc describe catalogsource <name> -n openshift-marketplace
oc get pods -n openshift-marketplace -l olm.catalogSource=<name>
oc logs -n openshift-marketplace -l olm.catalogSource=<name>
```

### Sample YAML
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: custom-operators
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: registry.example.com/operators/custom-catalog:v1.0.0
  displayName: Custom Operators
  publisher: My Organization
  updateStrategy:
    registryPoll:
      interval: 30m
```

---

## ClusterServiceVersions

### Definition
- Defines a specific operator version with its deployment and RBAC requirements
- Contains install strategy, owned CRDs, and dependency information

### Why It Is Used
- Provides versioned operator metadata for OLM to manage operator lifecycle.

### Use Cases
- Checking operator version and status
- Troubleshooting operator installation failures

### Important CLI Commands
```bash
oc get csv -n <namespace>
oc describe csv <name> -n <namespace>
oc get csv -n <namespace> -o jsonpath='{.items[*].spec.version}'
oc delete csv <name> -n <namespace>
```

### Sample YAML
```yaml
# CSVs are typically created by OLM from Subscription
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: myoperator.v1.0.0
  namespace: openshift-operators
spec:
  displayName: My Operator
  version: 1.0.0
  install:
    strategy: deployment
    spec:
      deployments:
        - name: myoperator-controller
          spec:
            replicas: 1
            selector:
              matchLabels:
                app: myoperator
            template:
              spec:
                containers:
                  - name: controller
                    image: registry.example.com/myoperator:v1.0.0
```

---

# Custom Resources

## CustomResourceDefinitions

### Definition
- Extends Kubernetes API with custom resource types
- Defines schema, validation, and versioning for custom objects

### Why It Is Used
- Enables operators and controllers to manage domain-specific resources declaratively.

### Use Cases
- Creating custom application configurations
- Defining operator-managed resources

### Important CLI Commands
```bash
oc get crds
oc describe crd <name>
oc get crd <name> -o yaml
oc get <custom-resource-kind> -A
oc api-resources | grep <name>
```

### Sample YAML
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    listKind: DatabaseList
    plural: databases
    singular: database
    shortNames:
      - db
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum: ["postgresql", "mysql"]
                version:
                  type: string
                storage:
                  type: string
```

---

# Cluster Configuration

## Authentication

### Definition
- Cluster-wide authentication configuration specifying identity providers
- Managed via OAuth cluster resource

### Why It Is Used
- Configures how users authenticate to the cluster (LDAP, OIDC, HTPasswd, etc.).

### Use Cases
- Integrating with corporate LDAP/Active Directory
- Configuring OIDC for SSO

### Important CLI Commands
```bash
oc get oauth cluster -o yaml
oc describe oauth cluster
oc get identityproviders
oc get users
oc get identity
```

### Sample YAML
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
    - name: ldap
      type: LDAP
      mappingMethod: claim
      ldap:
        url: "ldap://ldap.example.com:389/ou=users,dc=example,dc=com?uid"
        insecure: false
        ca:
          name: ldap-ca-configmap
        bindDN: "cn=admin,dc=example,dc=com"
        bindPassword:
          name: ldap-bind-password
        attributes:
          id: ["dn"]
          email: ["mail"]
          name: ["cn"]
          preferredUsername: ["uid"]
```

---

## OAuth

### Definition
- OpenShift OAuth server configuration for token-based authentication
- Manages token lifetimes, grant methods, and identity providers

### Why It Is Used
- Controls OAuth token policies and integrates external identity providers.

### Use Cases
- Configuring token expiration policies
- Setting up multiple identity providers

### Important CLI Commands
```bash
oc get oauth cluster -o yaml
oc edit oauth cluster
oc get oauthclients
oc describe oauthclient openshift-browser-client
oc get pods -n openshift-authentication
```

### Sample YAML
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  tokenConfig:
    accessTokenMaxAgeSeconds: 86400
    accessTokenInactivityTimeout: 3600s
  identityProviders:
    - name: htpasswd
      type: HTPasswd
      mappingMethod: claim
      htpasswd:
        fileData:
          name: htpasswd-secret
```

---

## FeatureGates

### Definition
- Controls enablement of alpha/beta Kubernetes and OpenShift features
- Applied cluster-wide via FeatureGate resource

### Why It Is Used
- Enables early access to new features or disables deprecated functionality.

### Use Cases
- Enabling tech preview features
- Disabling deprecated APIs before upgrade

### Important CLI Commands
```bash
oc get featuregate cluster -o yaml
oc describe featuregate cluster
oc edit featuregate cluster
```

### Sample YAML
```yaml
apiVersion: config.openshift.io/v1
kind: FeatureGate
metadata:
  name: cluster
spec:
  featureSet: TechPreviewNoUpgrade
---
# Or custom feature gates:
apiVersion: config.openshift.io/v1
kind: FeatureGate
metadata:
  name: cluster
spec:
  featureSet: CustomNoUpgrade
  customNoUpgrade:
    enabled:
      - CSIDriverRegistry
      - ExpandCSIVolumes
```

---

## Schedulers

### Definition
- Cluster-wide scheduler configuration for pod placement policies
- Controls default scheduling profiles and predicates

### Why It Is Used
- Customizes pod scheduling behavior across the cluster.

### Use Cases
- Configuring scheduler profiles for different workload types
- Enabling topology-aware scheduling

### Important CLI Commands
```bash
oc get scheduler cluster -o yaml
oc describe scheduler cluster
oc edit scheduler cluster
oc get pods -n openshift-kube-scheduler
```

### Sample YAML
```yaml
apiVersion: config.openshift.io/v1
kind: Scheduler
metadata:
  name: cluster
spec:
  policy:
    name: scheduler-policy
  defaultNodeSelector: "node-role.kubernetes.io/worker="
  mastersSchedulable: false
  profile: HighNodeUtilization
```

---

## Proxies

### Definition
- Cluster-wide proxy configuration for egress traffic
- Configures HTTP/HTTPS proxy and no-proxy settings

### Why It Is Used
- Routes cluster egress traffic through corporate proxy for internet access.

### Use Cases
- Configuring cluster to use corporate proxy
- Excluding internal domains from proxy

### Important CLI Commands
```bash
oc get proxy cluster -o yaml
oc describe proxy cluster
oc edit proxy cluster
oc get pods -n openshift-cluster-node-tuning-operator
```

### Sample YAML
```yaml
apiVersion: config.openshift.io/v1
kind: Proxy
metadata:
  name: cluster
spec:
  httpProxy: http://proxy.example.com:3128
  httpsProxy: http://proxy.example.com:3128
  noProxy: .cluster.local,.svc,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.example.com
  trustedCA:
    name: proxy-ca-bundle
```

---

# Quick Reference Commands

## Cluster Health
```bash
oc get clusterversion
oc get clusteroperators
oc get nodes
oc adm top nodes
oc adm top pods -A
```

## Troubleshooting
```bash
oc describe pod <pod> -n <ns>
oc logs <pod> -n <ns> --previous
oc debug node/<node>
oc adm inspect clusteroperator/<co>
oc adm must-gather
```

## Common Admin Tasks
```bash
oc adm cordon <node>
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data
oc adm uncordon <node>
oc adm policy add-scc-to-user <scc> -z <sa> -n <ns>
oc adm prune images --keep-tag-revisions=3 --keep-younger-than=60m
```

---

*Document prepared for OpenShift Administrator interviews - Production-focused, CLI-ready*

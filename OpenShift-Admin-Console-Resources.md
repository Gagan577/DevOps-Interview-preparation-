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
- Single-pane view of cluster health, alerts, and resource usage
- First place to check if something is wrong

### Why It Is Used
- Quick triage before making any changes to the cluster

### Use Cases
- **Scenario:** Got paged at 2 AM for production issue. Opened Overview, saw 2 nodes NotReady and CPU alerts firing. Instantly knew where to focus
- **Scenario:** Before deploying a release, checked Overview to confirm cluster was healthy—avoided deploying into an already degraded cluster

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
- Grafana dashboards showing cluster metrics over time
- Powered by Prometheus for historical data and trends

### Why It Is Used
- Capacity planning and performance troubleshooting with visual data

### Use Cases
- **Scenario:** App was slow, checked dashboard, saw memory climbing for 3 days—found a memory leak before it caused outage
- **Scenario:** Needed to justify adding nodes—showed management dashboard proving 85% memory usage trending up

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
- Physical or virtual machines where your pods actually run
- Workers run apps, Masters run control plane

### Why It Is Used
- Pods run on nodes—unhealthy nodes mean unhealthy apps

### Use Cases
- **Scenario:** Pods stuck in Pending. Ran `oc adm top nodes`, found all nodes at 95% memory. Added more nodes to fix
- **Scenario:** Needed to patch a node. Cordoned it first, then drained, patched, uncordoned. Zero downtime

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
- Represents the actual VM or server in your cloud provider (AWS EC2, Azure VM)
- Links cloud infrastructure to Kubernetes nodes

### Why It Is Used
- Debug infrastructure-level issues when nodes won't come up

### Use Cases
- **Scenario:** Node showing NotReady. Checked Machine object, saw "InsufficientInstanceCapacity" from AWS. Cloud quota issue, not Kubernetes
- **Scenario:** New node stuck provisioning for 20 mins. Machine events showed cloud API timeout. Deleted Machine, MachineSet created new one

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
- Template that maintains desired number of Machines (like ReplicaSet for nodes)
- Scale replicas up/down, Machines are created/deleted automatically

### Why It Is Used
- Automatic node scaling and replacement when nodes fail

### Use Cases
- **Scenario:** Big sale coming. Scaled MachineSet from 5 to 10 replicas. After sale, scaled back to 5. Saved cloud costs
- **Scenario:** Worker node crashed. MachineSet automatically created replacement—no manual intervention needed

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
- OS-level configuration for nodes (kernel params, certificates, systemd units)
- Changes trigger rolling node reboots

### Why It Is Used
- Only supported way to customize RHCOS without breaking upgrades

### Use Cases
- **Scenario:** Elasticsearch pods crashing. Needed vm.max_map_count=262144. Created MachineConfig, nodes rebooted one by one, ES started working
- **Scenario:** Security required corporate CA on all nodes. Added via MachineConfig—all nodes got it automatically

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
- Smallest unit in Kubernetes—one or more containers running together
- Temporary by design; controllers recreate them when they die

### Why It Is Used
- This is where your application actually runs

### Use Cases
- **Scenario:** App returning 500 errors. Ran `oc logs pod-name --previous`, found NullPointerException in last crashed container
- **Scenario:** Needed to check if app could reach database. Ran `oc exec -it pod-name -- curl db-service:5432`—connection refused, DB was down

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
- Manages pod replicas with rolling updates and rollbacks
- Standard way to deploy stateless applications

### Why It Is Used
- Zero-downtime deployments with easy rollback if something goes wrong

### Use Cases
- **Scenario:** Deployed bad version, users complained. Ran `oc rollout undo deployment/myapp`—back to working version in 10 seconds
- **Scenario:** Black Friday traffic spike. Ran `oc scale deployment myapp --replicas=10`—scaled from 3 to 10 pods instantly

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
- OpenShift-specific deployment controller (older than Deployments)
- Unique feature: auto-deploys when ImageStream updates

### Why It Is Used
- Automatic deployments triggered by image changes without webhooks

### Use Cases
- **Scenario:** Security pushed patched base image to ImageStream. DC auto-deployed new pods with patched image—no manual trigger needed
- **Scenario:** Legacy app using DC. Migrating to Deployment for better Kubernetes compatibility

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
- Maintains specified number of pod copies running
- Created automatically by Deployments—rarely touch directly

### Why It Is Used
- Self-healing: if pod dies, ReplicaSet creates replacement

### Use Cases
- **Scenario:** Deployment stuck at "1/3 replicas available". Described ReplicaSet, found "ImagePullBackOff"—wrong image tag
- **Scenario:** Old ReplicaSets piling up. Set `revisionHistoryLimit: 3` in Deployment to auto-cleanup

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
- Old version of ReplicaSet—still used by DeploymentConfigs
- Legacy resource, avoid for new workloads

### Why It Is Used
- Powers DeploymentConfigs internally

### Use Cases
- **Scenario:** DC rollout stuck. Checked RC events—found "quota exceeded" error that DC didn't show clearly
- **Scenario:** Migrating old apps from DC/RC to Deployment/RS for better portability

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
- For apps needing stable identity: predictable pod names (app-0, app-1) and dedicated storage per pod
- Used for databases, Kafka, Elasticsearch

### Why It Is Used
- Stateful apps need consistent identity and persistent storage that follows the pod

### Use Cases
- **Scenario:** Running 3-node Postgres cluster. Each pod (postgres-0, postgres-1, postgres-2) has its own PVC that survives restarts
- **Scenario:** Kafka broker restarted. Same pod name, same PVC—no data loss, partitions didn't reassign

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
- Runs one pod on every node (or selected nodes)
- New node added? Pod automatically scheduled there

### Why It Is Used
- Node-level agents: logging, monitoring, security scanners

### Use Cases
- **Scenario:** Fluentd DaemonSet ships logs from every node. Added 3 new nodes—Fluentd pods auto-deployed, logs started flowing immediately
- **Scenario:** Security team deployed vulnerability scanner as DaemonSet—guaranteed coverage on every node

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
- Runs a task until completion, then stops
- Retries on failure (configurable)

### Why It Is Used
- One-time tasks: migrations, batch processing, data imports

### Use Cases
- **Scenario:** Database migration before new release. Created Job, it ran migration script, completed successfully, deployment proceeded
- **Scenario:** Job failed twice, succeeded on third retry—`backoffLimit: 3` saved manual intervention

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
- Scheduled Jobs using cron syntax (like Linux crontab)
- Creates new Job at each scheduled time

### Why It Is Used
- Recurring tasks: backups, cleanups, report generation

### Use Cases
- **Scenario:** Database backup runs every night at 2 AM via CronJob. `concurrencyPolicy: Forbid` prevents overlap if backup runs long
- **Scenario:** Weekly cleanup job prunes old data. `failedJobsHistoryLimit: 3` keeps last 3 failures for debugging

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
- Limits how many pods can be down during voluntary disruptions (drains, upgrades)
- Example: "always keep at least 2 pods running"

### Why It Is Used
- Prevents outages during cluster maintenance

### Use Cases
- **Scenario:** Cluster upgrade started draining nodes. PDB with `minAvailable: 2` blocked drain until pods moved safely—zero downtime
- **Scenario:** Without PDB, upgrade drained node with all 3 replicas—app went down. Added PDB, never happened again

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
- Stable IP/DNS name that load-balances traffic to pods
- Pods change IPs, Service stays constant

### Why It Is Used
- Internal service discovery—apps connect to service name, not pod IPs

### Use Cases
- **Scenario:** Frontend connects to `backend-svc:8080`. Backend pods restart, get new IPs—frontend unaffected, Service handles it
- **Scenario:** Created headless Service (`clusterIP: None`) for StatefulSet—each pod gets direct DNS: `postgres-0.postgres-headless`

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
- Exposes Services externally with a URL (myapp.apps.cluster.com)
- Built-in TLS termination and traffic splitting

### Why It Is Used
- External access to apps with HTTPS—OpenShift-native, no nginx/traefik needed

### Use Cases
- **Scenario:** Created Route with edge TLS. Got `https://myapp.apps.cluster.com` with auto HTTP-to-HTTPS redirect in one YAML
- **Scenario:** Canary deployment: Route splits 90% to v1, 10% to v2 using `alternateBackends`. Tested v2 with real traffic safely

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
- Standard Kubernetes ingress resource
- OpenShift auto-converts Ingress to Route behind the scenes

### Why It Is Used
- Portability—same YAML works on OpenShift and other Kubernetes platforms

### Use Cases
- **Scenario:** Helm chart created Ingress objects. Worked on OpenShift without modification—auto-converted to Routes
- **Scenario:** Team migrated from GKE. Kept existing Ingress YAMLs, OpenShift handled them natively

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
- Firewall rules for pods—control which pods can talk to which
- Whitelist model: once applied, only allowed traffic gets through

### Why It Is Used
- Security and compliance—isolate sensitive workloads

### Use Cases
- **Scenario:** Auditor asked "can any pod reach database?" Showed NetworkPolicy allowing only app pods to connect. Audit passed
- **Scenario:** Created deny-all policy, then allowed only frontend→backend→database. Rogue pod in namespace can't reach anything

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
- List of pod IPs behind a Service
- Auto-managed based on Service selector

### Why It Is Used
- Debug service connectivity—shows if pods are actually backing the Service

### Use Cases
- **Scenario:** Service returning 503. Checked Endpoints—empty! Pods had wrong labels, selector didn't match. Fixed labels, worked
- **Scenario:** Created manual Endpoints for external database—apps connect to `external-db` Service like it's internal

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
- Actual storage provisioned in infrastructure (EBS, NFS, Azure Disk)
- Cluster-scoped, not namespace-scoped

### Why It Is Used
- Durable storage that survives pod restarts and deletions

### Use Cases
- **Scenario:** PVC stuck Pending. Checked PVs—no matching size available. Created larger PV, PVC bound successfully
- **Scenario:** Set `reclaimPolicy: Retain` for database PV—even if PVC deleted accidentally, data is preserved

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
- Request for storage: "I need 10Gi of ReadWriteOnce storage"
- Binds to PV or triggers dynamic provisioning

### Why It Is Used
- Apps request storage without knowing infrastructure details

### Use Cases
- **Scenario:** Created PVC requesting 10Gi. StorageClass auto-provisioned EBS volume. No admin intervention needed
- **Scenario:** Database running low on disk. Patched PVC to 20Gi—volume expanded online, zero downtime

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
- Template defining how to provision storage (type, IOPS, provisioner)
- Enables dynamic provisioning—create PVC, get PV automatically

### Why It Is Used
- Self-service storage without admin creating PVs manually

### Use Cases
- **Scenario:** Created "fast-ssd" and "standard-hdd" StorageClasses. Devs choose based on need—databases get fast, logs get standard
- **Scenario:** Set `volumeBindingMode: WaitForFirstConsumer`—PV created in same zone as pod, avoiding cross-zone latency

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
- Point-in-time backup of a PVC
- Can create new PVC from snapshot (clone)

### Why It Is Used
- Fast backups and cloning without stopping applications

### Use Cases
- **Scenario:** Before database migration, created VolumeSnapshot. Migration failed—restored from snapshot in minutes instead of hours from backup
- **Scenario:** Needed prod data in staging. Snapshot prod PVC, created new PVC from snapshot in staging namespace—instant clone

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
- Builds container images from source code inside OpenShift
- Supports S2I (Source-to-Image), Dockerfile, and Pipeline strategies

### Why It Is Used
- Built-in CI—no external Jenkins needed for image builds

### Use Cases
- **Scenario:** S2I BuildConfig for Node.js app. Push to Git, webhook triggers build, image pushed to internal registry, deployment triggered—full CI/CD
- **Scenario:** Team doesn't know Docker. S2I builds image from source code automatically—no Dockerfile required

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
- Single execution of a BuildConfig
- Has its own logs and status

### Why It Is Used
- Track and debug individual build executions

### Use Cases
- **Scenario:** Build failed. Ran `oc logs build/myapp-15`—found npm install failed due to missing dependency
- **Scenario:** Checking build history to find which build introduced a bug—each build shows the triggering commit

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
- Pointer to container images—tracks tags and triggers deployments on changes
- Like a smart alias that remembers image history

### Why It Is Used
- Image lifecycle management and automatic deployment triggers

### Use Cases
- **Scenario:** Base image updated with security patch. ImageStream detected change, DeploymentConfig auto-rolled out new pods
- **Scenario:** Promoted image: `oc tag dev/myapp:tested prod/myapp:latest`—same image SHA, no rebuild

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
- Specific tag within an ImageStream (myapp:v1.2.0)
- Points to exact image SHA

### Why It Is Used
- Version tracking and rollback capability

### Use Cases
- **Scenario:** Need to rollback? `oc tag myapp@sha256:abc123 myapp:latest`—latest now points to old known-good image
- **Scenario:** Compliance audit asked "what image ran on Jan 5th?"—ImageStreamTag history showed exact SHA

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
- Reference to image by SHA digest (immutable)
- Tags can move, SHA never changes

### Why It Is Used
- Guarantee exact image for compliance and reproducibility

### Use Cases
- **Scenario:** Locked production to SHA reference—even if someone pushes to :latest, prod doesn't change accidentally
- **Scenario:** "Works in dev, fails in prod"—compared SHAs, found dev and prod had different images despite same tag

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
- Represents someone who authenticated to the cluster
- Auto-created on first login via identity provider

### Why It Is Used
- RBAC bindings reference Users for permissions

### Use Cases
- **Scenario:** `oc get users` showed ex-employee still listed. Deleted User and Identity objects as part of offboarding
- **Scenario:** Granting cluster-admin: `oc adm policy add-cluster-role-to-user cluster-admin john`

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
- Collection of users for bulk permission assignment
- Can sync from LDAP/Active Directory

### Why It Is Used
- Manage permissions for teams, not individuals

### Use Cases
- **Scenario:** 20 developers need access to dev namespace. Created Group, one RoleBinding—done. New dev joins, add to Group
- **Scenario:** LDAP sync pulls "OpenShift-Admins" AD group hourly. Add user in AD, they get cluster-admin automatically

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
- Set of permissions within a namespace
- Defines verbs (get, create, delete) on resources (pods, secrets)

### Why It Is Used
- Least privilege—give only the permissions needed

### Use Cases
- **Scenario:** CI/CD needs to deploy but not delete PVCs. Created custom Role with only deployment permissions
- **Scenario:** Auditor needs read-only access. Used built-in "view" Role—can see everything, change nothing

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
- Connects Users/Groups/ServiceAccounts to Roles
- Scoped to one namespace

### Why It Is Used
- This is how permissions are actually granted

### Use Cases
- **Scenario:** `oc adm policy add-role-to-group edit developers -n myproject`—developers Group gets edit Role in myproject namespace
- **Scenario:** CI/CD ServiceAccount in 'cicd' namespace needs to deploy to 'prod'—RoleBinding in prod references SA from cicd

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
- Like Role but cluster-wide scope
- Can include non-namespaced resources (nodes, PVs)

### Why It Is Used
- Permissions for cluster-level resources and cross-namespace access

### Use Cases
- **Scenario:** Monitoring tool needs to list pods in ALL namespaces. ClusterRole with get/list pods, bound cluster-wide
- **Scenario:** Custom ClusterRole for viewing nodes and PVs—these resources don't live in namespaces

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
- Binds ClusterRole to subjects cluster-wide
- Grants permissions across ALL namespaces

### Why It Is Used
- Admin access, monitoring, operators that need cluster-wide permissions

### Use Cases
- **Scenario:** Platform team Group bound to cluster-admin ClusterRole—full cluster access
- **Scenario:** Prometheus ServiceAccount needs metrics from all namespaces—ClusterRoleBinding to cluster-monitoring-view

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
- Identity for pods and automation (not humans)
- Pods use SA to authenticate to Kubernetes API

### Why It Is Used
- Pods need identity for API access and image pulls

### Use Cases
- **Scenario:** CI/CD pipeline runs as ServiceAccount with deploy permissions. Can deploy apps but can't delete namespaces
- **Scenario:** Pods need to pull from private registry. Linked imagePullSecret to ServiceAccount—all pods using that SA can pull

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
- OpenShift's wrapper around namespace—adds ownership, quotas, network isolation
- What developers request; admins approve

### Why It Is Used
- Multi-tenancy with isolation and resource control

### Use Cases
- **Scenario:** Team requested project via self-service. Template auto-applied with quotas, NetworkPolicies, default SA
- **Scenario:** Deleting project cleans up ALL resources inside—pods, services, routes, PVCs. One command cleanup

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
- Virtual cluster within a cluster—isolates resources by name
- Same name can exist in different namespaces

### Why It Is Used
- Logical separation for environments, teams, apps

### Use Cases
- **Scenario:** myapp-dev, myapp-staging, myapp-prod—same manifests, different namespaces
- **Scenario:** Developers have admin in their namespace but can't touch other namespaces—RBAC boundary

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
- Sets min/max/default CPU and memory per pod/container
- Prevents runaway pods and enforces sane defaults

### Why It Is Used
- Protect cluster from resource hogs and ensure fair sharing

### Use Cases
- **Scenario:** Pod without limits tried to request 100Gi memory—LimitRange rejected it at admission
- **Scenario:** Developer forgot to set limits. LimitRange applied default of 256Mi memory, 100m CPU automatically

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
- Caps total resource consumption per namespace
- "This namespace can use max 10 cores and 20Gi memory total"

### Why It Is Used
- Fair resource distribution across teams

### Use Cases
- **Scenario:** Team tried to scale to 50 replicas—quota blocked it because total CPU would exceed limit
- **Scenario:** Chargeback: quota doubles as tracking. Each team's namespace has quota equal to their allocated budget

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
- Kubernetes' activity log—records what happened to resources
- Short-lived (1 hour default)

### Why It Is Used
- First place to check when troubleshooting

### Use Cases
- **Scenario:** Pod stuck in Pending. Events showed "FailedScheduling: insufficient cpu"—needed more nodes
- **Scenario:** Pod in ImagePullBackOff. Event showed "unauthorized: authentication required"—missing imagePullSecret

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
- Software that extends Kubernetes with custom controllers
- Automates complex app lifecycle (install, upgrade, backup)

### Why It Is Used
- Day 2 operations automation—operators know how to manage their apps

### Use Cases
- **Scenario:** Elasticsearch Operator manages ES cluster. Scales, upgrades, handles rolling restarts—I just set desired version
- **Scenario:** Jaeger Operator creates entire tracing infrastructure from one CR

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
- Marketplace for operators in OpenShift Console
- One-click install of Red Hat, certified, and community operators

### Why It Is Used
- Easy discovery and installation of operators

### Use Cases
- **Scenario:** Needed Kafka. Searched OperatorHub, installed Strimzi operator in 2 clicks
- **Scenario:** Disconnected cluster—mirrored operator catalog to internal registry, OperatorHub shows only approved operators

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
- Tracks operator version and update channel
- Controls automatic vs manual updates

### Why It Is Used
- Operator lifecycle management

### Use Cases
- **Scenario:** Production on "stable" channel with manual approval. Test gets updates first, we approve prod after validation
- **Scenario:** Operator update broke things. Changed subscription to pin specific version until fix available

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
- Points to operator index/repository
- Where OperatorHub gets its list of available operators

### Why It Is Used
- Custom operators and disconnected environments

### Use Cases
- **Scenario:** Disconnected cluster—created CatalogSource pointing to internal mirror with curated operator list
- **Scenario:** Internal team built custom operator. Added CatalogSource so it appears in OperatorHub

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
- Represents installed version of an operator
- Contains RBAC, CRDs, and deployment specs

### Why It Is Used
- Tracks what's installed and enables uninstall/upgrade

### Use Cases
- **Scenario:** `oc get csv`—showed all operators and their phases (Succeeded, Failed, Pending)
- **Scenario:** Operator stuck. Deleted CSV, subscription recreated it, fixed the stuck state

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

## CustomResourceDefinitions (CRDs)

### Definition
- Extends Kubernetes API with new resource types
- Your own "kind: MyApp" resources

### Why It Is Used
- Operators use CRDs to accept configuration

### Use Cases
- **Scenario:** Kafka operator installed CRD for "Kafka" resource. Now I can create Kafka clusters with `kind: Kafka`
- **Scenario:** Team built Backup CRD. Users create Backup resources, controller handles the actual backup logic

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
- Cluster-wide config for identity providers
- How users prove who they are

### Why It Is Used
- Enterprise SSO integration

### Use Cases
- **Scenario:** Configured LDAP provider. Users now login with AD credentials
- **Scenario:** OIDC integration with Okta—users get SSO across all tools

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
- Configures how users authenticate to OpenShift
- Connects to LDAP, OIDC, HTPasswd, GitHub, etc.

### Why It Is Used
- Enterprise SSO and token management

### Use Cases
- **Scenario:** Added LDAP identity provider. Users now login with AD credentials
- **Scenario:** Break-glass admin—kept HTPasswd with emergency admin account alongside LDAP

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
- Enables/disables alpha and beta Kubernetes features
- Usually leave at default

### Why It Is Used
- Test new features or disable problematic ones

### Use Cases
- **Scenario:** Needed topology-aware scheduling (beta). Enabled FeatureGate in test cluster first
- **Scenario:** Support said disable a specific feature causing issues—used FeatureGate override

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
- Controls how pods get placed on nodes
- Profiles for different placement strategies

### Why It Is Used
- Optimize placement for density, spread, or custom rules

### Use Cases
- **Scenario:** High-density profile—packs pods tightly to minimize node count and save costs
- **Scenario:** Spread profile for HA—distributes replicas across failure domains

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
- Cluster-wide proxy configuration for outbound traffic
- Sets HTTP_PROXY, HTTPS_PROXY, NO_PROXY

### Why It Is Used
- Required in enterprise networks with egress proxies

### Use Cases
- **Scenario:** Cluster behind corporate firewall. Configured proxy for image pulls from external registries
- **Scenario:** NO_PROXY set for internal registry and API—direct access to internal resources

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

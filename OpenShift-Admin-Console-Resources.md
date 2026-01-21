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
- This is the first thing I check every morning—it's basically your cluster's heartbeat monitor showing health, alerts, and resource consumption in one place
- Think of it as the cockpit view; if something's wrong anywhere in the cluster, you'll see red flags here first

### Why It Is Used
- In production, I use this to quickly triage before touching anything—saves me from blindly deploying into a cluster that's already on fire.

### Use Cases
- Morning sanity check before rolling out changes—are nodes healthy? Any firing alerts?
- When on-call gets paged at 2 AM, this is where you land first to assess blast radius

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
- Grafana-style dashboards powered by Prometheus—this is where you go beyond "is it up" to "why is it slow"
- I've spent hours here correlating CPU spikes with deployment times to prove it wasn't my code causing issues

### Why It Is Used
- When leadership asks "how's the cluster performing this quarter?", these dashboards give you the data to back up your capacity planning requests.

### Use Cases
- Tracking memory trends to justify adding more worker nodes before Black Friday traffic
- Correlating pod restart spikes with specific deployments—caught a memory leak this way once

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
- These are your actual VMs or bare-metal servers running the workloads—workers run your apps, masters run the control plane
- Every pod you deploy ultimately lands on a node; if nodes are unhealthy, nothing else matters

### Why It Is Used
- You can't troubleshoot scheduling issues without understanding node capacity. "Pending" pods? First thing I check is node resource pressure.

### Use Cases
- Cordoning a node before patching—learned the hard way that draining without cordoning first causes pods to reschedule back
- Labeling nodes for workload isolation—we run GPU workloads on specific nodes using node selectors

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
- The Machine object is OpenShift's view of the underlying infrastructure—it's the glue between your cloud provider's VM and the Kubernetes node
- When a node shows "NotReady", I always check the Machine status because the problem might be at the infrastructure layer, not Kubernetes

### Why It Is Used
- Debugging "why won't my new node come up" almost always involves looking at Machine status and events—it tells you if the VM even got created.

### Use Cases
- Node stuck in provisioning? Check if the Machine is waiting on cloud API rate limits or quota issues
- Correlating intermittent node failures with underlying VM health checks failing on AWS/Azure

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
- Think of it as a ReplicaSet but for infrastructure—you say "I want 5 workers in us-east-1a" and it maintains that count
- This is how you scale your cluster without manually provisioning VMs; change the replica count and machines appear

### Why It Is Used
- Autoscaling and disaster recovery depend on MachineSets. If a node dies, the MachineSet spins up a replacement automatically.

### Use Cases
- Creating a separate MachineSet for infra nodes so router and monitoring pods don't compete with app workloads
- Scaling up before a big release, then scaling back down after—saved significant cloud costs this way

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
- This is how you touch the RHCOS operating system—need to add a CA cert, change a kernel parameter, or modify kubelet config? MachineConfig is your answer
- Fair warning: applying a MachineConfig triggers a rolling reboot of all nodes in that pool, so plan your maintenance windows

### Why It Is Used
- Security team says "add our corporate CA to all nodes"—MachineConfig is the only supported way to do this without breaking cluster upgrades.

### Use Cases
- Setting vm.max_map_count for Elasticsearch—without this, ES pods crash on startup
- Adding custom chrony config because the default NTP servers weren't reachable in our air-gapped environment

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
- The atomic unit of deployment—one or more containers sharing network and storage. But here's the thing: you almost never create pods directly
- Pods are cattle, not pets. They die, get evicted, OOMKilled—that's normal. Your job is to make sure the controllers recreate them properly

### Why It Is Used
- When something breaks, you're looking at pod logs, describing pods for events, exec'ing in to check filesystem state. This is where debugging lives.

### Use Cases
- Pod stuck in CrashLoopBackOff? Check logs, then previous logs (--previous flag is your friend for containers that restart)
- Exec into a pod to verify it can reach the database—faster than setting up tcpdump

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
- The workhorse for stateless apps—you declare the desired state, it figures out how to get there with rolling updates
- Under the hood, it creates ReplicaSets. Each rollout = new ReplicaSet. That's how rollback works—it just scales the old RS back up

### Why It Is Used
- 90% of our apps use Deployments. It handles rolling updates, rollbacks, and scaling without me writing custom logic.

### Use Cases
- Rollback saved us during a bad release—one command and we're back to the previous version in seconds
- Set maxUnavailable to 0 for zero-downtime deploys; learned this after accidentally taking down production during a rollout

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
- OpenShift's original deployment mechanism—older than Kubernetes Deployments but has unique features like ImageStream triggers
- Honestly, we're migrating away from DCs to Deployments, but legacy apps still use them and you'll see them in older clusters

### Why It Is Used
- The killer feature is ImageStream triggers—push a new image and DC automatically rolls out. No CI/CD webhook needed.

### Use Cases
- Legacy apps that rely on automatic rollouts when base images update in ImageStreams
- Running database migrations in lifecycle hooks before the new version starts accepting traffic

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
- The controller that actually maintains your pod count—Deployments create these, you rarely touch them directly
- Each deployment revision creates a new ReplicaSet. Old ones stick around (scaled to 0) for rollback capability

### Why It Is Used
- When a rollout is stuck, looking at ReplicaSet events tells you why the new pods aren't coming up—usually image pull issues or resource constraints.

### Use Cases
- Debugging stuck deployments—RS events show "FailedCreate" with the actual error message
- Cleaning up old ReplicaSets to reduce etcd bloat in clusters with frequent deployments

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
- The OG replication controller—superseded by ReplicaSets but still powers DeploymentConfigs under the hood
- If you're managing a cluster with old workloads, you'll see these. New clusters? Probably never

### Why It Is Used
- DeploymentConfigs create RCs, so when debugging DC rollouts, you end up looking at RC status and events.

### Use Cases
- Stuck DC rollout? Check the RC events for the real error—usually quota exceeded or image pull failures
- Migration projects: identifying workloads still using DC/RC that need to move to Deployment/RS

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
- For apps that care about identity—databases, Kafka, anything that says "I'm node-0 and I need to stay node-0"
- Pods get sticky names (myapp-0, myapp-1) and dedicated PVCs that follow them through restarts. Deployments can't do this

### Why It Is Used
- You can't run a proper database cluster with Deployments. StatefulSets give you ordered startup, stable DNS, and persistent storage per pod.

### Use Cases
- Running a 3-node PostgreSQL cluster where each node needs its own persistent volume and predictable hostname for replication
- Kafka brokers that need stable network identity so partitions don't get reassigned on every pod restart

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
- "Run this pod on every node"—that's it. Perfect for agents that need to be everywhere: logging, monitoring, security scanning
- Add a new node? DaemonSet automatically schedules a pod there. No manual intervention needed

### Why It Is Used
- Node-level concerns like log forwarding, metrics collection, and security agents have to run on every node. DaemonSet is the only way to guarantee that.

### Use Cases
- Fluentd on every node shipping logs to Splunk—missing a node means missing logs, which means audit failures
- Node-exporter for Prometheus—without it on every node, your monitoring dashboards have blind spots

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
- "Run this once and make sure it finishes"—unlike Deployments that keep pods running forever, Jobs are for tasks with an end
- It'll retry on failure (configurable), and you can run multiple pods in parallel for batch processing

### Why It Is Used
- Database migrations, data imports, one-time cleanups—anything that shouldn't run continuously but needs Kubernetes' scheduling and retry logic.

### Use Cases
- Database migrations as part of deployment pipeline—Job runs before the new app version starts
- Bulk data processing: spin up 10 parallel pods to process a queue, Job completes when all items are done

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
- Cron for Kubernetes—schedule Jobs to run at specific times using standard cron syntax
- Each execution creates a new Job object. Watch out for overlap if your job takes longer than the schedule interval

### Why It Is Used
- Backups at 2 AM, daily report generation, weekly cleanup—anything you'd put in crontab but want Kubernetes to manage.

### Use Cases
- Nightly database backups with `concurrencyPolicy: Forbid` so a slow backup doesn't cause two running simultaneously
- Hourly cleanup job that prunes old data—`failedJobsHistoryLimit` keeps failed runs around for debugging

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
- Your safety net during cluster maintenance—tells Kubernetes "you can't evict pods if it would drop us below X available replicas"
- Without PDBs, a node drain can take down all your pods at once. Ask me how I learned this lesson

### Why It Is Used
- Cluster upgrades drain nodes one by one. PDB ensures your app stays up by blocking drains that would cause outage.

### Use Cases
- Critical payment service with PDB requiring minAvailable: 2—upgrade can't proceed if it would leave us with 1 pod
- Learned to always create PDBs after an upgrade took down all 3 replicas because they were on the same node

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
- The stable front door to your pods—pods come and go, but the Service IP and DNS name stay constant
- It's basically an internal load balancer. Traffic hits the Service, Service routes to healthy pods. Simple but critical

### Why It Is Used
- Pods get random IPs every restart. Without Services, your apps would need to discover each other constantly. Service DNS (myapp.namespace.svc) solves this.

### Use Cases
- ClusterIP for internal communication—frontend talks to backend-service, not individual pod IPs
- Headless Service (clusterIP: None) for StatefulSets where each pod needs direct addressability

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
- OpenShift's way of exposing apps to the outside world—creates a hostname like myapp.apps.cluster.example.com
- Under the hood, it's HAProxy handling TLS termination, path routing, and traffic splitting. Production-grade ingress out of the box

### Why It Is Used
- Need external HTTPS access with automatic cert management? Route + OpenShift's built-in router handles it without deploying nginx-ingress or traefik.

### Use Cases
- Edge TLS termination with auto-redirect from HTTP—one YAML and you have secure external access
- A/B testing: split traffic 90/10 between v1 and v2 using alternateBackends. Real canary deployments without service mesh

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
- Standard Kubernetes ingress resource—OpenShift automatically converts these to Routes behind the scenes
- Useful when you're migrating from vanilla Kubernetes or using Helm charts that expect Ingress resources

### Why It Is Used
- Portability: same YAML works on OpenShift and other K8s distros. Good for multi-cloud or hybrid scenarios.

### Use Cases
- Running Helm charts that create Ingress objects—they just work on OpenShift, no modification needed
- Team familiar with Ingress from GKE/EKS can use same patterns; OpenShift handles the translation to Routes

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
- Firewall rules for pods—without them, every pod can talk to every other pod. That's a security audit failure waiting to happen
- Once you apply any NetworkPolicy to a namespace, it becomes default-deny for that traffic type. Whitelist model

### Why It Is Used
- Compliance requires network segmentation. NetworkPolicies prove to auditors that your database isn't exposed to the entire cluster.

### Use Cases
- Database pods only accepting connections from specific app pods—even if someone deploys a rogue pod, it can't reach the DB
- Namespace isolation: production namespace can't talk to development, preventing accidental cross-environment calls

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
- The actual list of pod IPs behind a Service—Kubernetes maintains this automatically based on which pods match the Service selector
- When you can't reach a service, checking Endpoints tells you if any pods are actually backing it

### Why It Is Used
- "Service exists but returns 503"—first check Endpoints. Empty or missing IPs means selector doesn't match any running pods.

### Use Cases
- Debugging service connectivity: "why can't I reach my app?"—Endpoints shows 0 addresses, pods have wrong labels
- Manually creating Endpoints for external services—makes external database look like an in-cluster service to your apps

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
- The actual storage provisioned in your infrastructure—could be AWS EBS, Azure Disk, NFS, whatever your cluster supports
- PVs exist at cluster scope, not namespace. They get bound to PVCs when a claim matches their capacity and access mode

### Why It Is Used
- Stateful apps need storage that survives pod restarts. PVs are that durable storage layer decoupled from pod lifecycle.

### Use Cases
- Investigating "Pending" PVCs—check if a PV with matching specs exists or if dynamic provisioning is configured
- Setting reclaimPolicy to Retain for databases—you don't want to accidentally delete production data when someone deletes the PVC

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
- Developer's request for storage—"I need 10Gi of fast storage"—without caring if it's EBS, Azure Disk, or NFS
- PVC either binds to an existing PV or triggers dynamic provisioning via StorageClass. Either way, developer gets their volume

### Why It Is Used
- Abstracts storage infrastructure from application teams. They request what they need; platform team handles the backend.

### Use Cases
- PVC stuck in Pending? Either no matching PV exists, StorageClass can't provision, or you hit quota limits
- Expanding PVC online without downtime—if StorageClass supports it, just patch the spec.resources.requests.storage

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
- Your storage menu—defines different storage tiers with different performance characteristics and provisioners
- When a PVC references a StorageClass, it triggers dynamic provisioning. No admin intervention needed per request

### Why It Is Used
- Self-service storage: developers pick "fast-ssd" or "cheap-hdd" StorageClass, platform provisions automatically. Scales without you being a bottleneck.

### Use Cases
- Default StorageClass for general workloads, premium class for databases needing IOPS guarantees
- WaitForFirstConsumer binding mode—delays provisioning until pod is scheduled, ensures volume is in the right availability zone

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
- Point-in-time backup of a PVC at the storage layer—much faster than copying data because it's typically copy-on-write
- You can create a new PVC from a snapshot, essentially cloning your data instantly

### Why It Is Used
- Pre-upgrade database backups, cloning prod data to test environment, disaster recovery—all without application downtime.

### Use Cases
- Snapshot before database migration—if migration fails, restore from snapshot instead of waiting for backup restore
- "Clone production to staging"—snapshot prod PVC, create new PVC from snapshot in staging namespace

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
- OpenShift's built-in CI—point it at Git repo, it builds your image using S2I, Dockerfile, or pipeline strategy
- No need for external Jenkins or GitHub Actions just to build images. BuildConfig does it inside the cluster with full access to internal registries

### Why It Is Used
- Reduces external dependencies. Build runs in-cluster, pushes to internal registry, triggers deployment—all without leaving OpenShift.

### Use Cases
- S2I builds for standard frameworks—Node.js app builds without writing a Dockerfile, S2I handles it
- Webhook triggers: push to main branch, BuildConfig automatically starts building. Instant CI/CD

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
- The actual execution of a BuildConfig—one BuildConfig, many Builds over time
- Each Build has its own logs, status, and output. Failed build? Check Build logs, not BuildConfig

### Why It Is Used
- When build fails, Build object has the logs showing exactly what went wrong—npm install failed, Dockerfile syntax error, whatever.

### Use Cases
- Debugging failed builds: `oc logs build/myapp-15` shows the actual compilation errors
- Checking which commit triggered which build—useful when tracking down "which build introduced the bug"

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
- A pointer to images rather than the images themselves—tracks tags across registries and triggers deployments when images change
- Think of it as a smart alias: myapp:latest in ImageStream can point to different actual images over time, and OpenShift tracks the history

### Why It Is Used
- Image promotion across environments without rebuilding: tag dev image as prod, DeploymentConfig in prod namespace auto-deploys. No rebuild, same tested image.

### Use Cases
- Scheduled imports from external registry—ImageStream polls Quay.io daily, auto-deploys when base image gets security updates
- Image promotion: `oc tag dev/myapp:tested prod/myapp:latest` promotes through environments using the same image SHA

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
- A specific version pointer within an ImageStream—myapp:v1.2.0 points to one specific image SHA
- Unlike mutable Docker tags, you can see exactly which image SHA a tag pointed to at any point in history

### Why It Is Used
- Audit trail: compliance asks "what image was running on January 5th?"—ImageStreamTag history shows the exact SHA.

### Use Cases
- Rolling back by re-tagging: `oc tag myapp@sha256:abc123 myapp:latest` instantly points latest back to a known-good image
- Tracking which exact image is deployed in each environment—prod:latest might be different SHA than dev:latest

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
- Reference to image by SHA digest, not tag—absolutely immutable, tags can move but SHA never lies
- When you need to prove exactly what binary was deployed, ImageStreamImage gives you the cryptographic proof

### Why It Is Used
- Security and compliance: "prove this exact image passed security scan" requires SHA reference, not mutable tag.

### Use Cases
- Locking deployments to specific SHA for compliance—no accidental updates when someone pushes to :latest
- Debugging "works in dev, fails in prod"—compare SHAs to confirm same image or catch tag divergence

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
- Represents someone who authenticated to the cluster—could be from LDAP, OIDC, HTPasswd, whatever identity provider you configured
- User objects are created automatically on first login. You manage permissions, not user creation typically

### Why It Is Used
- RBAC bindings reference Users. "Give John cluster-admin"—you need the User object to bind roles to.

### Use Cases
- Auditing who has access: `oc get users` shows everyone who's ever logged in
- Offboarding: delete User and Identity objects when someone leaves (though usually handled by IdP)

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
- Collection of users for bulk permission assignment—way easier than binding roles to 50 individual users
- Can be manually created or synced from LDAP/AD. We sync from Active Directory every hour

### Why It Is Used
- Team-based access: "developers" group gets edit on dev namespace, "sre" group gets admin. One binding per team, not per person.

### Use Cases
- LDAP group sync: AD group "OpenShift-Admins" auto-syncs to OpenShift Group, gets cluster-admin
- Project onboarding: add team's AD group to namespace with one RoleBinding instead of managing individuals

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
- A set of permissions scoped to one namespace—"can get/list pods" or "can create deployments"
- Built-in roles like 'edit' and 'view' cover most cases. Create custom roles for fine-grained control

### Why It Is Used
- Least privilege: CI/CD pipeline needs to deploy but shouldn't delete PVCs. Custom Role with only deployment permissions.

### Use Cases
- Read-only role for auditors—can view everything but change nothing
- CI/CD service account role: create/update Deployments and ConfigMaps, nothing else

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
- The glue between "who" (User/Group/ServiceAccount) and "what they can do" (Role)
- Scoped to namespace: RoleBinding in 'production' namespace only grants permissions in 'production'

### Why It Is Used
- This is how access actually gets granted. Role defines permissions, RoleBinding assigns them to identities.

### Use Cases
- Grant 'developers' group edit access to their namespace: one RoleBinding, whole team has access
- Cross-namespace service account access: SA in 'cicd' namespace needs to deploy to 'production'—RoleBinding in production references SA from cicd

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
- Like Roles but cluster-wide—can include permissions for cluster-scoped resources (nodes, PVs) that don't live in namespaces
- Built-in cluster-admin, cluster-reader, etc. cover most needs. Custom ClusterRoles for operators and specialized access

### Why It Is Used
- Some things aren't namespaced: nodes, persistent volumes, CRDs. You need ClusterRole to grant access to these.

### Use Cases
- Read-only cluster access for monitoring tools: ClusterRole with get/list on nodes, pods across all namespaces
- Operator ClusterRole: needs to watch CRDs and manage resources cluster-wide

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
- Binds ClusterRole to subjects with cluster-wide scope—subject gets those permissions everywhere
- Be careful: ClusterRoleBinding with 'edit' ClusterRole = edit access to ALL namespaces

### Why It Is Used
- Platform admins, monitoring systems, operators—anything needing cross-namespace or cluster-scoped access.

### Use Cases
- Platform team gets cluster-admin via ClusterRoleBinding to their AD group
- Prometheus ServiceAccount needs to scrape metrics from all namespaces—ClusterRoleBinding to cluster-monitoring-view

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
- Identity for pods and automation—when your pod calls the Kubernetes API, it authenticates as its ServiceAccount
- Every namespace has 'default' SA, but you should create dedicated SAs with minimal permissions per workload

### Why It Is Used
- Pods need identity for API access, image pulls, and external service authentication. Never use default SA for production workloads.

### Use Cases
- CI/CD pipeline SA with just enough permissions to deploy to target namespace
- SA with imagePullSecret for pulling from private registry—link secret to SA, all pods using that SA can pull

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
- OpenShift's enhanced namespace—same isolation as namespace plus display names, descriptions, and self-provisioning support
- When developers run `oc new-project`, they get a Project. It creates the underlying Namespace automatically

### Why It Is Used
- Multi-tenancy: each team gets their Project with quotas and network isolation. Self-service without cluster-admin involvement.

### Use Cases
- Developer self-service: user creates their own project, gets admin on it automatically, stays isolated from others
- Project templates: auto-apply LimitRanges, NetworkPolicies, default quotas to every new project

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
- The raw Kubernetes isolation boundary—resources in one namespace are separate from another
- In OpenShift, you usually work with Projects, but system components and operators use Namespaces directly

### Why It Is Used
- Isolation: teams can have same-named resources (ConfigMap 'config') in different namespaces without conflict.

### Use Cases
- System namespaces like openshift-monitoring, openshift-ingress—these aren't Projects, they're pure Namespaces
- Namespace labels for policy targeting: label namespace with 'environment: production', NetworkPolicy can select it

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
- Defaults and constraints for container resources in a namespace—if pod doesn't specify limits, LimitRange sets them
- Also enforces min/max: "no container can request more than 4 CPU" prevents resource hogging

### Why It Is Used
- Prevents runaway resource consumption and ensures every pod has defined limits. No more "forgot to set limits" causing OOM on shared nodes.

### Use Cases
- Default limits so developers don't have to specify them in every pod spec—reduces YAML boilerplate
- Max limits to prevent one team from requesting 64GB pods on a 128GB node, hogging half the capacity

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
- Hard limits on total resources a namespace can consume—CPU, memory, storage, object counts
- Unlike LimitRange (per-container), ResourceQuota is for the whole namespace: "team can use max 20 CPU total"

### Why It Is Used
- Fair sharing: without quotas, one namespace could consume the entire cluster. Quotas enforce boundaries.

### Use Cases
- Dev namespaces get 10 CPU, 20Gi memory quota; production gets more—prevents dev workloads from starving prod
- Quota on LoadBalancer services: "max 2 per namespace" prevents cloud cost explosion from accidental LB creation

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
- Real-time log of what's happening to your resources—scheduling decisions, image pulls, health check failures, everything
- Short-lived by default (1 hour), so check them quickly when troubleshooting or set up event forwarding

### Why It Is Used
- First place to look when something's wrong. "Pod not starting"—Events tell you it's FailedScheduling due to insufficient memory.

### Use Cases
- Pod stuck in Pending? Events show "0/5 nodes available: 5 insufficient memory"—instant diagnosis
- Image pull failures: Events show "Failed to pull image: unauthorized"—immediately know it's credentials, not network

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
- Operators running in your cluster managed by OLM (Operator Lifecycle Manager)
- They watch for CRs and reconcile state—you create a PostgresCluster CR, PostgreSQL operator provisions the actual cluster

### Why It Is Used
- Day 2 operations automation: patching, backups, scaling handled by operator logic instead of manual runbooks.

### Use Cases
- Elasticsearch operator: create one CR, get a production-ready ES cluster with proper node roles and persistent storage
- Cert-manager operator: handles Let's Encrypt certificate rotation automatically

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
- App store for operators—browse, search, and install operators from Red Hat, certified vendors, and community
- Backed by CatalogSources that define what's available. Disable sources you don't want users installing from

### Why It Is Used
- One-click operator installation with dependency resolution. Need monitoring? Install Prometheus operator from OperatorHub.

### Use Cases
- Finding the right operator: search "kafka", compare Red Hat AMQ Streams vs Strimzi community operator
- Air-gapped clusters: mirror OperatorHub content to internal registry, create custom CatalogSource pointing to it

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
- Your intent to install and maintain an operator—specifies which operator, from which catalog, on which update channel
- Subscription watches for updates; when new version appears in channel, it creates an InstallPlan

### Why It Is Used
- Declarative operator management: "I want Elasticsearch operator on stable channel, approve updates manually."

### Use Cases
- Manual approval for production: set `installPlanApproval: Manual`, review changes before operator updates
- Automatic updates for dev/test: `installPlanApproval: Automatic`, always on latest version in channel

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
- Defines where OLM finds operators—points to a container image containing operator metadata and bundle references
- Default CatalogSources pull from internet. Air-gapped? Create your own pointing to mirrored content

### Why It Is Used
- Customization and air-gap support: add internal operators, mirror Red Hat catalogs, disable community sources.

### Use Cases
- Air-gapped install: mirror redhat-operators to internal registry, create CatalogSource pointing there
- Internal operators: package your custom operator, publish to internal catalog, install via OperatorHub like any other

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
- The installed version of an operator—contains the deployment spec, RBAC requirements, and CRDs it manages
- CSV status tells you if operator is healthy: "Succeeded" means running, "Failed" means check events for errors

### Why It Is Used
- Troubleshooting operator issues: CSV status and events show why operator isn't starting—usually RBAC or resource constraints.

### Use Cases
- Operator stuck "Installing"? Describe CSV, check events for the actual error
- Verifying operator version: CSV name includes version (myoperator.v1.2.3)

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
- How you extend Kubernetes API with your own resource types—operators use CRDs to define resources like PostgresCluster, Kafka, etc.
- Once CRD exists, you can `oc get postgresclusters` just like built-in resources. API server validates against CRD schema

### Why It Is Used
- Operators need custom resources to manage. Without CRD, there's no PostgresCluster resource type for the operator to watch.

### Use Cases
- Debugging "resource type not found": CRD not installed or not in the expected API group
- Viewing all custom resources in cluster: `oc get crds` shows what operators have installed

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
- Cluster-wide config for how users prove their identity—LDAP, OIDC, HTPasswd, whatever your org uses
- Configured via the OAuth cluster resource. Change it, and authentication-operator rolls out the changes

### Why It Is Used
- Integration with corporate identity: users login with AD credentials, not separate OpenShift passwords.

### Use Cases
- LDAP integration: users authenticate against Active Directory, groups sync automatically
- OIDC for SSO: integrate with Okta/Azure AD, users get seamless login across all tools

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
- Controls token policies and identity provider configuration—token lifetime, inactivity timeout, which IdPs are enabled
- The authentication-operator manages the OAuth server pods based on this config

### Why It Is Used
- Security compliance: "tokens must expire after 8 hours" or "inactive sessions timeout after 1 hour."

### Use Cases
- Reducing token lifetime for security-sensitive clusters
- Multiple IdPs: HTPasswd for break-glass admin, LDAP for regular users, OIDC for contractors

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
- Toggle switches for alpha/beta features—enable tech preview features or disable deprecated APIs
- Warning: TechPreviewNoUpgrade literally means "enable this and you can't upgrade." Use carefully

### Why It Is Used
- Early access to features before GA, or disabling deprecated APIs to prepare for future versions.

### Use Cases
- Testing new features in non-prod before they GA
- Rare: enabling specific feature gates required by certain workloads (usually not needed)

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
- Controls how pods get placed on nodes—profiles, predicates, default node selectors
- Scheduler config is cluster-wide; affects every pod scheduling decision

### Why It Is Used
- Customizing scheduling behavior: prefer certain nodes, spread pods across zones, pack nodes tightly.

### Use Cases
- HighNodeUtilization profile to pack nodes tightly (save cost) vs LowNodeUtilization (spread for resilience)
- Default node selector ensuring all pods land on worker nodes, not accidentally on infra or master

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
- Cluster-wide egress proxy configuration—all cluster components and workloads route through corporate proxy for internet access
- Set httpProxy, httpsProxy, and noProxy (critical: include your internal domains in noProxy)

### Why It Is Used
- Enterprise requirement: all internet traffic goes through proxy for inspection/logging. Without this config, cluster can't pull images or reach external services.

### Use Cases
- Corporate proxy setup: cluster routes through proxy.corp.com:3128 for external access
- noProxy tuning: exclude internal registries, cluster network CIDRs, and service domains from proxy

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

# Jenkins Interview Preparation Guide
## 50 Scenario-Based Questions: From Basics to Advanced

---

### 📌 Introduction

This guide contains **50 carefully crafted scenario-based Jenkins interview questions** designed for DevOps engineers with 3–6 years of experience. Each question simulates real-world production problems that interviewers commonly ask to evaluate your hands-on expertise.

**How to use this guide:**
- Read the scenario carefully before looking at the solution.
- Practice explaining your thought process out loud.
- Try to implement the examples in a local Jenkins environment.
- Focus on understanding "why" rather than memorizing answers.

**What this guide covers:**
- Jenkins Architecture & Core Concepts
- Freestyle Jobs & Pipeline Development
- CI/CD Design Patterns
- Jenkins Administration & Security
- Performance Tuning & Production Debugging

---

## 📚 Table of Contents

### [Section 1: Jenkins Basics & Architecture (Q1–Q10)](#section-1-jenkins-basics--architecture-q1q10-1)

| # | Question | Key Topic |
|---|----------|-----------|
| [Q1](#q1) | Jenkins Architecture Explanation | Master-Agent Architecture |
| [Q2](#q2) | Build Stuck in Queue | Executors & Troubleshooting |
| [Q3](#q3) | Jenkins Installation on Linux | Installation & Setup |
| [Q4](#q4) | Freestyle vs Pipeline Jobs | Job Types |
| [Q5](#q5) | OutOfMemoryError Fix | JVM Memory Tuning |
| [Q6](#q6) | Auto-trigger Builds from GitHub | Webhooks & SCM Polling |
| [Q7](#q7) | Command Not Found on Agent | Environment Consistency |
| [Q8](#q8) | Parameterized Builds | Job Parameters |
| [Q9](#q9) | Disk Space & Log Management | Build Retention |
| [Q10](#q10) | Backup & Restore Jenkins | Disaster Recovery |

### [Section 2: Jenkins Jobs, Pipelines & Groovy (Q11–Q20)](#section-2-jenkins-jobs-pipelines--groovy-q11q20-1)

| # | Question | Key Topic |
|---|----------|-----------|
| [Q11](#q11) | Declarative vs Scripted Pipeline | Pipeline Syntax |
| [Q12](#q12) | Shared Libraries Setup | Code Reusability |
| [Q13](#q13) | Multi-branch Pipeline | Branch Management |
| [Q14](#q14) | Pipeline Stages & Parallel Execution | Performance |
| [Q15](#q15) | Jenkinsfile Best Practices | Code Quality |
| [Q16](#q16) | Groovy Sandbox Restrictions | Security |
| [Q17](#q17) | Post Actions & Notifications | Build Feedback |
| [Q18](#q18) | Credentials in Pipelines | Secrets Management |
| [Q19](#q19) | Triggering Downstream Jobs | Job Orchestration |
| [Q20](#q20) | Pipeline Input & Approval Gates | Manual Intervention |

### [Section 3: CI/CD Design & Real DevOps Scenarios (Q21–Q30)](#section-3-cicd-design--real-devops-scenarios-q21q30-1)

| # | Question | Key Topic |
|---|----------|-----------|
| [Q21](#q21) | Complete CI/CD Pipeline Design | End-to-End Flow |
| [Q22](#q22) | Blue-Green Deployment | Zero-Downtime Deploy |
| [Q23](#q23) | Canary Releases with Jenkins | Progressive Rollout |
| [Q24](#q24) | Database Migrations in Pipeline | Data Management |
| [Q25](#q25) | Container Build & Push | Docker Integration |
| [Q26](#q26) | Kubernetes Deployment | K8s CI/CD |
| [Q27](#q27) | Artifact Management | Nexus/Artifactory |
| [Q28](#q28) | Environment Promotion Strategy | Dev→Staging→Prod |
| [Q29](#q29) | Rollback Strategies | Failure Recovery |
| [Q30](#q30) | Monorepo CI/CD | Large Codebases |

### [Section 4: Jenkins Administration, Security & Scaling (Q31–Q40)](#section-4-jenkins-administration-security--scaling-q31q40-1)

| # | Question | Key Topic |
|---|----------|-----------|
| [Q31](#q31) | RBAC & Folder Permissions | Access Control |
| [Q32](#q32) | LDAP/AD Integration | Authentication |
| [Q33](#q33) | Securing Jenkins Credentials | Secrets Protection |
| [Q34](#q34) | Plugin Management Strategy | Maintenance |
| [Q35](#q35) | Jenkins High Availability | HA Setup |
| [Q36](#q36) | Horizontal Scaling with Agents | Scaling |
| [Q37](#q37) | Kubernetes-based Dynamic Agents | Cloud Agents |
| [Q38](#q38) | Jenkins Performance Tuning | Optimization |
| [Q39](#q39) | Audit Logging & Compliance | Governance |
| [Q40](#q40) | Upgrading Jenkins Safely | Version Management |

### [Section 5: Advanced Jenkins, Debugging & Production Issues (Q41–Q50)](#section-5-advanced-jenkins-debugging--production-issues-q41q50-1)

| # | Question | Key Topic |
|---|----------|-----------|
| [Q41](#q41) | Pipeline Hanging / Stuck Builds | Debugging |
| [Q42](#q42) | Agent Connection Failures | Connectivity |
| [Q43](#q43) | Plugin Conflicts & Crashes | Troubleshooting |
| [Q44](#q44) | Slow Pipeline Diagnosis | Performance Debug |
| [Q45](#q45) | Git Checkout Failures | SCM Issues |
| [Q46](#q46) | Workspace Corruption | File System |
| [Q47](#q47) | Memory Leaks in Jenkins | Resource Monitoring |
| [Q48](#q48) | Migration to New Jenkins Server | Infrastructure |
| [Q49](#q49) | Jenkins in Air-Gapped Environment | Offline Setup |
| [Q50](#q50) | Designing Jenkins for Enterprise | Architecture |

---

## Section 1: Jenkins Basics & Architecture (Q1–Q10)

---

<a id="q1"></a>
### Q1. Your team just started using Jenkins. The manager asks you to explain how Jenkins works internally. How would you describe the Jenkins architecture?

**Scenario / Problem Statement**
- A new team member is confused about how Jenkins executes jobs. You need to explain the master-agent architecture in simple terms during a knowledge-sharing session.

**What is being tested**
- Understanding of Jenkins master-agent architecture.
- Knowledge of distributed build concepts.

**Thinking & Problem-Solving Approach**
- Start with the core components: Jenkins Controller (Master) and Agents (Slaves).
- Explain the communication mechanism.
- Describe how jobs are distributed.

**Solution / Best Practice**
Jenkins follows a **Master-Agent architecture**:

1. **Jenkins Controller (Master)**:
   - Hosts the web UI and REST API
   - Manages job configurations and scheduling
   - Distributes builds to agents
   - Stores build history and artifacts

2. **Jenkins Agent (Slave)**:
   - Executes build tasks assigned by the master
   - Can run on different OS/environments
   - Connects via SSH, JNLP, or WebSocket

3. **Communication Flow**:
   - Master schedules a job → Assigns to available agent → Agent executes → Results sent back to master

**Example**
```
+------------------+          +------------------+
|  Jenkins Master  |  ------> |   Agent (Linux)  |
|  (Scheduler/UI)  |          |  (Build Executor)|
+------------------+          +------------------+
        |
        +----------------------> +------------------+
                                 |  Agent (Windows) |
                                 |  (Build Executor)|
                                 +------------------+
```

**Interview Tip**
- Interviewers expect you to mention that the master should NOT run heavy builds.
- Common mistake: Forgetting to mention that agents can be dynamically provisioned (e.g., Kubernetes pods).

---

<a id="q2"></a>
### Q2. A developer complains that their build is stuck in the queue for too long. What could be the reason and how would you troubleshoot?

**Scenario / Problem Statement**
- Multiple developers report that builds are waiting in the queue for 15+ minutes before starting, even though the Jenkins server seems idle.

**What is being tested**
- Understanding of Jenkins executors and job queuing.
- Troubleshooting skills.

**Thinking & Problem-Solving Approach**
- Check executor availability on master and agents.
- Verify if jobs are restricted to specific labels.
- Look for resource locks or blocked pipelines.

**Solution / Best Practice**

**Common reasons for queue delays:**

| Reason | Solution |
|--------|----------|
| No available executors | Increase executor count or add agents |
| Job restricted to unavailable label | Verify label assignment on agents |
| Resource lock held by another job | Check `Lockable Resources` plugin |
| Agent offline | Reconnect or provision new agent |
| Waiting for input approval | Check for `input` step blocking pipeline |

**Troubleshooting steps:**
1. Go to **Manage Jenkins → Nodes** → Check agent status
2. Click on **Build Queue** → See why job is waiting
3. Check job configuration for `label` restrictions
4. Review **System Load** statistics

**Example**
```groovy
// Job restricted to 'docker' label but no agent has this label
pipeline {
    agent { label 'docker' }  // Ensure an agent with this label exists
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}
```

**Interview Tip**
- Always mention checking the "Build Queue" tooltip — it shows exactly why a job is waiting.
- Common mistake: Not considering label mismatches.

---

<a id="q3"></a>
### Q3. You need to install Jenkins on a new Linux server. Walk me through the installation and initial setup process.

**Scenario / Problem Statement**
- Your company is setting up a new Jenkins instance on an Ubuntu 22.04 server. You are responsible for the installation and initial configuration.

**What is being tested**
- Practical knowledge of Jenkins installation.
- Understanding of prerequisites and initial security setup.

**Thinking & Problem-Solving Approach**
- Ensure Java is installed (Jenkins requires JDK 11 or 17).
- Add Jenkins repository and install.
- Complete the unlock wizard and install recommended plugins.

**Solution / Best Practice**

**Step-by-step installation on Ubuntu:**

```bash
# 1. Install Java
sudo apt update
sudo apt install openjdk-17-jdk -y

# 2. Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# 3. Install Jenkins
sudo apt update
sudo apt install jenkins -y

# 4. Start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# 5. Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Post-installation:**
1. Access Jenkins at `http://<server-ip>:8080`
2. Enter the initial admin password
3. Install suggested plugins
4. Create first admin user
5. Configure Jenkins URL

**Interview Tip**
- Mention that Jenkins home directory (`/var/lib/jenkins`) contains all configurations and should be backed up.
- Common mistake: Forgetting to open port 8080 in firewall.

---

<a id="q4"></a>
### Q4. What is the difference between a Freestyle job and a Pipeline job? When would you use each?

**Scenario / Problem Statement**
- A junior developer created a Freestyle job for a microservice. The project now requires conditional logic, parallel stages, and code review for CI changes. Should they continue with Freestyle or migrate to Pipeline?

**What is being tested**
- Understanding of Jenkins job types.
- Decision-making for CI/CD design.

**Thinking & Problem-Solving Approach**
- Compare flexibility, version control, and complexity.
- Consider long-term maintainability.
- Evaluate team's Groovy knowledge.

**Solution / Best Practice**

| Feature | Freestyle Job | Pipeline Job |
|---------|---------------|--------------|
| Configuration | UI-based (point & click) | Code-based (Jenkinsfile) |
| Version Control | Not easily versioned | Stored in SCM with code |
| Complex Logic | Limited | Full Groovy scripting |
| Parallel Execution | Manual setup | Native `parallel` block |
| Reusability | Limited | Shared Libraries |
| Visualization | Basic | Blue Ocean / Stage View |

**When to use Freestyle:**
- Simple, one-time jobs
- Quick prototyping
- Non-developers managing jobs

**When to use Pipeline:**
- Production CI/CD workflows
- Need for code review on CI changes
- Complex multi-stage builds
- Microservices with similar patterns

**Example**
```groovy
// Pipeline with conditional logic - not possible in Freestyle
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy-staging.sh'
            }
        }
    }
}
```

**Interview Tip**
- Always recommend Pipeline for production workloads.
- Common mistake: Saying Freestyle is "outdated" — it still has valid use cases.

---

<a id="q5"></a>
### Q5. Jenkins build failed with "java.lang.OutOfMemoryError: Java heap space". How do you fix this?

**Scenario / Problem Statement**
- A Java application build that was working yesterday suddenly fails with OutOfMemoryError. The build log shows the error occurs during the Maven compile phase.

**What is being tested**
- Understanding of JVM memory management in Jenkins.
- Ability to tune Jenkins and build tool configurations.

**Thinking & Problem-Solving Approach**
- Identify if the error is in Jenkins JVM or build tool JVM.
- Check if build size increased (new dependencies).
- Increase heap size appropriately.

**Solution / Best Practice**

**Where to increase memory depends on the error location:**

1. **Jenkins Master/Agent JVM:**
   - Edit `/etc/default/jenkins` (Debian) or `/etc/sysconfig/jenkins` (RHEL)
   ```bash
   JAVA_OPTS="-Xms1g -Xmx4g"
   ```

2. **Maven build:**
   - Set in Jenkinsfile or environment
   ```groovy
   environment {
       MAVEN_OPTS = '-Xmx2g -XX:MaxMetaspaceSize=512m'
   }
   ```

3. **Gradle build:**
   ```groovy
   environment {
       GRADLE_OPTS = '-Xmx2g -Dorg.gradle.daemon=false'
   }
   ```

**Diagnosis commands:**
```bash
# Check current Jenkins JVM settings
ps aux | grep jenkins

# Monitor memory during build
top -p $(pgrep -f jenkins)
```

**Interview Tip**
- Mention that increasing memory is a temporary fix — the root cause might be a memory leak or inefficient code.
- Common mistake: Blindly increasing heap without analyzing the actual memory consumer.

---

<a id="q6"></a>
### Q6. How do you configure Jenkins to automatically trigger a build when code is pushed to GitHub?

**Scenario / Problem Statement**
- Developers are manually clicking "Build Now" after every push. They want builds to trigger automatically when code is pushed to the `main` branch.

**What is being tested**
- Knowledge of Jenkins-GitHub integration.
- Understanding of webhooks vs polling.

**Thinking & Problem-Solving Approach**
- Webhooks are preferred (instant, efficient).
- Polling (SCM) is a fallback when webhooks aren't possible.
- Consider security for webhook endpoints.

**Solution / Best Practice**

**Method 1: GitHub Webhooks (Recommended)**

1. Install **GitHub Plugin** in Jenkins
2. In GitHub repository → Settings → Webhooks → Add webhook:
   ```
   Payload URL: http://<jenkins-url>/github-webhook/
   Content type: application/json
   Events: Just the push event
   ```
3. In Jenkins job → Build Triggers → Select **GitHub hook trigger for GITScm polling**

**Method 2: SCM Polling (Fallback)**
```groovy
pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')  // Poll every 5 minutes
    }
    stages {
        stage('Build') {
            steps {
                git 'https://github.com/org/repo.git'
                sh 'make build'
            }
        }
    }
}
```

**Webhook vs Polling:**

| Aspect | Webhook | Polling |
|--------|---------|---------|
| Trigger Speed | Instant | Delayed (poll interval) |
| Resource Usage | Efficient | Wasteful |
| Firewall | Jenkins must be reachable | Works behind firewall |

**Interview Tip**
- Mention that webhooks require Jenkins to be publicly accessible or use a reverse proxy.
- Common mistake: Forgetting to add the trailing slash in webhook URL.

---

<a id="q7"></a>
### Q7. A build works on one agent but fails on another with "command not found". How do you ensure consistent build environments?

**Scenario / Problem Statement**
- A Node.js build succeeds on Agent-A but fails on Agent-B with `npm: command not found`. Both agents are supposed to be identical.

**What is being tested**
- Understanding of agent configuration and tool management.
- Knowledge of environment consistency strategies.

**Thinking & Problem-Solving Approach**
- Check if required tools are installed on both agents.
- Use Jenkins tool installers or Docker for consistency.
- Consider agent labels and job restrictions.

**Solution / Best Practice**

**Approach 1: Use Jenkins Global Tool Configuration**

1. Manage Jenkins → Tools → NodeJS installations
2. Add NodeJS with auto-installer
3. In Jenkinsfile:
```groovy
pipeline {
    agent any
    tools {
        nodejs 'NodeJS-18'  // Name from Global Tool Configuration
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
    }
}
```

**Approach 2: Use Docker Agent (Best for consistency)**
```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
    }
}
```

**Approach 3: Use labels to target specific agents**
```groovy
agent { label 'nodejs-18' }
```

**Interview Tip**
- Docker agents provide the most consistent environments.
- Common mistake: Assuming all agents have identical tool installations.

---

<a id="q8"></a>
### Q8. How do you pass parameters to a Jenkins job and use them in the build?

**Scenario / Problem Statement**
- The deployment job needs to accept the environment name (dev/staging/prod) and version number as inputs so that the same job can deploy to different environments.

**What is being tested**
- Knowledge of parameterized builds.
- Ability to make reusable jobs.

**Thinking & Problem-Solving Approach**
- Define parameters in job configuration or Jenkinsfile.
- Access parameters using `params` object.
- Validate parameter values.

**Solution / Best Practice**

**Defining parameters in Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Target deployment environment'
        )
        string(
            name: 'VERSION',
            defaultValue: 'latest',
            description: 'Version to deploy'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run integration tests after deployment'
        )
    }
    
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying version ${params.VERSION} to ${params.ENVIRONMENT}"
                sh "./deploy.sh --env ${params.ENVIRONMENT} --version ${params.VERSION}"
            }
        }
        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                sh './run-integration-tests.sh'
            }
        }
    }
}
```

**Parameter types available:**
- `string` — Free text input
- `choice` — Dropdown selection
- `booleanParam` — Checkbox
- `password` — Masked input
- `file` — File upload

**Interview Tip**
- Mention that first build after adding parameters needs manual trigger to register them.
- Common mistake: Using `${ENVIRONMENT}` instead of `${params.ENVIRONMENT}`.

---

<a id="q9"></a>
### Q9. Jenkins build logs are growing very large and consuming disk space. How do you manage this?

**Scenario / Problem Statement**
- The Jenkins server disk is 90% full. Investigation shows that build logs and old artifacts are consuming most of the space.

**What is being tested**
- Knowledge of Jenkins housekeeping and maintenance.
- Understanding of build retention policies.

**Thinking & Problem-Solving Approach**
- Configure build discard policies.
- Archive only necessary artifacts.
- Set up automated cleanup.

**Solution / Best Practice**

**Method 1: Configure Build Discarder in Jenkinsfile**
```groovy
pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(
            numToKeepStr: '10',        // Keep last 10 builds
            daysToKeepStr: '30',       // Keep builds for 30 days
            artifactNumToKeepStr: '5', // Keep artifacts from last 5 builds
            artifactDaysToKeepStr: '15' // Keep artifacts for 15 days
        ))
        disableConcurrentBuilds()
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}
```

**Method 2: Global Cleanup using Disk Usage Plugin**
1. Install **Disk Usage Plugin**
2. Manage Jenkins → Configure System → Set workspace cleanup

**Method 3: Workspace Cleanup Plugin**
```groovy
post {
    always {
        cleanWs()  // Clean workspace after build
    }
}
```

**Method 4: CLI cleanup for immediate relief**
```bash
# Find large build directories
du -sh /var/lib/jenkins/jobs/*/builds/* | sort -hr | head -20

# Delete old builds (careful!)
find /var/lib/jenkins/jobs/*/builds -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
```

**Interview Tip**
- Mention that artifact storage should be external (Nexus, Artifactory, S3) for large files.
- Common mistake: Not configuring build retention from day one.

---

<a id="q10"></a>
### Q10. How do you backup and restore Jenkins configuration?

**Scenario / Problem Statement**
- The Jenkins server crashed and needs to be rebuilt. The team realizes there's no backup. How do you prevent this in the future and what should be backed up?

**What is being tested**
- Understanding of Jenkins data structure.
- Knowledge of backup strategies and disaster recovery.

**Thinking & Problem-Solving Approach**
- Identify critical Jenkins data locations.
- Choose between file backup and configuration-as-code.
- Automate backup process.

**Solution / Best Practice**

**Critical data to backup:**

| Path | Content |
|------|---------|
| `$JENKINS_HOME/config.xml` | Global configuration |
| `$JENKINS_HOME/jobs/` | Job configurations |
| `$JENKINS_HOME/users/` | User accounts |
| `$JENKINS_HOME/secrets/` | Encryption keys |
| `$JENKINS_HOME/plugins/` | Installed plugins |
| `$JENKINS_HOME/nodes/` | Agent configurations |

**Method 1: File System Backup**
```bash
#!/bin/bash
# backup-jenkins.sh
JENKINS_HOME="/var/lib/jenkins"
BACKUP_DIR="/backup/jenkins"
DATE=$(date +%Y%m%d)

# Stop Jenkins for consistent backup
systemctl stop jenkins

# Create backup
tar -czvf $BACKUP_DIR/jenkins-backup-$DATE.tar.gz \
    --exclude='$JENKINS_HOME/workspace' \
    --exclude='$JENKINS_HOME/caches' \
    $JENKINS_HOME

# Start Jenkins
systemctl start jenkins

# Keep only last 7 backups
find $BACKUP_DIR -name "jenkins-backup-*.tar.gz" -mtime +7 -delete
```

**Method 2: ThinBackup Plugin**
1. Install **ThinBackup** plugin
2. Manage Jenkins → ThinBackup → Settings
3. Configure backup directory and schedule

**Method 3: Configuration as Code (JCasC) - Best Practice**
```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Jenkins configured via JCasC"
  numExecutors: 2
  securityRealm:
    local:
      users:
        - id: admin
          password: ${ADMIN_PASSWORD}
```

**Restore process:**
1. Install fresh Jenkins
2. Stop Jenkins
3. Restore `$JENKINS_HOME` from backup
4. Start Jenkins

**Interview Tip**
- Recommend JCasC for reproducible Jenkins setups.
- Common mistake: Backing up workspace directories (usually not needed and very large).

---

## ✅ Section 1 Complete

**Questions covered:** Q1–Q10  
**Topics:** Jenkins architecture, executors, installation, job types, memory tuning, GitHub integration, agent consistency, parameters, log management, backup/restore.

---

## Section 2: Jenkins Jobs, Pipelines & Groovy (Q11–Q20)

---

<a id="q11"></a>
### Q11. What is the difference between Declarative and Scripted Pipeline? When would you use each?

**Scenario / Problem Statement**
- Your team is starting a new project and debating whether to use Declarative or Scripted Pipeline syntax. The senior developer prefers Scripted for flexibility, but you want standardization.

**What is being tested**
- Understanding of both pipeline syntaxes.
- Decision-making based on team needs.

**Thinking & Problem-Solving Approach**
- Compare syntax structure and learning curve.
- Consider team expertise and maintainability.
- Evaluate specific use cases requiring flexibility.

**Solution / Best Practice**

| Aspect | Declarative Pipeline | Scripted Pipeline |
|--------|---------------------|-------------------|
| Syntax | Structured, predefined blocks | Full Groovy flexibility |
| Learning Curve | Easier for beginners | Requires Groovy knowledge |
| Validation | Syntax validated before run | Runtime errors only |
| Structure | Requires `pipeline {}` block | Uses `node {}` block |
| Best For | Standard CI/CD workflows | Complex custom logic |

**Example - Declarative:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
```

**Example - Scripted:**
```groovy
node {
    try {
        stage('Build') {
            sh 'mvn clean package'
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        junit '**/target/surefire-reports/*.xml'
    }
}
```

**Interview Tip**
- Recommend Declarative for 90% of use cases due to better readability and built-in error handling.
- Common mistake: Using Scripted just because it feels "more powerful."

---

<a id="q12"></a>
### Q12. Your organization has 50+ microservices with similar CI/CD pipelines. How do you avoid code duplication?

**Scenario / Problem Statement**
- Each microservice has its own Jenkinsfile with nearly identical build, test, and deploy stages. Any change requires updating 50+ files.

**What is being tested**
- Knowledge of Jenkins Shared Libraries.
- Code reusability and DRY principles.

**Thinking & Problem-Solving Approach**
- Centralize common logic in a Shared Library.
- Version the library in Git.
- Allow teams to consume specific versions.

**Solution / Best Practice**

**Step 1: Create Shared Library structure**
```
jenkins-shared-library/
├── vars/
│   ├── standardPipeline.groovy    # Global variables
│   └── deployToK8s.groovy
├── src/
│   └── com/company/
│       └── BuildUtils.groovy      # Classes
└── resources/
    └── scripts/
        └── deploy.sh
```

**Step 2: Define reusable pipeline (vars/standardPipeline.groovy)**
```groovy
def call(Map config) {
    pipeline {
        agent { label config.agent ?: 'default' }
        stages {
            stage('Build') {
                steps {
                    sh "mvn clean package -DskipTests=${config.skipTests ?: false}"
                }
            }
            stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
            stage('Deploy') {
                when { branch 'main' }
                steps {
                    deployToK8s(config.appName, config.namespace)
                }
            }
        }
    }
}
```

**Step 3: Configure in Jenkins**
- Manage Jenkins → System → Global Pipeline Libraries
- Add library with Git repository URL

**Step 4: Use in microservice Jenkinsfile**
```groovy
@Library('jenkins-shared-library@v1.0') _

standardPipeline(
    appName: 'user-service',
    namespace: 'production',
    agent: 'docker'
)
```

**Interview Tip**
- Mention versioning (`@v1.0`) to prevent breaking changes across teams.
- Common mistake: Putting too much logic in shared libraries, making them hard to debug.

---

<a id="q13"></a>
### Q13. How do you set up a Multibranch Pipeline to automatically build feature branches?

**Scenario / Problem Statement**
- Developers create feature branches frequently. They want Jenkins to automatically discover and build these branches without manual job creation.

**What is being tested**
- Knowledge of Multibranch Pipeline.
- Understanding of branch discovery and filtering.

**Thinking & Problem-Solving Approach**
- Use Multibranch Pipeline job type.
- Configure branch discovery strategy.
- Filter branches using patterns.

**Solution / Best Practice**

**Step 1: Create Multibranch Pipeline job**
1. New Item → Multibranch Pipeline
2. Add Branch Source (GitHub/GitLab/Bitbucket)
3. Configure credentials

**Step 2: Configure branch discovery**
```
Branch Sources:
  - GitHub:
      Repository: org/repo
      Behaviors:
        - Discover branches: All branches
        - Discover pull requests: Merging to current branch
```

**Step 3: Filter branches (optional)**
```
Filter by name (with regular expression):
  Regular expression: (main|develop|feature/.*)
```

**Step 4: Jenkinsfile in repository**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Dev') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy-dev.sh'
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy-prod.sh'
            }
        }
    }
}
```

**Interview Tip**
- Mention orphaned item strategy to clean up deleted branches.
- Common mistake: Not configuring scan triggers for automatic discovery.

---

<a id="q14"></a>
### Q14. Your pipeline has 4 independent test suites. How do you run them in parallel to reduce build time?

**Scenario / Problem Statement**
- The CI pipeline takes 40 minutes because unit tests, integration tests, security scans, and linting run sequentially. Each takes ~10 minutes.

**What is being tested**
- Knowledge of parallel execution in Jenkins.
- Performance optimization skills.

**Thinking & Problem-Solving Approach**
- Identify independent stages that can run concurrently.
- Use `parallel` block in Declarative Pipeline.
- Consider resource constraints.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test -Dtest=*UnitTest'
                    }
                }
                stage('Integration Tests') {
                    agent { label 'docker' }
                    steps {
                        sh 'mvn verify -Dtest=*IntegrationTest'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'trivy fs --exit-code 1 .'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

**With `failFast` option:**
```groovy
stage('Parallel Tests') {
    failFast true  // Stop all parallel stages if one fails
    parallel {
        // ... stages
    }
}
```

**Interview Tip**
- Mention that parallel stages can run on different agents for better resource utilization.
- Common mistake: Running dependent tasks in parallel.

---

<a id="q15"></a>
### Q15. What are the best practices for writing maintainable Jenkinsfiles?

**Scenario / Problem Statement**
- A 500-line Jenkinsfile has become unmaintainable. It has hardcoded values, no error handling, and is difficult to debug.

**What is being tested**
- Code quality awareness.
- Pipeline engineering maturity.

**Thinking & Problem-Solving Approach**
- Apply software engineering principles to pipelines.
- Externalize configuration.
- Add proper error handling and logging.

**Solution / Best Practice**

**1. Use environment variables for configuration:**
```groovy
pipeline {
    agent any
    environment {
        APP_NAME = 'my-service'
        DOCKER_REGISTRY = 'registry.company.com'
        DEPLOY_ENV = "${env.BRANCH_NAME == 'main' ? 'prod' : 'dev'}"
    }
    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} ."
            }
        }
    }
}
```

**2. Add timeout and retry:**
```groovy
stage('Deploy') {
    options {
        timeout(time: 10, unit: 'MINUTES')
        retry(3)
    }
    steps {
        sh './deploy.sh'
    }
}
```

**3. Proper error handling with post blocks:**
```groovy
post {
    success {
        slackSend color: 'good', message: "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    failure {
        slackSend color: 'danger', message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    always {
        cleanWs()
        archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
    }
}
```

**4. Use `when` conditions:**
```groovy
stage('Deploy to Prod') {
    when {
        allOf {
            branch 'main'
            not { triggeredBy 'TimerTrigger' }
        }
    }
    steps {
        sh './deploy-prod.sh'
    }
}
```

**Interview Tip**
- Always mention timeout to prevent stuck builds.
- Common mistake: Hardcoding secrets or environment-specific values.

---

<a id="q16"></a>
### Q16. Your pipeline fails with "Scripts not permitted to use method". How do you resolve this?

**Scenario / Problem Statement**
- A Jenkinsfile using `new Date()` or `JsonSlurper` fails with a security exception about unapproved methods.

**What is being tested**
- Understanding of Groovy Sandbox.
- Security awareness in Jenkins.

**Thinking & Problem-Solving Approach**
- Understand why sandbox exists (security).
- Know the options: approve scripts or use approved methods.
- Balance security with functionality.

**Solution / Best Practice**

**Understanding the Sandbox:**
- Jenkins runs pipeline scripts in a restricted sandbox.
- Certain Groovy methods are blocked by default.
- This prevents malicious code execution.

**Option 1: Approve the method (Admin)**
1. Go to Manage Jenkins → In-process Script Approval
2. Review and approve pending scripts
3. ⚠️ Only approve trusted code!

**Option 2: Use pipeline-compatible alternatives**
```groovy
// Instead of new Date()
def now = new Date()  // ❌ Blocked

// Use pipeline step
def now = sh(script: 'date +%Y%m%d', returnStdout: true).trim()  // ✅ Works

// Or use currentBuild
echo "Build started at: ${currentBuild.startTimeInMillis}"
```

**Option 3: Move logic to Shared Library**
```groovy
// In shared library (runs outside sandbox)
// src/com/company/Utils.groovy
class Utils {
    static String getCurrentDate() {
        return new Date().format('yyyy-MM-dd')
    }
}
```

**Option 4: Use @NonCPS annotation**
```groovy
@NonCPS
def parseJson(String json) {
    def slurper = new groovy.json.JsonSlurper()
    return slurper.parseText(json)
}
```

**Interview Tip**
- Emphasize that script approval should be done carefully by reviewing code.
- Common mistake: Blindly approving all scripts without review.

---

<a id="q17"></a>
### Q17. How do you send notifications (Slack/Email) based on build status?

**Scenario / Problem Statement**
- The team wants to receive Slack notifications when builds fail and email notifications for successful production deployments.

**What is being tested**
- Knowledge of post-build actions.
- Integration with external services.

**Thinking & Problem-Solving Approach**
- Use `post` block for status-based actions.
- Configure notification plugins.
- Customize messages with build information.

**Solution / Best Practice**

**Install required plugins:**
- Slack Notification Plugin
- Email Extension Plugin

**Configure Slack in Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    
    post {
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: """✅ *Build Succeeded*
                    |*Job:* ${env.JOB_NAME}
                    |*Build:* #${env.BUILD_NUMBER}
                    |*Branch:* ${env.BRANCH_NAME}
                    |<${env.BUILD_URL}|View Build>""".stripMargin()
            )
        }
        failure {
            slackSend(
                channel: '#alerts',
                color: 'danger',
                message: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Build failed. Check console output at ${env.BUILD_URL}""",
                to: 'team@company.com',
                attachLog: true
            )
        }
        unstable {
            slackSend(
                channel: '#builds',
                color: 'warning',
                message: "⚠️ Build Unstable: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
```

**Interview Tip**
- Mention that credentials for Slack webhook should be stored securely.
- Common mistake: Sending too many notifications (notification fatigue).

---

<a id="q18"></a>
### Q18. How do you securely use credentials (passwords, API keys) in Jenkins pipelines?

**Scenario / Problem Statement**
- A developer hardcoded a database password in the Jenkinsfile. It was exposed in build logs and Git history. How do you prevent this?

**What is being tested**
- Knowledge of Jenkins credentials management.
- Security best practices.

**Thinking & Problem-Solving Approach**
- Never hardcode secrets.
- Use Jenkins Credentials store.
- Mask credentials in logs.

**Solution / Best Practice**

**Step 1: Add credentials in Jenkins**
1. Manage Jenkins → Credentials → System → Global credentials
2. Add credential (Username/Password, Secret text, SSH key, etc.)
3. Note the credential ID

**Step 2: Use in Jenkinsfile**

**Method 1: Using `credentials()` helper**
```groovy
pipeline {
    agent any
    environment {
        DB_CREDS = credentials('database-credentials')  // Creates DB_CREDS_USR and DB_CREDS_PSW
        API_KEY = credentials('api-secret-key')  // For secret text
    }
    stages {
        stage('Deploy') {
            steps {
                sh '''
                    export DB_USER=$DB_CREDS_USR
                    export DB_PASS=$DB_CREDS_PSW
                    ./deploy.sh
                '''
            }
        }
    }
}
```

**Method 2: Using `withCredentials` block**
```groovy
stage('Push to Registry') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'docker-hub',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )
        ]) {
            sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                docker push myimage:latest
            '''
        }
    }
}
```

**Method 3: SSH credentials**
```groovy
withCredentials([sshUserPrivateKey(
    credentialsId: 'ssh-deploy-key',
    keyFileVariable: 'SSH_KEY'
)]) {
    sh 'ssh -i $SSH_KEY user@server "deploy.sh"'
}
```

**Interview Tip**
- Credentials are automatically masked in build logs.
- Common mistake: Using `echo` to print credentials (still masked, but bad practice).

---

<a id="q19"></a>
### Q19. How do you trigger one Jenkins job from another (upstream/downstream)?

**Scenario / Problem Statement**
- After the build job succeeds, you need to automatically trigger deployment and integration test jobs in sequence.

**What is being tested**
- Knowledge of job orchestration.
- Understanding of build triggers.

**Thinking & Problem-Solving Approach**
- Use `build` step to trigger downstream jobs.
- Pass parameters between jobs.
- Consider parallel vs sequential execution.

**Solution / Best Practice**

**Method 1: Trigger downstream job and wait**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Trigger Deployment') {
            steps {
                build job: 'deploy-service',
                      parameters: [
                          string(name: 'VERSION', value: "${BUILD_NUMBER}"),
                          string(name: 'ENVIRONMENT', value: 'staging')
                      ],
                      wait: true  // Wait for completion
            }
        }
        stage('Trigger Integration Tests') {
            steps {
                build job: 'integration-tests',
                      parameters: [
                          string(name: 'TARGET_ENV', value: 'staging')
                      ],
                      wait: true,
                      propagate: true  // Fail if downstream fails
            }
        }
    }
}
```

**Method 2: Trigger without waiting**
```groovy
stage('Trigger Async Jobs') {
    steps {
        build job: 'notify-team', wait: false
        build job: 'update-dashboard', wait: false
    }
}
```

**Method 3: Parallel downstream jobs**
```groovy
stage('Trigger Tests') {
    parallel {
        stage('API Tests') {
            steps {
                build job: 'api-tests', wait: true
            }
        }
        stage('UI Tests') {
            steps {
                build job: 'ui-tests', wait: true
            }
        }
    }
}
```

**Method 4: Get result from downstream**
```groovy
script {
    def result = build job: 'downstream-job', wait: true
    echo "Downstream build number: ${result.number}"
    echo "Downstream result: ${result.result}"
}
```

**Interview Tip**
- Use `propagate: false` if you want to handle downstream failures gracefully.
- Common mistake: Creating circular dependencies between jobs.

---

<a id="q20"></a>
### Q20. How do you implement manual approval gates in a Jenkins pipeline?

**Scenario / Problem Statement**
- Before deploying to production, a manager must review and approve the deployment. The pipeline should pause and wait for approval.

**What is being tested**
- Knowledge of `input` step.
- Understanding of deployment approval workflows.

**Thinking & Problem-Solving Approach**
- Use `input` step for manual intervention.
- Configure who can approve.
- Set timeout to avoid indefinite waits.

**Solution / Best Practice**

**Basic approval gate:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh './deploy-staging.sh'
            }
        }
        stage('Approval') {
            steps {
                input message: 'Deploy to production?',
                      ok: 'Deploy',
                      submitter: 'admin,release-managers'
            }
        }
        stage('Deploy to Production') {
            steps {
                sh './deploy-prod.sh'
            }
        }
    }
}
```

**With timeout and parameters:**
```groovy
stage('Production Approval') {
    options {
        timeout(time: 24, unit: 'HOURS')
    }
    steps {
        script {
            def approval = input(
                message: 'Deploy to production?',
                ok: 'Approve',
                submitter: 'devops-leads',
                parameters: [
                    choice(name: 'DEPLOY_REGION', 
                           choices: ['us-east', 'us-west', 'eu-west'],
                           description: 'Select deployment region'),
                    booleanParam(name: 'RUN_SMOKE_TESTS',
                                 defaultValue: true,
                                 description: 'Run smoke tests after deploy')
                ]
            )
            env.DEPLOY_REGION = approval.DEPLOY_REGION
            env.RUN_SMOKE_TESTS = approval.RUN_SMOKE_TESTS
        }
    }
}
```

**Non-blocking approval (agent released):**
```groovy
stage('Approval') {
    agent none  // Don't hold an executor
    steps {
        input message: 'Proceed to production?'
    }
}
```

**Interview Tip**
- Use `agent none` during input to free up executors.
- Common mistake: Not setting timeout, causing jobs to wait forever.

---

## ✅ Section 2 Complete

**Questions covered:** Q11–Q20  
**Topics:** Declarative vs Scripted, Shared Libraries, Multibranch, Parallel execution, Jenkinsfile best practices, Groovy sandbox, Notifications, Credentials, Job orchestration, Approval gates.

---

## Section 3: CI/CD Design & Real DevOps Scenarios (Q21–Q30)

---

<a id="q21"></a>
### Q21. Design a complete CI/CD pipeline for a Java microservice from code commit to production.

**Scenario / Problem Statement**
- You're architecting a CI/CD pipeline for a new Java microservice. Walk through the complete flow including all stages, quality gates, and deployment strategy.

**What is being tested**
- End-to-end CI/CD design skills.
- Understanding of DevOps best practices.

**Thinking & Problem-Solving Approach**
- Cover all phases: Build, Test, Security, Artifact, Deploy.
- Include quality gates at each phase.
- Consider rollback strategy.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'user-service'
        DOCKER_REGISTRY = 'registry.company.com'
        SONAR_HOST = 'https://sonar.company.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco execPattern: '**/target/jacoco.exec'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Security Scan') {
            parallel {
                stage('SAST') {
                    steps {
                        sh 'mvn dependency-check:check'
                    }
                }
                stage('Container Scan') {
                    steps {
                        sh "trivy image ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Build & Push Image') {
            steps {
                sh """
                    mvn package -DskipTests
                    docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} .
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                """
            }
        }
        
        stage('Deploy to Dev') {
            when { branch 'develop' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} -n dev"
            }
        }
        
        stage('Deploy to Staging') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} -n staging"
            }
        }
        
        stage('Integration Tests') {
            when { branch 'main' }
            steps {
                sh 'mvn verify -Pintegration-tests'
            }
        }
        
        stage('Production Approval') {
            when { branch 'main' }
            options { timeout(time: 24, unit: 'HOURS') }
            steps {
                input message: 'Deploy to Production?', submitter: 'release-team'
            }
        }
        
        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} -n production"
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good', message: "✅ ${APP_NAME} deployed successfully"
        }
        failure {
            slackSend color: 'danger', message: "❌ ${APP_NAME} pipeline failed"
        }
    }
}
```

**Interview Tip**
- Always mention quality gates and security scans.
- Common mistake: Skipping tests in CI to save time.

---

<a id="q22"></a>
### Q22. How do you implement Blue-Green deployment using Jenkins?

**Scenario / Problem Statement**
- Your application requires zero-downtime deployments. If the new version has issues, you need instant rollback capability.

**What is being tested**
- Knowledge of deployment strategies.
- Understanding of traffic switching.

**Thinking & Problem-Solving Approach**
- Maintain two identical environments (Blue/Green).
- Deploy to inactive environment.
- Switch traffic after validation.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'payment-service'
        NAMESPACE = 'production'
    }
    
    stages {
        stage('Determine Target') {
            steps {
                script {
                    // Check which environment is currently live
                    def currentLive = sh(
                        script: "kubectl get service ${APP_NAME} -n ${NAMESPACE} -o jsonpath='{.spec.selector.version}'",
                        returnStdout: true
                    ).trim()
                    
                    env.CURRENT_ENV = currentLive  // blue or green
                    env.TARGET_ENV = (currentLive == 'blue') ? 'green' : 'blue'
                    echo "Current: ${CURRENT_ENV}, Deploying to: ${TARGET_ENV}"
                }
            }
        }
        
        stage('Deploy to Target') {
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME}-${TARGET_ENV} \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} \
                        -n ${NAMESPACE}
                    kubectl rollout status deployment/${APP_NAME}-${TARGET_ENV} -n ${NAMESPACE}
                """
            }
        }
        
        stage('Smoke Tests') {
            steps {
                sh """
                    # Test against target environment directly
                    TARGET_URL=\$(kubectl get svc ${APP_NAME}-${TARGET_ENV} -n ${NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                    curl -f http://\${TARGET_URL}/health || exit 1
                    ./run-smoke-tests.sh \${TARGET_URL}
                """
            }
        }
        
        stage('Switch Traffic') {
            steps {
                input message: "Switch traffic to ${TARGET_ENV}?"
                sh """
                    kubectl patch service ${APP_NAME} -n ${NAMESPACE} \
                        -p '{"spec":{"selector":{"version":"${TARGET_ENV}"}}}'
                """
                echo "Traffic now routing to ${TARGET_ENV}"
            }
        }
        
        stage('Verify Production') {
            steps {
                sh './verify-production.sh'
            }
        }
    }
    
    post {
        failure {
            script {
                echo "Deployment failed. ${CURRENT_ENV} is still serving traffic."
            }
        }
    }
}
```

**Rollback script:**
```groovy
// Instant rollback - just switch the service selector back
stage('Rollback') {
    steps {
        sh """
            kubectl patch service ${APP_NAME} -n ${NAMESPACE} \
                -p '{"spec":{"selector":{"version":"${CURRENT_ENV}"}}}'
        """
    }
}
```

**Interview Tip**
- Mention that Blue-Green requires double infrastructure.
- Common mistake: Not testing the inactive environment before switching.

---

<a id="q23"></a>
### Q23. How do you implement Canary releases with Jenkins?

**Scenario / Problem Statement**
- You want to release a new version to 10% of users first, monitor for errors, then gradually increase to 100%.

**What is being tested**
- Knowledge of progressive delivery.
- Understanding of traffic splitting.

**Thinking & Problem-Solving Approach**
- Deploy canary alongside stable version.
- Use weighted traffic routing.
- Monitor metrics and decide to promote or rollback.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'CANARY_WEIGHT', defaultValue: '10', description: 'Initial canary traffic %')
    }
    
    stages {
        stage('Deploy Canary') {
            steps {
                sh """
                    # Deploy canary version
                    kubectl apply -f k8s/deployment-canary.yaml
                    kubectl set image deployment/${APP_NAME}-canary \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                """
            }
        }
        
        stage('Configure Traffic Split') {
            steps {
                sh """
                    # Using Istio VirtualService
                    cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ${APP_NAME}
spec:
  hosts:
  - ${APP_NAME}
  http:
  - route:
    - destination:
        host: ${APP_NAME}-stable
      weight: ${100 - params.CANARY_WEIGHT.toInteger()}
    - destination:
        host: ${APP_NAME}-canary
      weight: ${params.CANARY_WEIGHT}
EOF
                """
            }
        }
        
        stage('Monitor Canary') {
            steps {
                script {
                    // Monitor for 10 minutes
                    def success = true
                    for (int i = 0; i < 10; i++) {
                        sleep(60)
                        
                        // Check error rate from Prometheus
                        def errorRate = sh(
                            script: """
                                curl -s 'http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~"5..",app="${APP_NAME}-canary"}[5m])'
                            """,
                            returnStdout: true
                        )
                        
                        if (errorRate.toFloat() > 0.01) {  // >1% error rate
                            success = false
                            break
                        }
                        echo "Minute ${i+1}: Canary healthy"
                    }
                    
                    env.CANARY_SUCCESS = success.toString()
                }
            }
        }
        
        stage('Promote or Rollback') {
            steps {
                script {
                    if (env.CANARY_SUCCESS == 'true') {
                        input message: 'Promote canary to 100%?'
                        
                        // Update stable deployment
                        sh """
                            kubectl set image deployment/${APP_NAME}-stable \
                                ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                            kubectl delete deployment ${APP_NAME}-canary
                        """
                    } else {
                        echo "Canary failed! Rolling back..."
                        sh "kubectl delete deployment ${APP_NAME}-canary"
                        error("Canary deployment failed health checks")
                    }
                }
            }
        }
    }
}
```

**Interview Tip**
- Mention the need for proper observability (metrics, logs, traces).
- Common mistake: Not having automated rollback criteria.

---

<a id="q24"></a>
### Q24. How do you handle database migrations in a CI/CD pipeline?

**Scenario / Problem Statement**
- Your application requires database schema changes. How do you safely run migrations without downtime or data loss?

**What is being tested**
- Knowledge of database migration strategies.
- Understanding of deployment ordering.

**Thinking & Problem-Solving Approach**
- Run migrations before deploying application code.
- Ensure backward compatibility.
- Have rollback scripts ready.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    stages {
        stage('Validate Migrations') {
            steps {
                sh """
                    # Validate migration scripts syntax
                    flyway validate -url=\${DB_URL} -user=\${DB_USER} -password=\${DB_PASS}
                """
            }
        }
        
        stage('Backup Database') {
            steps {
                sh """
                    # Create backup before migration
                    pg_dump -h \${DB_HOST} -U \${DB_USER} -d \${DB_NAME} > backup-\${BUILD_NUMBER}.sql
                    aws s3 cp backup-\${BUILD_NUMBER}.sql s3://db-backups/
                """
            }
        }
        
        stage('Run Migrations') {
            steps {
                sh """
                    # Apply migrations
                    flyway migrate -url=\${DB_URL} -user=\${DB_USER} -password=\${DB_PASS}
                """
            }
        }
        
        stage('Verify Schema') {
            steps {
                sh """
                    # Verify migration success
                    flyway info -url=\${DB_URL}
                """
            }
        }
        
        stage('Deploy Application') {
            steps {
                sh 'kubectl rollout restart deployment/${APP_NAME}'
            }
        }
    }
    
    post {
        failure {
            script {
                echo "Rolling back database..."
                sh """
                    # Rollback to previous version
                    flyway undo -url=\${DB_URL} -user=\${DB_USER} -password=\${DB_PASS}
                    # Or restore from backup
                    # psql -h \${DB_HOST} -U \${DB_USER} -d \${DB_NAME} < backup-\${BUILD_NUMBER}.sql
                """
            }
        }
    }
}
```

**Best practices for migrations:**

| Practice | Description |
|----------|-------------|
| Backward compatible | New code should work with old schema |
| Forward compatible | Old code should work with new schema |
| Small migrations | Avoid large schema changes |
| Test migrations | Run against copy of production data |
| Version control | Store migrations with application code |

**Interview Tip**
- Mention expand-contract pattern for zero-downtime migrations.
- Common mistake: Not testing migrations with production-like data.

---

<a id="q25"></a>
### Q25. How do you build and push Docker images in Jenkins?

**Scenario / Problem Statement**
- Your pipeline needs to build a Docker image, tag it properly, scan for vulnerabilities, and push to a private registry.

**What is being tested**
- Docker integration with Jenkins.
- Image tagging and security scanning.

**Thinking & Problem-Solving Approach**
- Use Docker Pipeline plugin or shell commands.
- Implement proper tagging strategy.
- Scan before pushing.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.company.com'
        IMAGE_NAME = "${DOCKER_REGISTRY}/myapp"
        DOCKER_CREDENTIALS = credentials('docker-registry-creds')
    }
    
    stages {
        stage('Build Image') {
            steps {
                script {
                    // Build with multiple tags
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}", "--build-arg VERSION=${BUILD_NUMBER} .")
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh """
                    # Scan with Trivy
                    trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-creds') {
                        def image = docker.image("${IMAGE_NAME}:${BUILD_NUMBER}")
                        
                        // Push with build number
                        image.push()
                        
                        // Push with branch-based tag
                        image.push("${env.BRANCH_NAME}-${BUILD_NUMBER}")
                        
                        // Push latest for main branch
                        if (env.BRANCH_NAME == 'main') {
                            image.push('latest')
                        }
                    }
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
            }
        }
    }
}
```

**Alternative using Kaniko (no Docker daemon needed):**
```groovy
stage('Build with Kaniko') {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:latest
                    command: ["/busybox/cat"]
                    tty: true
            '''
        }
    }
    steps {
        container('kaniko') {
            sh """
                /kaniko/executor \
                    --dockerfile=Dockerfile \
                    --destination=${IMAGE_NAME}:${BUILD_NUMBER}
            """
        }
    }
}
```

**Interview Tip**
- Mention using Kaniko for Kubernetes environments (no Docker socket needed).
- Common mistake: Not cleaning up local images after push.

---

<a id="q26"></a>
### Q26. How do you deploy to Kubernetes from Jenkins?

**Scenario / Problem Statement**
- Your Jenkins pipeline needs to deploy applications to multiple Kubernetes clusters (dev, staging, prod).

**What is being tested**
- Kubernetes deployment strategies.
- Multi-cluster management.

**Thinking & Problem-Solving Approach**
- Use kubectl or Helm.
- Manage kubeconfig securely.
- Implement deployment verification.

**Solution / Best Practice**

**Method 1: Using kubectl**
```groovy
pipeline {
    agent any
    
    environment {
        KUBECONFIG = credentials('kubeconfig-prod')
    }
    
    stages {
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    # Update deployment
                    kubectl set image deployment/myapp \
                        myapp=${DOCKER_REGISTRY}/myapp:${BUILD_NUMBER} \
                        -n production
                    
                    # Wait for rollout
                    kubectl rollout status deployment/myapp -n production --timeout=300s
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh """
                    # Check pod status
                    kubectl get pods -n production -l app=myapp
                    
                    # Run health check
                    kubectl exec -n production deploy/myapp -- curl -f localhost:8080/health
                """
            }
        }
    }
}
```

**Method 2: Using Helm**
```groovy
stage('Deploy with Helm') {
    steps {
        sh """
            helm upgrade --install myapp ./helm-chart \
                --namespace production \
                --set image.tag=${BUILD_NUMBER} \
                --set replicas=3 \
                --wait --timeout=5m
        """
    }
}
```

**Method 3: Using Kubernetes Plugin**
```groovy
stage('Deploy') {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  serviceAccountName: jenkins-deployer
                  containers:
                  - name: kubectl
                    image: bitnami/kubectl
                    command: ["cat"]
                    tty: true
            '''
        }
    }
    steps {
        container('kubectl') {
            sh 'kubectl apply -f k8s/'
        }
    }
}
```

**Interview Tip**
- Mention using service accounts with RBAC for least privilege.
- Common mistake: Storing kubeconfig as plain text.

---

<a id="q27"></a>
### Q27. How do you integrate Jenkins with artifact repositories like Nexus or Artifactory?

**Scenario / Problem Statement**
- Your organization wants to store build artifacts centrally, manage dependencies, and ensure artifact traceability.

**What is being tested**
- Knowledge of artifact management.
- Integration patterns.

**Thinking & Problem-Solving Approach**
- Configure artifact repository credentials.
- Publish artifacts after successful builds.
- Download artifacts for deployments.

**Solution / Best Practice**

**Publishing to Nexus (Maven):**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        mvn deploy \
                            -DaltDeploymentRepository=nexus::default::https://nexus.company.com/repository/maven-releases/ \
                            -Dusername=$NEXUS_USER \
                            -Dpassword=$NEXUS_PASS
                    '''
                }
            }
        }
    }
}
```

**Using Nexus Artifact Uploader Plugin:**
```groovy
stage('Upload Artifact') {
    steps {
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'https',
            nexusUrl: 'nexus.company.com',
            groupId: 'com.company',
            version: "${BUILD_NUMBER}",
            repository: 'maven-releases',
            credentialsId: 'nexus-creds',
            artifacts: [
                [artifactId: 'myapp',
                 classifier: '',
                 file: 'target/myapp.jar',
                 type: 'jar']
            ]
        )
    }
}
```

**Downloading artifacts:**
```groovy
stage('Download Artifact') {
    steps {
        sh """
            curl -u \${NEXUS_USER}:\${NEXUS_PASS} \
                -O https://nexus.company.com/repository/maven-releases/com/company/myapp/${VERSION}/myapp-${VERSION}.jar
        """
    }
}
```

**Interview Tip**
- Mention snapshot vs release repositories.
- Common mistake: Not cleaning up old snapshots.

---

<a id="q28"></a>
### Q28. How do you implement environment promotion (Dev → Staging → Prod)?

**Scenario / Problem Statement**
- The same artifact must be promoted through environments with proper testing at each stage. How do you design this workflow?

**What is being tested**
- Understanding of promotion patterns.
- Environment management strategies.

**Thinking & Problem-Solving Approach**
- Build once, deploy many times.
- Use artifact versioning.
- Gate promotions with approvals and tests.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'ARTIFACT_VERSION', description: 'Version to promote')
    }
    
    stages {
        stage('Deploy to Dev') {
            steps {
                deployToEnvironment('dev', params.ARTIFACT_VERSION)
            }
        }
        
        stage('Test in Dev') {
            steps {
                sh './run-integration-tests.sh dev'
            }
        }
        
        stage('Promote to Staging') {
            steps {
                input message: 'Promote to Staging?', submitter: 'developers'
                deployToEnvironment('staging', params.ARTIFACT_VERSION)
            }
        }
        
        stage('Test in Staging') {
            steps {
                sh './run-integration-tests.sh staging'
                sh './run-performance-tests.sh staging'
            }
        }
        
        stage('Promote to Production') {
            steps {
                input message: 'Promote to Production?', submitter: 'release-managers'
                deployToEnvironment('production', params.ARTIFACT_VERSION)
            }
        }
        
        stage('Verify Production') {
            steps {
                sh './run-smoke-tests.sh production'
            }
        }
    }
}

def deployToEnvironment(String env, String version) {
    sh """
        helm upgrade --install myapp ./chart \
            --namespace ${env} \
            --values values-${env}.yaml \
            --set image.tag=${version}
    """
}
```

**Alternative: Separate pipelines per environment**
```
Build Pipeline → Dev Pipeline → Staging Pipeline → Prod Pipeline
      ↓              ↓               ↓                 ↓
  Artifact      Auto-deploy     Manual trigger    Manual trigger
```

**Interview Tip**
- Emphasize "build once, deploy many" to ensure artifact consistency.
- Common mistake: Rebuilding for each environment.

---

<a id="q29"></a>
### Q29. How do you implement rollback strategies in Jenkins?

**Scenario / Problem Statement**
- A production deployment caused issues. How do you quickly rollback to the previous working version?

**What is being tested**
- Understanding of rollback mechanisms.
- Incident response readiness.

**Thinking & Problem-Solving Approach**
- Track deployed versions.
- Implement automated rollback triggers.
- Test rollback procedures regularly.

**Solution / Best Practice**

**Kubernetes rollback:**
```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ACTION', choices: ['deploy', 'rollback'], description: 'Action to perform')
        string(name: 'ROLLBACK_TO', defaultValue: '', description: 'Version to rollback to (for rollback)')
    }
    
    stages {
        stage('Deploy') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                sh """
                    # Record current version before deploy
                    kubectl get deployment myapp -o jsonpath='{.spec.template.spec.containers[0].image}' > previous-version.txt
                    
                    # Deploy new version
                    kubectl set image deployment/myapp myapp=${IMAGE}:${BUILD_NUMBER}
                    kubectl rollout status deployment/myapp --timeout=300s
                """
            }
        }
        
        stage('Rollback') {
            when {
                expression { params.ACTION == 'rollback' }
            }
            steps {
                script {
                    if (params.ROLLBACK_TO) {
                        // Rollback to specific version
                        sh "kubectl set image deployment/myapp myapp=${IMAGE}:${params.ROLLBACK_TO}"
                    } else {
                        // Rollback to previous revision
                        sh "kubectl rollout undo deployment/myapp"
                    }
                }
                sh "kubectl rollout status deployment/myapp"
            }
        }
    }
}
```

**Automated rollback on failure:**
```groovy
stage('Deploy with Auto-Rollback') {
    steps {
        script {
            try {
                sh """
                    kubectl set image deployment/myapp myapp=${IMAGE}:${BUILD_NUMBER}
                    kubectl rollout status deployment/myapp --timeout=300s
                """
                
                // Health check
                sh 'curl -f http://myapp.prod.svc/health'
                
            } catch (Exception e) {
                echo "Deployment failed. Rolling back..."
                sh "kubectl rollout undo deployment/myapp"
                sh "kubectl rollout status deployment/myapp"
                error("Deployment failed and was rolled back: ${e.message}")
            }
        }
    }
}
```

**Interview Tip**
- Mention keeping deployment history (`revisionHistoryLimit`).
- Common mistake: Not testing rollback procedures.

---

<a id="q30"></a>
### Q30. How do you handle CI/CD for a monorepo with multiple services?

**Scenario / Problem Statement**
- Your organization has a monorepo containing 10 microservices. You want to only build/deploy services that have changed.

**What is being tested**
- Understanding of monorepo CI challenges.
- Efficient build strategies.

**Thinking & Problem-Solving Approach**
- Detect changed paths.
- Build only affected services.
- Handle shared dependencies.

**Solution / Best Practice**

```groovy
pipeline {
    agent any
    
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Get changed files
                    def changes = sh(
                        script: "git diff --name-only HEAD~1",
                        returnStdout: true
                    ).trim().split('\n')
                    
                    // Map paths to services
                    env.BUILD_USER_SERVICE = changes.any { it.startsWith('services/user-service/') }
                    env.BUILD_ORDER_SERVICE = changes.any { it.startsWith('services/order-service/') }
                    env.BUILD_PAYMENT_SERVICE = changes.any { it.startsWith('services/payment-service/') }
                    env.BUILD_SHARED_LIBS = changes.any { it.startsWith('shared/') }
                    
                    // If shared libs changed, rebuild all
                    if (env.BUILD_SHARED_LIBS == 'true') {
                        env.BUILD_USER_SERVICE = 'true'
                        env.BUILD_ORDER_SERVICE = 'true'
                        env.BUILD_PAYMENT_SERVICE = 'true'
                    }
                    
                    echo "Services to build: user=${BUILD_USER_SERVICE}, order=${BUILD_ORDER_SERVICE}, payment=${BUILD_PAYMENT_SERVICE}"
                }
            }
        }
        
        stage('Build Services') {
            parallel {
                stage('User Service') {
                    when {
                        expression { env.BUILD_USER_SERVICE == 'true' }
                    }
                    steps {
                        dir('services/user-service') {
                            sh 'mvn clean package'
                            sh "docker build -t user-service:${BUILD_NUMBER} ."
                        }
                    }
                }
                stage('Order Service') {
                    when {
                        expression { env.BUILD_ORDER_SERVICE == 'true' }
                    }
                    steps {
                        dir('services/order-service') {
                            sh 'mvn clean package'
                            sh "docker build -t order-service:${BUILD_NUMBER} ."
                        }
                    }
                }
                stage('Payment Service') {
                    when {
                        expression { env.BUILD_PAYMENT_SERVICE == 'true' }
                    }
                    steps {
                        dir('services/payment-service') {
                            sh 'mvn clean package'
                            sh "docker build -t payment-service:${BUILD_NUMBER} ."
                        }
                    }
                }
            }
        }
    }
}
```

**Using changeset condition:**
```groovy
stage('Build User Service') {
    when {
        changeset "services/user-service/**"
    }
    steps {
        dir('services/user-service') {
            sh 'make build'
        }
    }
}
```

**Interview Tip**
- Mention handling transitive dependencies between services.
- Common mistake: Not rebuilding dependent services when shared code changes.

---

## ✅ Section 3 Complete

**Questions covered:** Q21–Q30  
**Topics:** Complete CI/CD design, Blue-Green deployment, Canary releases, Database migrations, Docker builds, Kubernetes deployment, Artifact management, Environment promotion, Rollback strategies, Monorepo CI/CD.

---

## Section 4: Jenkins Administration, Security & Scaling (Q31–Q40)

---

<a id="q31"></a>
### Q31. How do you implement Role-Based Access Control (RBAC) in Jenkins?

**Scenario / Problem Statement**
- Your Jenkins has 100+ users across multiple teams. You need to ensure developers can only access their team's jobs, while admins have full access.

**What is being tested**
- Knowledge of Jenkins security plugins.
- Access control design.

**Thinking & Problem-Solving Approach**
- Use Role-Based Authorization Strategy plugin.
- Define global and project-level roles.
- Organize jobs into folders per team.

**Solution / Best Practice**

**Step 1: Install and configure plugin**
1. Install "Role-Based Authorization Strategy" plugin
2. Manage Jenkins → Security → Authorization → Role-Based Strategy

**Step 2: Define Global Roles**

| Role | Permissions |
|------|-------------|
| admin | Full control |
| developer | Read, Build, Cancel |
| viewer | Read only |

**Step 3: Define Project Roles (pattern-based)**

| Role | Pattern | Permissions |
|------|---------|-------------|
| team-a-dev | `team-a/.*` | Build, Read, Workspace |
| team-b-dev | `team-b/.*` | Build, Read, Workspace |
| devops | `.*` | Build, Read, Configure |

**Step 4: Assign roles to users/groups**
```
Users/Groups → Roles:
- john@company.com → team-a-dev
- DevOps-Team (AD Group) → devops
- jane@company.com → admin
```

**Folder-based organization:**
```
Jenkins/
├── team-a/
│   ├── service-1/
│   └── service-2/
├── team-b/
│   ├── service-3/
│   └── service-4/
└── shared/
    └── infrastructure/
```

**Interview Tip**
- Mention integrating with LDAP/AD groups for easier management.
- Common mistake: Giving too many permissions to avoid complaints.

---

<a id="q32"></a>
### Q32. How do you integrate Jenkins with LDAP/Active Directory?

**Scenario / Problem Statement**
- Your organization uses Active Directory for identity management. Users should authenticate with their corporate credentials.

**What is being tested**
- Enterprise authentication integration.
- Security configuration.

**Thinking & Problem-Solving Approach**
- Configure LDAP security realm.
- Map AD groups to Jenkins roles.
- Test with a service account.

**Solution / Best Practice**

**Configuration via UI:**
1. Manage Jenkins → Security
2. Security Realm → LDAP
3. Configure settings:

```
Server: ldap://ad.company.com:389
Root DN: DC=company,DC=com
User search base: OU=Users
User search filter: sAMAccountName={0}
Group search base: OU=Groups
Group search filter: (&(objectClass=group)(member={0}))
Manager DN: CN=jenkins-svc,OU=ServiceAccounts,DC=company,DC=com
Manager Password: ********
```

**Configuration as Code (JCasC):**
```yaml
jenkins:
  securityRealm:
    ldap:
      configurations:
        - server: "ldap://ad.company.com:389"
          rootDN: "DC=company,DC=com"
          userSearchBase: "OU=Users"
          userSearch: "sAMAccountName={0}"
          groupSearchBase: "OU=Groups"
          groupSearchFilter: "(&(objectClass=group)(member={0}))"
          managerDN: "CN=jenkins-svc,OU=ServiceAccounts,DC=company,DC=com"
          managerPasswordSecret: "${LDAP_MANAGER_PASSWORD}"
      cache:
        size: 100
        ttl: 300
```

**Testing LDAP connection:**
```bash
# Test from Jenkins server
ldapsearch -x -H ldap://ad.company.com:389 \
    -D "CN=jenkins-svc,OU=ServiceAccounts,DC=company,DC=com" \
    -w 'password' \
    -b "DC=company,DC=com" \
    "sAMAccountName=testuser"
```

**Interview Tip**
- Always test with a limited service account first.
- Common mistake: Not handling SSL certificates for LDAPS.

---

<a id="q33"></a>
### Q33. How do you secure Jenkins credentials and prevent secret leakage?

**Scenario / Problem Statement**
- A security audit found credentials exposed in build logs and Jenkinsfiles stored in Git. How do you remediate and prevent this?

**What is being tested**
- Security best practices.
- Secret management.

**Thinking & Problem-Solving Approach**
- Use Jenkins credentials store.
- Integrate with external secret managers.
- Enable audit logging.

**Solution / Best Practice**

**1. Use credentials store properly:**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'prod-db', 
                                     usernameVariable: 'DB_USER',
                                     passwordVariable: 'DB_PASS'),
                    string(credentialsId: 'api-key', variable: 'API_KEY')
                ]) {
                    // Credentials masked in logs automatically
                    sh './deploy.sh'
                }
            }
        }
    }
}
```

**2. Integrate with HashiCorp Vault:**
```groovy
stage('Get Secrets') {
    steps {
        withVault(
            configuration: [
                vaultUrl: 'https://vault.company.com',
                vaultCredentialId: 'vault-approle'
            ],
            vaultSecrets: [
                [
                    path: 'secret/data/prod/db',
                    secretValues: [
                        [envVar: 'DB_PASSWORD', vaultKey: 'password']
                    ]
                ]
            ]
        ) {
            sh 'echo "Using password from Vault"'
        }
    }
}
```

**3. Mask sensitive output:**
```groovy
steps {
    wrap([$class: 'MaskPasswordsBuildWrapper', 
          varPasswordPairs: [[password: 'secret123', var: 'SECRET']]]) {
        sh 'echo $SECRET'  // Will be masked as ****
    }
}
```

**4. Enable credential encryption:**
- Ensure `$JENKINS_HOME/secrets/master.key` is backed up securely
- Credentials are encrypted at rest

**Interview Tip**
- Mention rotating credentials regularly.
- Common mistake: Printing credentials with `echo` for debugging.

---

<a id="q34"></a>
### Q34. How do you manage Jenkins plugins effectively?

**Scenario / Problem Statement**
- Jenkins has 150+ plugins installed. Some have security vulnerabilities, some are outdated, and some conflict with each other.

**What is being tested**
- Plugin lifecycle management.
- Maintenance best practices.

**Thinking & Problem-Solving Approach**
- Audit installed plugins.
- Remove unnecessary plugins.
- Test updates in staging environment.

**Solution / Best Practice**

**1. List installed plugins:**
```groovy
// Script Console (Manage Jenkins → Script Console)
Jenkins.instance.pluginManager.plugins.each {
    println "${it.shortName}: ${it.version}"
}
```

**2. Export plugin list:**
```bash
# Via CLI
java -jar jenkins-cli.jar -s http://jenkins:8080/ list-plugins > plugins.txt

# Or via API
curl -s http://jenkins:8080/pluginManager/api/json?depth=1 | \
    jq -r '.plugins[] | "\(.shortName):\(.version)"' > plugins.txt
```

**3. Install plugins from file:**
```bash
# plugins.txt format:
# git:4.11.0
# workflow-aggregator:2.6

java -jar jenkins-cli.jar -s http://jenkins:8080/ install-plugin $(cat plugins.txt | tr '\n' ' ')
```

**4. Configuration as Code for plugins:**
```yaml
# plugins.yaml
plugins:
  - artifactId: git
    source:
      version: "4.11.0"
  - artifactId: workflow-aggregator
    source:
      version: "2.6"
```

**5. Security scanning:**
```bash
# Check for vulnerable plugins
curl -s http://jenkins:8080/manage/pluginManager/api/json?depth=1 | \
    jq '.plugins[] | select(.hasUpdate==true) | {name:.shortName, current:.version}'
```

**Plugin management best practices:**

| Practice | Description |
|----------|-------------|
| Staging first | Test plugin updates in non-prod |
| Minimal plugins | Only install what's needed |
| Regular audits | Monthly plugin review |
| Pin versions | Use specific versions in automation |
| Monitor advisories | Subscribe to Jenkins security list |

**Interview Tip**
- Mention using Plugin Installation Manager Tool for Docker images.
- Common mistake: Updating all plugins at once without testing.

---

<a id="q35"></a>
### Q35. How do you set up Jenkins High Availability?

**Scenario / Problem Statement**
- Jenkins is a critical part of your CI/CD infrastructure. A single Jenkins failure causes deployment delays. How do you make it highly available?

**What is being tested**
- Understanding of HA architectures.
- Disaster recovery planning.

**Thinking & Problem-Solving Approach**
- Evaluate CloudBees HA vs DIY solutions.
- Consider active-passive vs active-active.
- Address shared storage and state management.

**Solution / Best Practice**

**Option 1: Active-Passive with Shared Storage**
```
                    +------------------+
                    |  Load Balancer   |
                    +------------------+
                            |
            +---------------+---------------+
            |                               |
    +-------v-------+               +-------v-------+
    | Jenkins Primary|               | Jenkins Standby|
    | (Active)       |               | (Passive)      |
    +-------+-------+               +-------+-------+
            |                               |
            +---------------+---------------+
                            |
                    +-------v-------+
                    | Shared Storage |
                    | (EFS/NFS)      |
                    +---------------+
```

**Kubernetes deployment example:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jenkins
spec:
  replicas: 1  # Only one active at a time
  serviceName: jenkins
  template:
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
  volumeClaimTemplates:
  - metadata:
      name: jenkins-home
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "fast"
      resources:
        requests:
          storage: 100Gi
```

**Option 2: Jenkins with External Build Agents**
```
    +------------------+     +------------------+
    | Jenkins Master 1 |     | Jenkins Master 2 |
    +--------+---------+     +--------+---------+
             |                        |
    +--------v------------------------v--------+
    |          Kubernetes Cluster              |
    |  +-------+  +-------+  +-------+        |
    |  |Agent 1|  |Agent 2|  |Agent 3|        |
    |  +-------+  +-------+  +-------+        |
    +------------------------------------------+
```

**Key components for HA:**
- **Load balancer**: HAProxy, NGINX, or cloud LB
- **Shared storage**: NFS, EFS, or GlusterFS
- **Database**: External database for CloudBees
- **Backup**: Automated regular backups

**Interview Tip**
- Mention that true HA is complex; sometimes fast recovery is more practical.
- Common mistake: Not testing failover procedures.

---

<a id="q36"></a>
### Q36. Your Jenkins builds are slow due to limited agents. How do you scale horizontally?

**Scenario / Problem Statement**
- Build queue is always full. Builds wait 30+ minutes to start. Adding permanent agents is expensive and wasteful during off-hours.

**What is being tested**
- Understanding of dynamic scaling.
- Cost optimization.

**Thinking & Problem-Solving Approach**
- Use cloud-based dynamic agents.
- Implement auto-scaling based on queue length.
- Consider container-based agents.

**Solution / Best Practice**

**Option 1: EC2 Plugin for AWS**
```
Manage Jenkins → Nodes → Configure Clouds → Amazon EC2

AMI: ami-xxxx (preconfigured with Java)
Instance Type: m5.large
Label: linux-build
Max instances: 10
Idle termination time: 30 minutes
```

**Option 2: Kubernetes Plugin**
```yaml
# Pod template for dynamic agents
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        cpu: "1"
        memory: "2Gi"
      limits:
        cpu: "2"
        memory: "4Gi"
  - name: maven
    image: maven:3.8-jdk-11
    command: ["cat"]
    tty: true
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
```

**Jenkinsfile using Kubernetes agent:**
```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.8-jdk-11
                    command: ["cat"]
                    tty: true
            '''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

**Option 3: Docker Plugin**
```groovy
agent {
    docker {
        image 'maven:3.8-jdk-11'
        args '-v $HOME/.m2:/root/.m2'
    }
}
```

**Interview Tip**
- Mention cost savings from spot/preemptible instances.
- Common mistake: Not setting resource limits on dynamic agents.

---

<a id="q37"></a>
### Q37. How do you configure Jenkins to use Kubernetes pods as dynamic build agents?

**Scenario / Problem Statement**
- Your infrastructure is on Kubernetes. You want Jenkins agents to be ephemeral pods that spin up on demand and terminate after builds.

**What is being tested**
- Deep knowledge of Kubernetes Plugin.
- Pod template configuration.

**Thinking & Problem-Solving Approach**
- Install and configure Kubernetes plugin.
- Create pod templates for different build types.
- Optimize for fast startup time.

**Solution / Best Practice**

**Step 1: Configure Kubernetes cloud**
```yaml
# JCasC configuration
jenkins:
  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "https://kubernetes.default"
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
        jenkinsTunnel: "jenkins-agent.jenkins.svc.cluster.local:50000"
        containerCapStr: "50"
        podLabels:
          - key: "jenkins"
            value: "agent"
        templates:
          - name: "default"
            label: "jenkins-agent"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
                workingDir: "/home/jenkins/agent"
```

**Step 2: Create specialized pod templates**
```groovy
// In Jenkinsfile - Java build template
pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-jdk-17
    command: ["sleep", "infinity"]
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
    volumeMounts:
    - name: maven-cache
      mountPath: /root/.m2
  volumes:
  - name: maven-cache
    persistentVolumeClaim:
      claimName: maven-cache-pvc
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
    }
}
```

**Step 3: Multi-container pod for Docker builds**
```groovy
agent {
    kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  - name: tools
    image: alpine:latest
    command: ["sleep", "infinity"]
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
'''
    }
}
```

**Interview Tip**
- Mention caching (Maven repo, npm) for faster builds.
- Common mistake: Not setting resource requests/limits.

---

<a id="q38"></a>
### Q38. Jenkins is becoming slow. How do you diagnose and tune performance?

**Scenario / Problem Statement**
- Jenkins UI is slow, builds take longer to start, and the server occasionally becomes unresponsive.

**What is being tested**
- Performance troubleshooting skills.
- System administration knowledge.

**Thinking & Problem-Solving Approach**
- Analyze resource utilization.
- Check for common bottlenecks.
- Tune JVM and system settings.

**Solution / Best Practice**

**1. Monitor system resources:**
```bash
# CPU and memory
top -p $(pgrep -f jenkins)

# Disk I/O
iostat -x 1

# Jenkins metrics
curl -s http://jenkins:8080/metrics/currentUser/metrics | jq
```

**2. JVM tuning:**
```bash
# /etc/default/jenkins
JAVA_OPTS="-Xms4g -Xmx8g \
    -XX:+UseG1GC \
    -XX:+ParallelRefProcEnabled \
    -XX:+DisableExplicitGC \
    -XX:+HeapDumpOnOutOfMemoryError \
    -XX:HeapDumpPath=/var/log/jenkins/heapdump.hprof"
```

**3. Reduce master load:**
```groovy
// Run all builds on agents, not master
pipeline {
    agent { label 'build-agent' }  // Never use 'agent any' if master has executors
    ...
}

// Set master executors to 0
// Manage Jenkins → Nodes → Built-In Node → Configure → # of executors: 0
```

**4. Optimize job configurations:**
```groovy
options {
    // Limit build history
    buildDiscarder(logRotator(numToKeepStr: '10'))
    
    // Prevent long-running builds
    timeout(time: 1, unit: 'HOURS')
    
    // Disable concurrent builds
    disableConcurrentBuilds()
}
```

**5. Database optimization (if using external DB):**
```sql
-- Analyze slow queries
-- Ensure proper indexes on frequently queried tables
```

**6. Common performance fixes:**

| Issue | Solution |
|-------|----------|
| Slow UI | Reduce number of executors on master |
| Slow builds | Use faster agents or parallelize |
| High memory | Increase heap, reduce build history |
| Slow plugin | Disable/update problematic plugins |
| Disk full | Clean workspaces, remove old builds |

**Interview Tip**
- Mention monitoring plugins like Prometheus metrics.
- Common mistake: Running builds on the master node.

---

<a id="q39"></a>
### Q39. How do you implement audit logging for Jenkins compliance requirements?

**Scenario / Problem Statement**
- Your organization requires audit trails for all Jenkins activities for SOC2/PCI compliance. You need to track who did what and when.

**What is being tested**
- Compliance awareness.
- Audit logging implementation.

**Thinking & Problem-Solving Approach**
- Enable built-in audit logging.
- Use audit trail plugin.
- Ship logs to centralized logging system.

**Solution / Best Practice**

**1. Install Audit Trail Plugin:**
- Manage Jenkins → Plugins → Install "Audit Trail"

**2. Configure audit logging:**
```yaml
# JCasC configuration
unclassified:
  audit-trail:
    logBuildCause: true
    pattern: ".*/(?:configSubmit|doDelete|postBuildResult|enable|disable|cancelQueue|stop|toggleLogKeep|doWipeOutWorkspace|createItem|createView|toggleOffline|cancelQuietDown|quietDown|restart|safeRestart|safeExit|exit|doDisconnect|doLaunch|doMakeOffline|doMakeOnline|configure|build).*"
    loggers:
      - syslog:
          syslogServerHostname: "syslog.company.com"
          syslogServerPort: 514
          facility: "USER"
          messageFormat: "RFC_5424"
      - file:
          log: "/var/log/jenkins/audit.log"
          limit: 100
          count: 10
```

**3. Log format example:**
```
Jan 23 10:15:30 jenkins audit: user=john.doe action=configSubmit job=production-deploy ip=192.168.1.100
Jan 23 10:16:45 jenkins audit: user=admin action=build job=backend-service/main ip=192.168.1.101
```

**4. Ship to ELK/Splunk:**
```yaml
# Filebeat configuration
filebeat.inputs:
  - type: log
    paths:
      - /var/log/jenkins/audit.log
    fields:
      type: jenkins-audit
      
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "jenkins-audit-%{+yyyy.MM.dd}"
```

**5. Key events to track:**
- Job creation/modification/deletion
- Build execution
- User login/logout
- Configuration changes
- Plugin installation
- Credential access

**Interview Tip**
- Mention log retention requirements for compliance.
- Common mistake: Not tracking credential access events.

---

<a id="q40"></a>
### Q40. How do you safely upgrade Jenkins and its plugins?

**Scenario / Problem Statement**
- Jenkins LTS needs to be upgraded from 2.375.x to 2.401.x. Several plugins also need updates. How do you do this safely without breaking production pipelines?

**What is being tested**
- Upgrade planning skills.
- Risk mitigation.

**Thinking & Problem-Solving Approach**
- Test in staging environment first.
- Read changelogs and compatibility notes.
- Have rollback plan ready.

**Solution / Best Practice**

**Pre-upgrade checklist:**
```markdown
[ ] Backup $JENKINS_HOME
[ ] Export plugin list
[ ] Document current version
[ ] Check plugin compatibility
[ ] Review security advisories
[ ] Test in staging environment
[ ] Schedule maintenance window
[ ] Notify users
```

**Step 1: Backup**
```bash
# Stop Jenkins
systemctl stop jenkins

# Full backup
tar -czvf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins

# Export plugin list
curl -s http://jenkins:8080/pluginManager/api/json?depth=1 | \
    jq -r '.plugins[] | "\(.shortName):\(.version)"' > plugins-before.txt
```

**Step 2: Test in staging**
```bash
# Create staging from backup
docker run -d --name jenkins-staging \
    -v /backup/jenkins:/var/jenkins_home \
    -p 8081:8080 \
    jenkins/jenkins:2.401.1-lts

# Test critical pipelines
# Check plugin compatibility
# Verify integrations
```

**Step 3: Upgrade production**
```bash
# Upgrade package (Debian/Ubuntu)
apt update
apt install jenkins=2.401.1

# Or pull new Docker image
docker pull jenkins/jenkins:2.401.1-lts
```

**Step 4: Plugin updates**
```groovy
// Script to update plugins safely
Jenkins.instance.updateCenter.siteList.each { site ->
    site.updateDirectlyNow(true)
}

def plugins = Jenkins.instance.pluginManager.plugins
plugins.each { plugin ->
    def update = Jenkins.instance.updateCenter.getPlugin(plugin.shortName)
    if (update?.hasUpdates()) {
        println "Updating ${plugin.shortName} from ${plugin.version} to ${update.version}"
        update.deploy()
    }
}
```

**Rollback plan:**
```bash
# Stop Jenkins
systemctl stop jenkins

# Restore from backup
rm -rf /var/lib/jenkins/*
tar -xzvf jenkins-backup-YYYYMMDD.tar.gz -C /

# Start Jenkins
systemctl start jenkins
```

**Interview Tip**
- Mention using LTS versions for stability.
- Common mistake: Upgrading on Friday afternoon without testing.

---

## ✅ Section 4 Complete

**Questions covered:** Q31–Q40  
**Topics:** RBAC, LDAP integration, Credential security, Plugin management, High Availability, Horizontal scaling, Kubernetes agents, Performance tuning, Audit logging, Safe upgrades.

---

## Section 5: Advanced Jenkins, Debugging & Production Issues (Q41–Q50)

---

<a id="q41"></a>
### Q41. A pipeline is stuck and won't proceed or timeout. How do you diagnose and fix it?

**Scenario / Problem Statement**
- A production pipeline has been running for 6 hours. It's not showing any output and won't respond to abort. The stage shows as "In progress" but nothing is happening.

**What is being tested**
- Advanced debugging skills.
- Understanding of pipeline execution.

**Thinking & Problem-Solving Approach**
- Check thread dumps for deadlocks.
- Review agent connectivity.
- Identify the blocking step.

**Solution / Best Practice**

**1. Check pipeline paused points:**
```groovy
// Via Script Console
Jenkins.instance.getAllItems(org.jenkinsci.plugins.workflow.job.WorkflowJob).each { job ->
    job.builds.each { build ->
        def exec = build.getExecution()
        if (exec != null) {
            println "Job: ${job.fullName}, Build: ${build.number}"
            exec.getCurrentExecutions(true).each { e ->
                println "  Blocked at: ${e}"
            }
        }
    }
}
```

**2. Get thread dump:**
```bash
# Find Jenkins PID
PID=$(pgrep -f jenkins)

# Generate thread dump
kill -3 $PID

# Or via URL
curl -u admin:token http://jenkins:8080/threadDump
```

**3. Force abort stuck build:**
```groovy
// Script Console
def job = Jenkins.instance.getItemByFullName('folder/job-name')
def build = job.getBuildByNumber(123)

// Soft abort
build.doStop()

// Hard abort
build.getExecutor()?.interrupt()

// Force kill
build.doKill()
```

**4. Common causes and fixes:**

| Cause | Solution |
|-------|----------|
| `input` step waiting | Find and submit/abort input |
| Agent disconnected | Reconnect agent or abort |
| Infinite loop in script | Hard abort and fix code |
| Waiting for lock | Release lock or abort holding job |
| Groovy CPS transformation issue | Use `@NonCPS` annotation |

**5. Prevent future issues:**
```groovy
options {
    timeout(time: 1, unit: 'HOURS')
}

stage('Risky Step') {
    options {
        timeout(time: 10, unit: 'MINUTES')
    }
    steps {
        // ...
    }
}
```

**Interview Tip**
- Always set timeouts at pipeline and stage level.
- Common mistake: Not having timeout options configured.

---

<a id="q42"></a>
### Q42. Jenkins agent keeps disconnecting. How do you troubleshoot connectivity issues?

**Scenario / Problem Statement**
- A build agent frequently goes offline mid-build. Builds fail with "Agent went offline during the build."

**What is being tested**
- Network troubleshooting skills.
- Agent architecture understanding.

**Thinking & Problem-Solving Approach**
- Check network connectivity and firewall.
- Review agent logs.
- Analyze resource exhaustion.

**Solution / Best Practice**

**1. Check agent status:**
```bash
# On agent machine
# Check agent process
ps aux | grep agent.jar

# Check connectivity to master
telnet jenkins-master 50000
curl -v http://jenkins-master:8080/

# Check system resources
free -m
df -h
```

**2. Review agent logs:**
```bash
# SSH agent logs
cat /var/log/jenkins/agent.log

# JNLP agent logs (Windows)
type C:\Jenkins\agent.log
```

**3. Common causes:**

| Symptom | Cause | Solution |
|---------|-------|----------|
| Immediate disconnect | Firewall blocking port 50000 | Open port |
| Disconnect after time | Keep-alive timeout | Increase timeout |
| Disconnect during build | OOM on agent | Increase memory |
| Intermittent | Network instability | Use WebSocket |

**4. Enable WebSocket (more reliable):**
```
# Agent command line
java -jar agent.jar -url http://jenkins:8080/ \
    -secret xxxx \
    -name agent-1 \
    -webSocket
```

**5. Configure in agent settings:**
```yaml
# JCasC for permanent agent
jenkins:
  nodes:
    - permanent:
        name: "agent-1"
        remoteFS: "/var/jenkins"
        launcher:
          ssh:
            host: "agent-1.company.com"
            credentialsId: "ssh-agent-creds"
            sshHostKeyVerificationStrategy: "nonVerifyingKeyVerificationStrategy"
            launchTimeoutSeconds: 60
            maxNumRetries: 3
            retryWaitTime: 30
```

**6. Monitor agent health:**
```groovy
// Script to check agent status
Jenkins.instance.nodes.each { node ->
    def computer = node.toComputer()
    println "${node.name}: ${computer?.isOnline() ? 'ONLINE' : 'OFFLINE'}"
    if (!computer?.isOnline()) {
        println "  Reason: ${computer?.getOfflineCauseReason()}"
    }
}
```

**Interview Tip**
- WebSocket connection is more firewall-friendly than JNLP.
- Common mistake: Not monitoring agent health proactively.

---

<a id="q43"></a>
### Q43. Jenkins crashes after a plugin update. How do you recover?

**Scenario / Problem Statement**
- After updating plugins, Jenkins won't start. The log shows "Failed to load plugin" or Jenkins keeps restarting.

**What is being tested**
- Disaster recovery skills.
- Plugin troubleshooting.

**Thinking & Problem-Solving Approach**
- Check logs for specific plugin failure.
- Start in safe mode.
- Remove problematic plugin.

**Solution / Best Practice**

**1. Check error logs:**
```bash
# Jenkins logs
tail -f /var/log/jenkins/jenkins.log

# Or journalctl
journalctl -u jenkins -f
```

**2. Start in safe mode (disable all plugins):**
```bash
# Add to JAVA_OPTS
JAVA_OPTS="-Dhudson.model.LoadStatistics.clock=1000 -Dhudson.model.Hudson.noSlave=true"

# Or via URL
http://jenkins:8080/safeRestart

# Or
touch $JENKINS_HOME/plugins/.disable
systemctl restart jenkins
```

**3. Disable specific plugin:**
```bash
# Rename plugin files
cd $JENKINS_HOME/plugins/
mv problematic-plugin.jpi problematic-plugin.jpi.disabled
mv problematic-plugin problematic-plugin.disabled

# Restart Jenkins
systemctl restart jenkins
```

**4. Rollback plugin version:**
```bash
# Download previous version
wget https://updates.jenkins.io/download/plugins/problematic-plugin/1.0/problematic-plugin.hpi

# Replace plugin
mv problematic-plugin.hpi $JENKINS_HOME/plugins/
chown jenkins:jenkins $JENKINS_HOME/plugins/problematic-plugin.hpi

# Restart
systemctl restart jenkins
```

**5. Common plugin issues:**

| Error | Cause | Solution |
|-------|-------|----------|
| Class not found | Dependency missing | Install dependency |
| Version conflict | Incompatible plugins | Update both |
| Jenkins version | Plugin needs newer Jenkins | Upgrade or downgrade plugin |

**6. Restore from backup (last resort):**
```bash
systemctl stop jenkins
rm -rf $JENKINS_HOME/plugins/*
tar -xzvf jenkins-backup.tar.gz -C / --include="*/plugins/*"
systemctl start jenkins
```

**Interview Tip**
- Always backup before plugin updates.
- Common mistake: Updating many plugins at once.

---

<a id="q44"></a>
### Q44. Pipelines are taking much longer than before. How do you diagnose slow performance?

**Scenario / Problem Statement**
- A pipeline that used to take 10 minutes now takes 45 minutes. No code changes were made.

**What is being tested**
- Performance analysis skills.
- Pipeline optimization knowledge.

**Thinking & Problem-Solving Approach**
- Compare build times historically.
- Identify which stage is slow.
- Check external dependencies.

**Solution / Best Practice**

**1. Use Pipeline Stage View:**
- Click on build → Pipeline Steps
- Identify slow steps by duration

**2. Add timing to stages:**
```groovy
stage('Build') {
    steps {
        script {
            def startTime = System.currentTimeMillis()
            sh 'mvn clean package'
            def duration = System.currentTimeMillis() - startTime
            echo "Build took ${duration/1000} seconds"
        }
    }
}
```

**3. Enable pipeline durability logging:**
```groovy
// More logging for debugging
options {
    timestamps()
}
```

**4. Common slow points:**

| Issue | Diagnosis | Solution |
|-------|-----------|----------|
| Slow checkout | Large repo | Use shallow clone |
| Slow dependencies | Network/registry slow | Cache dependencies |
| Slow tests | More tests added | Parallelize tests |
| Slow agent start | Cold start | Use persistent agents |
| Resource contention | Shared resources | Increase capacity |

**5. Optimize checkout:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [
        [$class: 'CloneOption', depth: 1, shallow: true],
        [$class: 'CleanBeforeCheckout']
    ],
    userRemoteConfigs: [[url: 'https://github.com/org/repo.git']]
])
```

**6. Cache dependencies:**
```groovy
// Maven cache
agent {
    docker {
        image 'maven:3.8-jdk-11'
        args '-v $HOME/.m2:/root/.m2'  // Mount cache
    }
}

// npm cache
agent {
    docker {
        image 'node:18'
        args '-v $HOME/.npm:/root/.npm'
    }
}
```

**7. Analyze with Pipeline Metrics:**
```groovy
// Install "Pipeline Metrics Plugin"
// View at: Jenkins → Pipeline Metrics
```

**Interview Tip**
- Use Blue Ocean for visual pipeline analysis.
- Common mistake: Not caching build dependencies.

---

<a id="q45"></a>
### Q45. Git checkout fails with various errors. How do you troubleshoot SCM issues?

**Scenario / Problem Statement**
- Builds fail at checkout with errors like "Permission denied", "SSL certificate problem", or "reference not found".

**What is being tested**
- Git integration troubleshooting.
- Credential and network debugging.

**Thinking & Problem-Solving Approach**
- Check credential configuration.
- Verify network access.
- Test Git operations manually.

**Solution / Best Practice**

**1. Common errors and fixes:**

| Error | Cause | Solution |
|-------|-------|----------|
| Permission denied (publickey) | SSH key not configured | Add SSH credentials |
| SSL certificate problem | Self-signed cert | Configure Git to trust cert |
| Reference not found | Branch deleted | Verify branch exists |
| Repository not found | Wrong URL or permissions | Check URL and access |
| Timeout | Network issue | Check firewall/proxy |

**2. Fix SSH authentication:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'git@github.com:org/repo.git',
        credentialsId: 'github-ssh-key'
    ]]
])
```

**3. Fix SSL certificate issues:**
```groovy
// Option 1: Disable SSL verification (not recommended for production)
withEnv(['GIT_SSL_NO_VERIFY=true']) {
    git url: 'https://self-signed-gitlab.company.com/repo.git'
}

// Option 2: Add CA certificate
// On agent: git config --global http.sslCAInfo /path/to/ca-bundle.crt
```

**4. Fix credential issues:**
```groovy
// Use credential helper
withCredentials([usernamePassword(
    credentialsId: 'github-token',
    usernameVariable: 'GIT_USER',
    passwordVariable: 'GIT_PASS'
)]) {
    sh '''
        git config credential.helper store
        echo "https://${GIT_USER}:${GIT_PASS}@github.com" > ~/.git-credentials
        git clone https://github.com/org/repo.git
    '''
}
```

**5. Debug Git operations:**
```groovy
stage('Debug Checkout') {
    steps {
        sh '''
            # Test connectivity
            ssh -T git@github.com || true
            
            # Test HTTPS
            git ls-remote https://github.com/org/repo.git
            
            # Verbose clone
            GIT_TRACE=1 GIT_CURL_VERBOSE=1 git clone https://github.com/org/repo.git
        '''
    }
}
```

**6. Fix LFS issues:**
```groovy
checkout([
    $class: 'GitSCM',
    extensions: [
        [$class: 'GitLFSPull']
    ],
    ...
])
```

**Interview Tip**
- Always test credentials on the agent machine first.
- Common mistake: Using personal credentials instead of service accounts.

---

<a id="q46"></a>
### Q46. Build fails with "Disk space issue" or workspace corruption. How do you resolve?

**Scenario / Problem Statement**
- Builds fail randomly with "No space left on device" or strange compilation errors that disappear on retry.

**What is being tested**
- Storage management.
- Workspace handling.

**Thinking & Problem-Solving Approach**
- Monitor disk usage.
- Implement workspace cleanup.
- Investigate file corruption causes.

**Solution / Best Practice**

**1. Diagnose disk issues:**
```bash
# Check disk usage
df -h

# Find large directories
du -sh /var/lib/jenkins/workspace/* | sort -hr | head -20

# Find large build artifacts
find /var/lib/jenkins/jobs -name "*.log" -size +100M
```

**2. Immediate cleanup:**
```bash
# Clean all workspaces
rm -rf /var/lib/jenkins/workspace/*

# Clean specific job workspace
rm -rf /var/lib/jenkins/workspace/job-name*

# Clean old builds
find /var/lib/jenkins/jobs/*/builds -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
```

**3. Automatic cleanup in pipeline:**
```groovy
pipeline {
    agent any
    
    options {
        // Clean workspace before build
        skipDefaultCheckout()
    }
    
    stages {
        stage('Clean & Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    
    post {
        always {
            // Clean workspace after build
            cleanWs(
                cleanWhenNotBuilt: false,
                deleteDirs: true,
                disableDeferredWipeout: true,
                patterns: [
                    [pattern: '.gitignore', type: 'INCLUDE'],
                    [pattern: '.propsfile', type: 'EXCLUDE']
                ]
            )
        }
    }
}
```

**4. Fix workspace corruption:**
```groovy
options {
    // Use fresh workspace each build
    checkoutToSubdirectory('src')
}

// Or wipe workspace
stage('Setup') {
    steps {
        deleteDir()  // Delete entire workspace
        checkout scm
    }
}
```

**5. Configure disk monitoring:**
```groovy
// Install "Disk Usage Plugin"
// Set threshold in Manage Jenkins → Configure System
// Free Space Warning Threshold: 10GB
```

**Interview Tip**
- Use `cleanWs()` plugin instead of manual deletion.
- Common mistake: Not configuring build retention policies.

---

<a id="q47"></a>
### Q47. Jenkins is using too much memory and experiencing OOM. How do you identify and fix memory leaks?

**Scenario / Problem Statement**
- Jenkins memory usage grows continuously until it crashes with OutOfMemoryError. Restarting provides temporary relief.

**What is being tested**
- JVM memory analysis.
- Memory leak identification.

**Thinking & Problem-Solving Approach**
- Capture heap dumps.
- Analyze memory usage patterns.
- Identify leaking objects.

**Solution / Best Practice**

**1. Configure JVM for diagnostics:**
```bash
JAVA_OPTS="-Xms4g -Xmx8g \
    -XX:+HeapDumpOnOutOfMemoryError \
    -XX:HeapDumpPath=/var/log/jenkins/heapdump.hprof \
    -XX:+PrintGCDetails \
    -XX:+PrintGCDateStamps \
    -Xloggc:/var/log/jenkins/gc.log"
```

**2. Monitor memory usage:**
```groovy
// Script Console - check memory
def runtime = Runtime.getRuntime()
println "Max Memory: ${runtime.maxMemory() / 1024 / 1024} MB"
println "Total Memory: ${runtime.totalMemory() / 1024 / 1024} MB"
println "Free Memory: ${runtime.freeMemory() / 1024 / 1024} MB"
println "Used Memory: ${(runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024} MB"
```

**3. Trigger heap dump:**
```bash
# Find Jenkins PID
PID=$(pgrep -f jenkins)

# Generate heap dump
jmap -dump:format=b,file=/tmp/jenkins-heap.hprof $PID

# Analyze with Eclipse MAT or VisualVM
```

**4. Common memory leak sources:**

| Source | Solution |
|--------|----------|
| Too many builds retained | Configure build discarder |
| Large console logs | Truncate logs |
| Plugin memory leak | Update/remove plugin |
| Too many executors | Reduce executor count |
| Flyweight executor leak | Upgrade Jenkins |

**5. Fix common issues:**
```groovy
// Limit console log size
options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
}

// Truncate long output
stage('Build') {
    steps {
        sh 'make build 2>&1 | tail -1000'  // Keep only last 1000 lines
    }
}
```

**6. Reduce memory footprint:**
```groovy
// Disable unnecessary features
-Dhudson.model.DirectoryBrowserSupport.CSP="default-src 'self'"
-Dhudson.security.ArtifactsPermission=true
-Dhudson.model.UpdateCenter.never=true
```

**Interview Tip**
- Regular Jenkins restarts can be a valid strategy while investigating.
- Common mistake: Setting heap too large (causes long GC pauses).

---

<a id="q48"></a>
### Q48. How do you migrate Jenkins to a new server with minimal downtime?

**Scenario / Problem Statement**
- Jenkins needs to be migrated from an old server to new infrastructure. You need to minimize downtime and ensure all jobs, configurations, and history are preserved.

**What is being tested**
- Migration planning.
- Risk management.

**Thinking & Problem-Solving Approach**
- Plan the migration phases.
- Test in staging first.
- Have rollback plan ready.

**Solution / Best Practice**

**Migration plan:**
```markdown
1. Pre-migration (Days before)
   [ ] Document current setup
   [ ] Set up new server with same Jenkins version
   [ ] Test network connectivity
   
2. Sync phase (Day before)
   [ ] Initial rsync of JENKINS_HOME
   [ ] Configure new server
   
3. Cutover (Maintenance window)
   [ ] Stop Jenkins on old server
   [ ] Final rsync of changes
   [ ] Start Jenkins on new server
   [ ] Update DNS/load balancer
   
4. Validation
   [ ] Verify all jobs present
   [ ] Test critical pipelines
   [ ] Verify agent connectivity
```

**Step 1: Initial sync (while old Jenkins running):**
```bash
# Sync everything except workspaces
rsync -avz --progress \
    --exclude='workspace/' \
    --exclude='caches/' \
    --exclude='logs/' \
    jenkins@old-server:/var/lib/jenkins/ \
    /var/lib/jenkins/
```

**Step 2: Final cutover:**
```bash
# Stop old Jenkins
ssh old-server "systemctl stop jenkins"

# Final sync
rsync -avz --delete \
    --exclude='workspace/' \
    jenkins@old-server:/var/lib/jenkins/ \
    /var/lib/jenkins/

# Fix permissions
chown -R jenkins:jenkins /var/lib/jenkins

# Start new Jenkins
systemctl start jenkins
```

**Step 3: Update references:**
```groovy
// Update Jenkins URL
// Manage Jenkins → System → Jenkins URL

// Update agent configurations
// For each agent, update the Jenkins URL in agent.jar download
```

**Step 4: Verify migration:**
```groovy
// Script Console
println "Jobs: ${Jenkins.instance.allItems.size()}"
println "Nodes: ${Jenkins.instance.nodes.size()}"
println "Users: ${User.getAll().size()}"

Jenkins.instance.nodes.each { node ->
    println "Node ${node.name}: ${node.toComputer()?.isOnline() ? 'ONLINE' : 'OFFLINE'}"
}
```

**Docker migration alternative:**
```bash
# Export from old server
docker exec jenkins tar -czvf - /var/jenkins_home > jenkins-backup.tar.gz

# Import to new server
cat jenkins-backup.tar.gz | docker exec -i jenkins tar -xzvf - -C /
```

**Interview Tip**
- Test the migration procedure in staging first.
- Common mistake: Not updating the Jenkins URL after migration.

---

<a id="q49"></a>
### Q49. How do you set up Jenkins in an air-gapped environment without internet access?

**Scenario / Problem Statement**
- Your organization has a secure network with no internet access. You need to install and maintain Jenkins including plugins.

**What is being tested**
- Understanding of offline operations.
- Plugin dependency management.

**Thinking & Problem-Solving Approach**
- Download all dependencies externally.
- Set up local mirrors.
- Document update procedures.

**Solution / Best Practice**

**1. Download Jenkins WAR:**
```bash
# On internet-connected machine
wget https://get.jenkins.io/war-stable/latest/jenkins.war

# Transfer to air-gapped server
scp jenkins.war airgapped-server:/opt/jenkins/
```

**2. Download plugins with dependencies:**
```bash
# Use Plugin Installation Manager Tool
# On internet-connected machine

# Download the tool
wget https://github.com/jenkinsci/plugin-installation-manager-tool/releases/download/2.12.8/jenkins-plugin-manager-2.12.8.jar

# Create plugins.txt
cat > plugins.txt << EOF
git:latest
workflow-aggregator:latest
pipeline-utility-steps:latest
EOF

# Download plugins with dependencies
java -jar jenkins-plugin-manager-2.12.8.jar \
    --plugin-file plugins.txt \
    --plugin-download-directory ./plugins \
    --war /path/to/jenkins.war

# Transfer plugins
tar -czvf plugins.tar.gz plugins/
scp plugins.tar.gz airgapped-server:/opt/jenkins/
```

**3. Install plugins offline:**
```bash
# On air-gapped server
tar -xzvf plugins.tar.gz -C $JENKINS_HOME/plugins/
chown -R jenkins:jenkins $JENKINS_HOME/plugins/
systemctl restart jenkins
```

**4. Create local update center:**
```bash
# Mirror update-center.json
wget https://updates.jenkins.io/update-center.json

# Set up local web server
# nginx configuration
server {
    listen 80;
    server_name jenkins-updates.internal;
    root /var/www/jenkins-updates;
    
    location / {
        autoindex on;
    }
}
```

**5. Configure Jenkins to use local update center:**
```groovy
// Script Console
def site = Jenkins.instance.updateCenter.getSite('default')
site.url = 'http://jenkins-updates.internal/update-center.json'
site.save()
```

**6. Offline Docker setup:**
```bash
# On internet-connected machine
# Pull and save images
docker pull jenkins/jenkins:lts
docker save jenkins/jenkins:lts > jenkins-lts.tar

# Transfer and load
scp jenkins-lts.tar airgapped-server:
docker load < jenkins-lts.tar
```

**Interview Tip**
- Document the update process for the team.
- Common mistake: Forgetting plugin dependencies.

---

<a id="q50"></a>
### Q50. Design a Jenkins architecture for a large enterprise with 500+ developers and 1000+ jobs.

**Scenario / Problem Statement**
- You're designing Jenkins infrastructure for a large organization. Requirements: high availability, multi-team isolation, security, scalability, and audit compliance.

**What is being tested**
- Enterprise architecture skills.
- Comprehensive understanding of Jenkins at scale.

**Thinking & Problem-Solving Approach**
- Consider team isolation and governance.
- Plan for scalability and resilience.
- Implement security and compliance.

**Solution / Best Practice**

**Enterprise Architecture:**
```
                        +---------------------+
                        |   Load Balancer     |
                        |   (HAProxy/F5)      |
                        +----------+----------+
                                   |
            +----------------------+----------------------+
            |                      |                      |
    +-------v-------+     +--------v------+     +--------v------+
    | Jenkins Master|     | Jenkins Master|     | Jenkins Master|
    | (Team A)      |     | (Team B)      |     | (Shared)      |
    +-------+-------+     +-------+-------+     +-------+-------+
            |                     |                     |
    +-------v-------+     +-------v-------+     +-------v-------+
    | K8s Namespace |     | K8s Namespace |     | K8s Namespace |
    | (Team A Agents)|    | (Team B Agents)|    | (Shared Agents)|
    +---------------+     +---------------+     +---------------+
            |                     |                     |
            +---------------------+---------------------+
                                  |
                        +---------v---------+
                        |   Shared Services |
                        | - Artifact Repo   |
                        | - Secret Manager  |
                        | - Log Aggregation |
                        | - Metrics         |
                        +-------------------+
```

**Key components:**

**1. Multi-master strategy:**
```yaml
# Separate Jenkins instances per team/domain
jenkins-platform-team:
  url: https://jenkins-platform.company.com
  purpose: Shared services, infrastructure

jenkins-team-a:
  url: https://jenkins-team-a.company.com
  purpose: Team A applications

jenkins-team-b:
  url: https://jenkins-team-b.company.com
  purpose: Team B applications
```

**2. Standardized configuration with JCasC:**
```yaml
# jenkins.yaml - shared across all instances
jenkins:
  systemMessage: "Production Jenkins - ${TEAM_NAME}"
  
  securityRealm:
    ldap:
      configurations:
        - server: "ldaps://ad.company.com"
          rootDN: "DC=company,DC=com"
          
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            assignments:
              - "jenkins-admins"
          - name: "developer"
            permissions:
              - "Job/Build"
              - "Job/Read"
            assignments:
              - "developers"

  clouds:
    - kubernetes:
        name: "k8s"
        namespace: "${TEAM_NAMESPACE}"
        containerCapStr: "100"
```

**3. Shared library governance:**
```
jenkins-shared-libraries/
├── vars/
│   ├── standardBuild.groovy      # Enforced standards
│   ├── securityScan.groovy       # Required security
│   └── complianceChecks.groovy   # Compliance gates
├── src/
│   └── com/company/
│       ├── BuildConfig.groovy
│       └── DeploymentRules.groovy
└── resources/
    └── templates/
```

**4. Security controls:**
```groovy
// Mandatory security scan in all pipelines
pipeline {
    agent any
    stages {
        stage('Build') { ... }
        
        stage('Security Gate') {
            steps {
                script {
                    // Enforced by shared library
                    complianceChecks.runSecurityScan()
                    complianceChecks.checkVulnerabilities(maxCritical: 0)
                }
            }
        }
        
        stage('Deploy') { ... }
    }
}
```

**5. Monitoring and observability:**
```yaml
# Prometheus scrape config
scrape_configs:
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: 
        - 'jenkins-platform:8080'
        - 'jenkins-team-a:8080'
        - 'jenkins-team-b:8080'

# Key metrics to monitor
- jenkins_job_count_value
- jenkins_queue_size_value
- jenkins_node_online_count
- jenkins_builds_duration_milliseconds
```

**6. Disaster recovery:**
```bash
# Automated backup to S3
0 */4 * * * /opt/jenkins/backup.sh

# backup.sh
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d-%H%M)
tar -czvf /tmp/jenkins-${JENKINS_INSTANCE}-${TIMESTAMP}.tar.gz \
    --exclude='workspace' \
    --exclude='caches' \
    $JENKINS_HOME

aws s3 cp /tmp/jenkins-${JENKINS_INSTANCE}-${TIMESTAMP}.tar.gz \
    s3://jenkins-backups/${JENKINS_INSTANCE}/
```

**Enterprise checklist:**

| Area | Implementation |
|------|----------------|
| Authentication | LDAP/SAML with MFA |
| Authorization | Role-based with folder isolation |
| Secrets | HashiCorp Vault integration |
| Audit | Centralized logging to Splunk/ELK |
| HA | Multiple masters, shared storage |
| Scaling | Kubernetes dynamic agents |
| Compliance | Enforced via shared libraries |
| Backup | Automated to object storage |
| Monitoring | Prometheus + Grafana |
| Updates | Staged rollout with testing |

**Interview Tip**
- Emphasize governance and standardization across teams.
- Common mistake: Single Jenkins for entire organization (creates bottleneck).

---

## ✅ Section 5 Complete

**Questions covered:** Q41–Q50  
**Topics:** Stuck pipelines, Agent connectivity, Plugin crashes, Performance diagnosis, Git issues, Workspace problems, Memory leaks, Server migration, Air-gapped setup, Enterprise architecture.

---

## 🎯 Final Summary

Congratulations! You've completed the **Jenkins Interview Preparation Guide** with all **50 scenario-based questions**.

### Quick Reference by Topic

| Section | Questions | Key Topics |
|---------|-----------|------------|
| 1 | Q1-Q10 | Architecture, Installation, Jobs, GitHub Integration |
| 2 | Q11-Q20 | Pipelines, Groovy, Shared Libraries, Credentials |
| 3 | Q21-Q30 | CI/CD Design, Deployments, Kubernetes, Artifacts |
| 4 | Q31-Q40 | Security, RBAC, HA, Scaling, Administration |
| 5 | Q41-Q50 | Debugging, Performance, Migration, Enterprise |

### Interview Tips Recap

1. **Always explain your thought process** - Interviewers want to see how you analyze problems
2. **Mention trade-offs** - Every solution has pros and cons
3. **Be honest about limitations** - It's okay to say "I would research this further"
4. **Use real examples** - Reference actual scenarios you've encountered
5. **Stay updated** - Jenkins evolves; mention recent features when relevant

### Good luck with your interview! 🚀

---

> 📝 **Say "Continue with Section 2"** to proceed with **Jenkins Jobs, Pipelines & Groovy (Q11–Q20)**

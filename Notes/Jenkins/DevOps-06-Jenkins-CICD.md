# The Complete DevOps Engineer's Reference Guide
## Part 6: Jenkins CI/CD Complete Guide

---

# Chapter 18: Jenkins Fundamentals

## 18.1 Jenkins Architecture

```
Jenkins Architecture:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                        Jenkins Controller                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐    │   │
│  │  │   Web UI     │  │ REST API     │  │ Plugin Manager │    │   │
│  │  └──────────────┘  └──────────────┘  └────────────────┘    │   │
│  │                                                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐    │   │
│  │  │ Job Scheduler│  │ Build Queue  │  │ Credential Mgr │    │   │
│  │  └──────────────┘  └──────────────┘  └────────────────┘    │   │
│  │                                                             │   │
│  │  Storage: $JENKINS_HOME (jobs, configs, plugins, secrets)  │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│              ┌───────────────┼───────────────┐                      │
│              ▼               ▼               ▼                      │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │
│  │  Agent 1         │ │  Agent 2         │ │  Agent 3         │    │
│  │  (Linux)         │ │  (Windows)       │ │  (Kubernetes)    │    │
│  │                  │ │                  │ │                  │    │
│  │  - Executor 1    │ │  - Executor 1    │ │  - Dynamic pods  │    │
│  │  - Executor 2    │ │  - Executor 2    │ │  - Auto-scaling  │    │
│  │                  │ │                  │ │                  │    │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Key Concepts:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Controller (Master):                                               │
│  ├── Manages agents                                                 │
│  ├── Schedules builds                                               │
│  ├── Stores configuration                                           │
│  ├── Serves web UI                                                  │
│  └── Should NOT run builds (delegate to agents)                    │
│                                                                     │
│  Agent (Slave/Node):                                                │
│  ├── Executes builds                                                │
│  ├── Can be permanent or dynamic                                    │
│  ├── Has executors (parallel build slots)                          │
│  └── Connected via SSH, JNLP, or Kubernetes                        │
│                                                                     │
│  Executor:                                                          │
│  ├── A build slot on an agent                                       │
│  ├── Runs one build at a time                                       │
│  └── Agent with 2 executors can run 2 builds simultaneously        │
│                                                                     │
│  Job/Project:                                                       │
│  ├── A defined task (build, test, deploy)                          │
│  ├── Can be freestyle, pipeline, or multibranch                    │
│  └── Contains configuration and build history                      │
│                                                                     │
│  Build:                                                             │
│  ├── Single execution of a job                                      │
│  ├── Has build number, status, logs                                │
│  └── Produces artifacts                                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 18.2 Jenkins Directory Structure

```
$JENKINS_HOME Structure:
├── config.xml                 # Main Jenkins configuration
├── credentials.xml            # Stored credentials
├── secrets/                   # Encrypted secrets
│   ├── master.key            # Master encryption key
│   ├── hudson.util.Secret    # Secret encryption
│   └── initialAdminPassword  # Initial setup password
├── plugins/                   # Installed plugins
│   ├── plugin-name.jpi       # Plugin archive
│   └── plugin-name/          # Extracted plugin
├── users/                     # User configurations
│   └── admin/
│       └── config.xml
├── jobs/                      # Job configurations
│   └── my-job/
│       ├── config.xml        # Job configuration
│       ├── builds/           # Build history
│       │   ├── 1/
│       │   │   ├── build.xml
│       │   │   ├── log
│       │   │   └── changelog.xml
│       │   └── 2/
│       └── workspace/        # Working directory (can be elsewhere)
├── workspace/                # Alternative workspace location
├── logs/                     # Jenkins logs
├── nodes/                    # Agent configurations
│   └── my-agent/
│       └── config.xml
├── updates/                  # Update center data
└── war/                      # Exploded WAR file

Important files for backup:
├── config.xml
├── credentials.xml
├── secrets/
├── users/
├── jobs/*/config.xml         # Job configs (not builds)
└── nodes/
```

## 18.3 Jenkins Configuration as Code (JCasC)

```yaml
# jenkins.yaml - Configuration as Code
jenkins:
  systemMessage: "Jenkins configured automatically by JCasC"
  
  # Security configuration
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${JENKINS_ADMIN_PASSWORD}"
  
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            entries:
              - user: "admin"
          - name: "developer"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Read"
              - "Job/Workspace"
            entries:
              - group: "developers"
  
  # Agents
  nodes:
    - permanent:
        name: "linux-agent"
        labelString: "linux docker"
        numExecutors: 4
        remoteFS: "/home/jenkins"
        launcher:
          ssh:
            host: "agent.example.com"
            port: 22
            credentialsId: "agent-ssh-key"
  
  # Cloud (Kubernetes)
  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "https://kubernetes.default.svc"
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins.jenkins.svc:8080"
        podLabels:
          - key: "jenkins"
            value: "agent"
        templates:
          - name: "default"
            label: "default"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
                command: ""
                args: ""
                workingDir: "/home/jenkins/agent"
                resourceRequestCpu: "500m"
                resourceRequestMemory: "256Mi"
                resourceLimitCpu: "1000m"
                resourceLimitMemory: "512Mi"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: "github-creds"
              username: "${GITHUB_USERNAME}"
              password: "${GITHUB_TOKEN}"
              description: "GitHub credentials"
          - string:
              scope: GLOBAL
              id: "sonar-token"
              secret: "${SONAR_TOKEN}"
              description: "SonarQube token"
          - basicSSHUserPrivateKey:
              scope: GLOBAL
              id: "agent-ssh-key"
              username: "jenkins"
              privateKeySource:
                directEntry:
                  privateKey: "${SSH_PRIVATE_KEY}"

unclassified:
  location:
    url: "https://jenkins.example.com/"
    adminAddress: "devops@example.com"
  
  gitHubPluginConfig:
    configs:
      - name: "GitHub"
        apiUrl: "https://api.github.com"
        credentialsId: "github-creds"
  
  sonarGlobalConfiguration:
    buildWrapperEnabled: true
    installations:
      - name: "SonarQube"
        serverUrl: "https://sonar.example.com"
        credentialsId: "sonar-token"

tool:
  git:
    installations:
      - name: "Default"
        home: "git"
  
  maven:
    installations:
      - name: "Maven 3.9"
        properties:
          - installSource:
              installers:
                - maven:
                    id: "3.9.4"
  
  nodejs:
    installations:
      - name: "NodeJS 18"
        properties:
          - installSource:
              installers:
                - nodeJSInstaller:
                    id: "18.17.0"
                    npmPackages: ""
```

---

# Chapter 19: Jenkins Pipelines

## 19.1 Declarative Pipeline Syntax

```groovy
// Complete Declarative Pipeline Reference
pipeline {
    // Where to run
    agent any  // Run on any available agent
    // OR
    agent none  // Don't allocate agent at pipeline level
    // OR
    agent { label 'linux && docker' }  // Specific labels
    // OR
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            label 'docker'
            args '-v /root/.m2:/root/.m2'
        }
    }
    // OR
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command:
    - sleep
    args:
    - infinity
'''
        }
    }
    
    // Pipeline options
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep last 10 builds
        timeout(time: 1, unit: 'HOURS')                 // Pipeline timeout
        disableConcurrentBuilds()                       // No parallel runs
        timestamps()                                     // Add timestamps to log
        ansiColor('xterm')                              // Color output
        skipDefaultCheckout()                           // Don't auto-checkout
        retry(3)                                         // Retry on failure
        preserveStashes(buildCount: 5)                  // Keep stashes
    }
    
    // Environment variables
    environment {
        // Static values
        DEPLOY_ENV = 'production'
        APP_NAME = 'myapp'
        
        // From credentials
        GITHUB_TOKEN = credentials('github-token')      // Secret text
        DOCKER_CREDS = credentials('docker-hub')        // Username/password
        // DOCKER_CREDS_USR and DOCKER_CREDS_PSW are created
        
        // Computed values
        VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT?.take(7)}"
    }
    
    // Parameters (user input)
    parameters {
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Branch to build'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'production'],
            description: 'Target environment'
        )
        booleanParam(
            name: 'DEPLOY',
            defaultValue: true,
            description: 'Deploy after build?'
        )
        password(
            name: 'API_KEY',
            defaultValue: '',
            description: 'API key for deployment'
        )
    }
    
    // Triggers
    triggers {
        cron('H 2 * * *')                              // Nightly at 2 AM
        pollSCM('H/5 * * * *')                         // Poll every 5 minutes
        upstream(
            upstreamProjects: 'job1,job2',
            threshold: hudson.model.Result.SUCCESS
        )
        githubPush()                                    // GitHub webhook
    }
    
    // Tools
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 17'
        nodejs 'NodeJS 18'
    }
    
    // Stages
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                // OR explicit
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${params.BRANCH}"]],
                    extensions: [
                        [$class: 'CleanBeforeCheckout'],
                        [$class: 'CloneOption', depth: 1, shallow: true]
                    ],
                    userRemoteConfigs: [[
                        url: 'https://github.com/org/repo.git',
                        credentialsId: 'github-creds'
                    ]]
                ])
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                // For Windows: bat 'mvn clean package'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -DskipUnitTests'
                    }
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
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${env.APP_NAME}:${env.VERSION}")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.example.com', 'docker-creds') {
                        docker.image("${env.APP_NAME}:${env.VERSION}").push()
                        docker.image("${env.APP_NAME}:${env.VERSION}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy.sh dev'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh staging'
            }
        }
        
        stage('Deploy to Production') {
            when {
                allOf {
                    branch 'main'
                    expression { params.DEPLOY == true }
                }
            }
            input {
                message "Deploy to production?"
                ok "Deploy"
                submitter "admin,release-managers"
                parameters {
                    string(name: 'REASON', defaultValue: '', description: 'Deployment reason')
                }
            }
            steps {
                sh './deploy.sh production'
            }
        }
    }
    
    // Post-build actions
    post {
        always {
            // Always run
            cleanWs()
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
        success {
            slackSend(
                channel: '#builds',
                color: 'good',
                message: "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                channel: '#builds',
                color: 'danger',
                message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
            emailext(
                subject: "Build Failed: ${env.JOB_NAME}",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
        unstable {
            echo 'Build is unstable (test failures)'
        }
        aborted {
            echo 'Build was aborted'
        }
        cleanup {
            // Always run after all other post conditions
            deleteDir()
        }
    }
}
```

## 19.2 When Conditions

```groovy
stage('Conditional Stage') {
    when {
        // Branch conditions
        branch 'main'
        branch pattern: 'release-*', comparator: 'GLOB'
        branch pattern: 'PR-\\d+', comparator: 'REGEXP'
        
        // Tag conditions
        tag 'v*'
        tag pattern: 'v\\d+\\.\\d+\\.\\d+', comparator: 'REGEXP'
        
        // Environment conditions
        environment name: 'DEPLOY_TO', value: 'production'
        
        // Expression
        expression { return params.RUN_TESTS == true }
        expression { env.BRANCH_NAME == 'main' && currentBuild.previousBuild?.result == 'FAILURE' }
        
        // Change conditions
        changeset '**/src/**'              // Only if src changed
        changeset pattern: '.*\\.java$', comparator: 'REGEXP'
        
        // Build cause
        triggeredBy 'TimerTrigger'
        triggeredBy 'UpstreamCause'
        triggeredBy cause: 'UserIdCause', detail: 'admin'
        
        // Changelog
        changelog '.*\\[ci-skip\\].*'      // Skip if commit has [ci-skip]
        
        // Combining conditions
        allOf {
            branch 'main'
            environment name: 'DEPLOY', value: 'true'
        }
        anyOf {
            branch 'main'
            branch 'develop'
        }
        not {
            branch 'feature/*'
        }
    }
    steps {
        echo 'Condition met!'
    }
}
```

## 19.3 Scripted Pipeline

```groovy
// Scripted Pipeline (more flexible but complex)
node('linux') {
    // Properties
    properties([
        buildDiscarder(logRotator(numToKeepStr: '10')),
        parameters([
            string(name: 'ENVIRONMENT', defaultValue: 'dev')
        ])
    ])
    
    try {
        stage('Checkout') {
            checkout scm
        }
        
        stage('Build') {
            // Use credentials
            withCredentials([
                usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker build -t myapp .
                '''
            }
        }
        
        stage('Test') {
            // Parallel execution
            parallel(
                'Unit Tests': {
                    sh 'npm run test:unit'
                },
                'Integration Tests': {
                    sh 'npm run test:integration'
                }
            )
        }
        
        stage('Deploy') {
            // Conditional
            if (params.ENVIRONMENT == 'production') {
                // Manual approval
                input message: 'Deploy to production?'
            }
            
            // Retry with timeout
            retry(3) {
                timeout(time: 5, unit: 'MINUTES') {
                    sh './deploy.sh'
                }
            }
        }
        
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Cleanup
        cleanWs()
        
        // Notifications
        if (currentBuild.result == 'SUCCESS') {
            slackSend(color: 'good', message: 'Build succeeded')
        } else {
            slackSend(color: 'danger', message: 'Build failed')
        }
    }
}
```

## 19.4 Shared Libraries

```groovy
// vars/buildPipeline.groovy - Shared Library
def call(Map config) {
    pipeline {
        agent any
        
        stages {
            stage('Build') {
                steps {
                    script {
                        if (config.buildTool == 'maven') {
                            sh 'mvn clean package'
                        } else if (config.buildTool == 'npm') {
                            sh 'npm ci && npm run build'
                        }
                    }
                }
            }
            
            stage('Test') {
                steps {
                    script {
                        testApp(config)
                    }
                }
            }
            
            stage('Deploy') {
                when {
                    expression { config.deploy == true }
                }
                steps {
                    script {
                        deployApp(config)
                    }
                }
            }
        }
    }
}

// vars/testApp.groovy
def call(Map config) {
    if (config.buildTool == 'maven') {
        sh 'mvn test'
        junit 'target/surefire-reports/*.xml'
    } else if (config.buildTool == 'npm') {
        sh 'npm test'
        junit 'test-results/*.xml'
    }
}

// vars/deployApp.groovy
def call(Map config) {
    withCredentials([
        file(credentialsId: config.kubeconfigId, variable: 'KUBECONFIG')
    ]) {
        sh """
            kubectl apply -f k8s/
            kubectl rollout status deployment/${config.appName}
        """
    }
}

// Usage in Jenkinsfile
@Library('my-shared-library') _

buildPipeline(
    buildTool: 'maven',
    appName: 'my-app',
    deploy: true,
    kubeconfigId: 'aks-kubeconfig'
)
```

---

# Chapter 20: Jenkins + Kubernetes Integration

## 20.1 Kubernetes Plugin Configuration

```groovy
// Pod template with multiple containers
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins-agent: "true"
spec:
  containers:
  # JNLP container (required for agent connection)
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  
  # Maven container
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command:
    - sleep
    args:
    - infinity
    volumeMounts:
    - name: maven-cache
      mountPath: /root/.m2
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
  
  # Docker-in-Docker container
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run
  
  # kubectl container
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - sleep
    args:
    - infinity
  
  # Helm container
  - name: helm
    image: alpine/helm:latest
    command:
    - sleep
    args:
    - infinity
  
  # SonarQube scanner
  - name: sonar
    image: sonarsource/sonar-scanner-cli:latest
    command:
    - sleep
    args:
    - infinity
  
  # Trivy scanner
  - name: trivy
    image: aquasec/trivy:latest
    command:
    - sleep
    args:
    - infinity
  
  volumes:
  - name: maven-cache
    persistentVolumeClaim:
      claimName: maven-cache-pvc
  - name: docker-socket
    emptyDir: {}
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
        
        stage('Test') {
            steps {
                container('maven') {
                    sh 'mvn test'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                container('sonar') {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=myapp \
                              -Dsonar.sources=src \
                              -Dsonar.java.binaries=target/classes
                        '''
                    }
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t myacr.azurecr.io/myapp:${BUILD_NUMBER} .
                    '''
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                container('trivy') {
                    sh '''
                        trivy image --severity HIGH,CRITICAL \
                          --exit-code 1 \
                          myacr.azurecr.io/myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Push Image') {
            steps {
                container('docker') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'acr-creds',
                            usernameVariable: 'ACR_USER',
                            passwordVariable: 'ACR_PASS'
                        )
                    ]) {
                        sh '''
                            docker login myacr.azurecr.io -u $ACR_USER -p $ACR_PASS
                            docker push myacr.azurecr.io/myapp:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                container('helm') {
                    withCredentials([
                        file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')
                    ]) {
                        sh '''
                            helm upgrade --install myapp ./charts/myapp \
                              --namespace dev \
                              --set image.tag=${BUILD_NUMBER} \
                              --wait --timeout 5m
                        '''
                    }
                }
            }
        }
        
        stage('Smoke Test') {
            steps {
                container('kubectl') {
                    withCredentials([
                        file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')
                    ]) {
                        sh '''
                            kubectl rollout status deployment/myapp -n dev
                            kubectl get pods -n dev -l app=myapp
                        '''
                    }
                }
            }
        }
    }
}
```

## 20.2 Complete CI/CD Pipeline for Kubernetes

```groovy
@Library('jenkins-shared-library') _

pipeline {
    agent none
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        timestamps()
    }
    
    environment {
        REGISTRY = 'myacr.azurecr.io'
        APP_NAME = 'myapp'
        CHART_PATH = './charts/myapp'
    }
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                }
                stash includes: '**', name: 'source'
            }
        }
        
        stage('Build & Test') {
            agent {
                kubernetes {
                    yaml podTemplate('maven')
                }
            }
            steps {
                unstash 'source'
                container('maven') {
                    sh '''
                        mvn clean verify \
                          -Dmaven.test.failure.ignore=false
                    '''
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Code Analysis') {
            parallel {
                stage('SonarQube') {
                    agent {
                        kubernetes {
                            yaml podTemplate('sonar')
                        }
                    }
                    steps {
                        unstash 'source'
                        container('sonar') {
                            withSonarQubeEnv('SonarQube') {
                                sh 'sonar-scanner'
                            }
                        }
                    }
                }
                
                stage('Dependency Check') {
                    agent {
                        kubernetes {
                            yaml podTemplate('maven')
                        }
                    }
                    steps {
                        unstash 'source'
                        container('maven') {
                            sh 'mvn org.owasp:dependency-check-maven:check'
                        }
                    }
                    post {
                        always {
                            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                        }
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            agent any
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build & Push Image') {
            agent {
                kubernetes {
                    yaml podTemplate('docker')
                }
            }
            steps {
                unstash 'source'
                container('docker') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'acr-credentials',
                            usernameVariable: 'ACR_USER',
                            passwordVariable: 'ACR_PASS'
                        )
                    ]) {
                        sh '''
                            # Build image
                            docker build \
                              --build-arg VERSION=${IMAGE_TAG} \
                              -t ${REGISTRY}/${APP_NAME}:${IMAGE_TAG} \
                              -t ${REGISTRY}/${APP_NAME}:latest \
                              .
                            
                            # Login and push
                            echo $ACR_PASS | docker login ${REGISTRY} -u $ACR_USER --password-stdin
                            docker push ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                            docker push ${REGISTRY}/${APP_NAME}:latest
                        '''
                    }
                }
            }
        }
        
        stage('Image Security Scan') {
            agent {
                kubernetes {
                    yaml podTemplate('trivy')
                }
            }
            steps {
                container('trivy') {
                    sh '''
                        trivy image \
                          --severity HIGH,CRITICAL \
                          --format table \
                          ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        
        stage('Deploy to Dev') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'feature/*'
                }
            }
            agent {
                kubernetes {
                    yaml podTemplate('helm')
                }
            }
            environment {
                NAMESPACE = 'dev'
            }
            steps {
                unstash 'source'
                container('helm') {
                    withKubeConfig([credentialsId: 'aks-kubeconfig']) {
                        sh '''
                            helm upgrade --install ${APP_NAME} ${CHART_PATH} \
                              --namespace ${NAMESPACE} \
                              --create-namespace \
                              --set image.repository=${REGISTRY}/${APP_NAME} \
                              --set image.tag=${IMAGE_TAG} \
                              --values ${CHART_PATH}/values-${NAMESPACE}.yaml \
                              --wait --timeout 10m
                        '''
                    }
                }
            }
        }
        
        stage('Integration Tests') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            agent {
                kubernetes {
                    yaml podTemplate('test')
                }
            }
            steps {
                container('test') {
                    sh './run-integration-tests.sh'
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            agent {
                kubernetes {
                    yaml podTemplate('helm')
                }
            }
            environment {
                NAMESPACE = 'staging'
            }
            steps {
                unstash 'source'
                container('helm') {
                    withKubeConfig([credentialsId: 'aks-kubeconfig']) {
                        sh '''
                            helm upgrade --install ${APP_NAME} ${CHART_PATH} \
                              --namespace ${NAMESPACE} \
                              --set image.repository=${REGISTRY}/${APP_NAME} \
                              --set image.tag=${IMAGE_TAG} \
                              --values ${CHART_PATH}/values-${NAMESPACE}.yaml \
                              --wait --timeout 10m
                        '''
                    }
                }
            }
        }
        
        stage('Performance Tests') {
            when {
                branch 'main'
            }
            agent {
                kubernetes {
                    yaml podTemplate('k6')
                }
            }
            steps {
                container('k6') {
                    sh 'k6 run --out json=results.json tests/performance/load-test.js'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results.json'
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to Production?"
                ok "Deploy"
                submitter "admin,release-team"
                parameters {
                    string(name: 'CONFIRM', defaultValue: '', description: 'Type "DEPLOY" to confirm')
                }
            }
            agent {
                kubernetes {
                    yaml podTemplate('helm')
                }
            }
            environment {
                NAMESPACE = 'production'
            }
            steps {
                script {
                    if (env.CONFIRM != 'DEPLOY') {
                        error('Deployment not confirmed')
                    }
                }
                unstash 'source'
                container('helm') {
                    withKubeConfig([credentialsId: 'aks-kubeconfig']) {
                        sh '''
                            # Blue-green deployment
                            helm upgrade --install ${APP_NAME} ${CHART_PATH} \
                              --namespace ${NAMESPACE} \
                              --set image.repository=${REGISTRY}/${APP_NAME} \
                              --set image.tag=${IMAGE_TAG} \
                              --values ${CHART_PATH}/values-${NAMESPACE}.yaml \
                              --wait --timeout 15m
                        '''
                    }
                }
            }
        }
        
        stage('Verify Production') {
            when {
                branch 'main'
            }
            agent {
                kubernetes {
                    yaml podTemplate('kubectl')
                }
            }
            steps {
                container('kubectl') {
                    withKubeConfig([credentialsId: 'aks-kubeconfig']) {
                        sh '''
                            kubectl rollout status deployment/${APP_NAME} -n production
                            kubectl get pods -n production -l app=${APP_NAME}
                            
                            # Smoke test
                            POD=$(kubectl get pod -n production -l app=${APP_NAME} -o jsonpath='{.items[0].metadata.name}')
                            kubectl exec $POD -n production -- curl -sf localhost:8080/health
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            script {
                if (env.BRANCH_NAME == 'main') {
                    slackSend(
                        channel: '#releases',
                        color: 'good',
                        message: """
                            :white_check_mark: *${APP_NAME} ${IMAGE_TAG}* deployed to production
                            Build: ${BUILD_URL}
                        """
                    )
                }
            }
        }
        failure {
            slackSend(
                channel: '#devops-alerts',
                color: 'danger',
                message: """
                    :x: *Build Failed:* ${JOB_NAME} #${BUILD_NUMBER}
                    Branch: ${BRANCH_NAME}
                    Build: ${BUILD_URL}
                """
            )
        }
    }
}

// Helper function for pod templates
def podTemplate(String type) {
    switch(type) {
        case 'maven':
            return '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: [sleep, infinity]
    volumeMounts:
    - name: m2-cache
      mountPath: /root/.m2
  volumes:
  - name: m2-cache
    persistentVolumeClaim:
      claimName: maven-cache
'''
        case 'docker':
            return '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
'''
        // Add more templates...
        default:
            return '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: default
    image: alpine
    command: [sleep, infinity]
'''
    }
}
```

---

This concludes Part 6 covering Jenkins CI/CD.

**Continue to Part 7 for:**
- Observability (Monitoring, Logging, Tracing)
- Application Knowledge (Java/Node.js for DevOps)
- Troubleshooting Guide

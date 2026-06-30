# Jenkins Groovy & Shared Libraries

> **last_reviewed:** 2026-06-30  
> **Prerequisites:** [Jenkins CI/CD](DevOps-06-Jenkins-CICD.md) | **Troubleshooting:** [Jenkins](../../Troubleshooting/Jenkins/07-Jenkins-Troubleshooting.md) | **Project:** [PetClinic](../../Projects/PetClinic.md)

---

## 1. Pipeline types

| Type | Syntax | When to use |
|------|--------|-------------|
| **Declarative** | `pipeline { }` | Default — structured, Jenkins validates |
| **Scripted** | `node { }` | Legacy — full Groovy freedom |
| **Shared library** | `@Library('my-lib') _` | Reuse across many Jenkinsfiles |

---

## 2. Declarative pipeline anatomy

```groovy
pipeline {
    agent { label 'deploy01' }

    parameters {
        choice(name: 'environment', choices: ['dev', 'staging', 'prod'], description: 'Target env')
        string(name: 'image_tag', defaultValue: 'latest', description: 'Container tag')
        booleanParam(name: 'dry_run', defaultValue: false, description: 'Plan only')
    }

    environment {
        TF_IN_AUTOMATION = 'true'
        ACR_REGISTRY = 'myregistry.azurecr.io'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timeout(time: 2, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            when { not { params.dry_run } }
            steps {
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (params.environment == 'prod') {
                        input message: 'Deploy to PROD?', ok: 'Deploy'
                    }
                    deployToK8s(params.environment, params.image_tag)
                }
            }
        }
    }

    post {
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
        always  { cleanWs() }
    }
}
```

---

## 3. `script { }` block

Declarative stages only allow certain steps. Use `script { }` for Groovy logic:

```groovy
stage('Setup parameters') {
    steps {
        script {
            properties([
                parameters([
                    choice(name: 'region', choices: getRegions(), description: 'Azure region')
                ])
            ])
        }
    }
}

def getRegions() {
    return ['eastus', 'eastus2', 'westus2']
}
```

---

## 4. Loading external Groovy modules (`load`)

Enterprise pattern — keep Jenkinsfile thin, logic in `Groovy_modules/`:

```groovy
stage('Process values') {
    steps {
        script {
            def processor = load "${env.WORKSPACE}/devops/Groovy_modules/ValuesYamlProcessor.groovy"
            processor.run(
                environment: params.environment,
                chartPath: 'helm/myapp',
                valuesFile: "values-${params.environment}.yaml"
            )
        }
    }
}
```

**ValuesYamlProcessor.groovy:**

```groovy
def run(Map config) {
    echo "Processing ${config.valuesFile} for ${config.environment}"

    def values = readYaml file: "${config.chartPath}/${config.valuesFile}"
    values.global.environment = config.environment
    values.image.tag = env.BUILD_NUMBER

    writeYaml file: "${config.chartPath}/values-generated.yaml", data: values
    return values
}

return this
```

**Rules:**
- File must end with `return this`
- Call via `load` path relative to workspace
- Keep modules focused (one responsibility per file)

---

## 5. Credentials binding

```groovy
stage('Push to ACR') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'acr-service-principal',
                usernameVariable: 'ACR_USER',
                passwordVariable: 'ACR_PASS'
            ),
            string(credentialsId: 'azure-subscription-id', variable: 'AZURE_SUB_ID')
        ]) {
            sh '''
                echo "$ACR_PASS" | docker login $ACR_REGISTRY -u $ACR_USER --password-stdin
                docker push $ACR_REGISTRY/myapp:${BUILD_NUMBER}
            '''
        }
    }
}
```

**Azure Key Vault (plugin):**

```groovy
def secrets = [
    [path: 'kv/sql-connection', secretValues: [
        [vaultKey: 'password', envVar: 'SQL_PASSWORD']
    ]]
]
wrap([$class: 'VaultBuildWrapper', vaultSecrets: secrets]) {
    sh 'deploy-with-sql.sh'
}
```

See [Private Link & Key Vault](../Azure/Azure-Private-Link-KeyVault-CICD.md).

---

## 6. Shared library structure

```
shared-library/
├── vars/
│   ├── deployToK8s.groovy      # Global vars — call as deployToK8s()
│   └── notifySlack.groovy
├── src/
│   └── org/mycompany/
│       └── PipelineUtils.groovy  # Classes — import org.mycompany.PipelineUtils
└── resources/
    └── templates/k8s-deploy.sh
```

**vars/deployToK8s.groovy:**

```groovy
def call(String environment, String imageTag) {
    sh """
        helm upgrade --install myapp ./helm/myapp \
          -f values-${environment}.yaml \
          --set image.tag=${imageTag} \
          -n myapp-${environment} --create-namespace
    """
}
```

**Jenkinsfile usage:**

```groovy
@Library('my-company-lib@v2') _

pipeline {
    stages {
        stage('Deploy') {
            steps {
                deployToK8s('dev', env.BUILD_NUMBER)
            }
        }
    }
}
```

Configure library: **Manage Jenkins → System → Global Pipeline Libraries**

---

## 7. Passing data between stages

```groovy
stage('Build') {
    steps {
        script {
            env.IMAGE_DIGEST = sh(returnStdout: true, script: 'docker inspect --format="{{index .RepoDigests 0}}" myapp:latest').trim()
        }
    }
}

stage('Deploy') {
    steps {
        echo "Deploying digest: ${env.IMAGE_DIGEST}"
    }
}
```

For complex data, use `@NonCPS` methods or stash/unstash:

```groovy
stage('Build') {
    steps {
        writeFile file: 'artifacts/version.txt', text: env.BUILD_NUMBER
        stash includes: 'artifacts/**', name: 'build-artifacts'
    }
}

stage('Deploy') {
    steps {
        unstash 'build-artifacts'
        sh 'cat artifacts/version.txt'
    }
}
```

---

## 8. Kubernetes agents

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: terraform
    image: hashicorp/terraform:1.6
    command: ['cat']
    tty: true
'''
        }
    }
    stages {
        stage('Plan') {
            steps {
                container('terraform') {
                    sh 'terraform plan'
                }
            }
        }
    }
}
```

---

## 9. Proxy / no_proxy (corporate networks)

```groovy
environment {
    HTTPS_PROXY = 'http://proxy.corp.com:3128'
    HTTP_PROXY  = 'http://proxy.corp.com:3128'
    NO_PROXY    = 'localhost,127.0.0.1,.privatelink.database.windows.net,.azurecr.io,.vault.azure.net'
}
```

Missing privatelink domains in `NO_PROXY` causes silent failures to Azure private endpoints.

---

## 10. Common errors

| Error | Cause | Fix |
|-------|-------|-----|
| `MissingPropertyException` | Typo in `params.` or `env.` | Use `echo` to debug bindings |
| `load` returns null | No `return this` | Add to end of Groovy module |
| Credential not found | Wrong credentialsId | Match Jenkins credential store ID exactly |
| `script security` rejection | Admin approval needed | In-process Script Approval |
| Concurrent build conflict | Same workspace | `disableConcurrentBuilds()` or unique workspace |

---

## Related

- [Jenkins Troubleshooting](../../Troubleshooting/Jenkins/07-Jenkins-Troubleshooting.md)
- [Helm Enterprise Patterns](../Helm/Helm-Enterprise-Patterns.md)
- [PetClinic Phase 4](../../Projects/PetClinic.md)

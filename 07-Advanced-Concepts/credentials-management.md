# Credentials Management in Jenkins

## 1. Why Credentials Management Matters

CI/CD pipelines interact with:

* Git repositories
* Docker registries
* Cloud providers
* Artifact repositories
* External APIs

Hardcoding secrets inside jobs or Jenkinsfiles creates:

* Source control exposure
* Log leakage
* Lateral movement risk
* Compliance violations

Jenkins provides a centralized, encrypted credentials store.

---

## 2. Credentials Storage Model

Credentials are stored:

* Encrypted on disk
* Scoped to specific domains
* Bound at runtime to jobs

Managed via:

**Manage Jenkins → Credentials**

---

## 3. Credential Scope

### Global

Available to all jobs.

### System

Available only internally (e.g., agent connections).

### Folder-level

Restricted to jobs inside a specific folder.

Principle: Minimize scope. Never default to Global unless required.

---

## 4. Supported Credential Types

### Username & Password

Used for:

* Git HTTPS auth
* Basic authentication

### SSH Username with Private Key

Used for:

* Git SSH cloning
* Agent connections

### Secret Text

Used for:

* API tokens
* Slack tokens
* Webhooks

### Secret File

Used for:

* Service account JSON files
* Certificates

---

## 5. Creating Credentials

Steps:

1. Manage Jenkins
2. Credentials
3. Select scope
4. Add Credentials
5. Choose type
6. Provide:

   * ID (explicit name)
   * Description
   * Secret values

Always define a clear ID. That ID is referenced in pipelines.

---

## 6. Using Credentials in Pipeline

### Example: Username & Password

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'git-creds',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )
                ]) {
                    sh 'echo $GIT_USER'
                }
            }
        }
    }
}
```

Secrets are masked in logs automatically.

---

### Example: Secret Text

```groovy
withCredentials([
    string(credentialsId: 'slack-token', variable: 'SLACK_TOKEN')
]) {
    sh 'echo $SLACK_TOKEN'
}
```

---

### Example: SSH Key

```groovy
sshagent(['ssh-key-id']) {
    sh 'git clone git@github.com:repo/project.git'
}
```

---

## 7. Binding in Declarative Pipeline

Recommended pattern:

```groovy
environment {
    AWS_ACCESS_KEY_ID     = credentials('aws-creds-id')
}
```

Or with full binding:

```groovy
environment {
    MY_SECRET = credentials('secret-id')
}
```

---

## 8. Masking Behavior

Jenkins:

* Masks secrets in console output
* Replaces matched values with `****`
* Does not mask transformed values

Avoid:

* Printing secrets
* Passing secrets into debug logs
* Echoing environment dumps

---

## 9. Credentials with Docker

For Docker Hub authentication:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'dockerhub-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )
]) {
    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
}
```

---

## 10. Credentials Security Model

Jenkins encryption:

* Master key stored on controller
* Secrets encrypted in `credentials.xml`
* Compromise of controller compromises secrets

Mitigation:

* Restrict controller access
* Use OS-level encryption
* Integrate external secret managers

---

## 11. External Secret Integrations

Common integrations:

* HashiCorp Vault
* AWS Secrets Manager
* Azure Key Vault

Benefits:

* Dynamic secrets
* Secret rotation
* Centralized governance
* Reduced secret sprawl

---

## 12. Best Practices

1. Never hardcode secrets in Jenkinsfile
2. Use least-privilege credentials
3. Use folder-scoped credentials
4. Rotate secrets regularly
5. Use separate credentials per environment
6. Avoid global shared admin credentials
7. Audit credential usage

---

## 13. Common Failure Patterns

* Reusing one credential everywhere
* Storing cloud root keys
* Printing secrets for debugging
* Granting global read access to jobs
* Backing up unencrypted Jenkins home

---

## 14. Mental Model

Credentials in Jenkins are:

* Not environment variables
* Not plain text files
* Not Git-stored secrets

They are:

Encrypted runtime-bound secret references injected only when needed.

Control scope. Control blast radius. Reduce trust surface.

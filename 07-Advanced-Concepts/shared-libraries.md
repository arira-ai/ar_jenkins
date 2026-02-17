# Shared Libraries in Jenkins

## 1. Purpose

Shared Libraries solve duplication in pipelines.

Without them:

* Logic repeated across Jenkinsfiles
* No version control of pipeline logic
* Hard-to-maintain CI standards

Shared Libraries centralize reusable pipeline code.

---

## 2. What a Shared Library Is

A versioned Git repository containing:

* Reusable pipeline functions
* Custom steps
* Groovy classes
* Standardized CI logic

Loaded automatically or explicitly in Jenkins pipelines.

---

## 3. Repository Structure

Standard structure:

```
(root)
 ├── vars/
 │    └── buildApp.groovy
 ├── src/
 │    └── com/company/Utils.groovy
 └── resources/
      └── config.yaml
```

### vars/

Defines global pipeline steps.

Each file becomes a callable step.

Example:
`vars/buildApp.groovy`

```groovy
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "mvn clean package"
                }
            }
        }
    }
}
```

### src/

Reusable Groovy classes.

Example:
`src/com/company/Utils.groovy`

```groovy
package com.company

class Utils implements Serializable {
    def sayHello() {
        return "Hello"
    }
}
```

### resources/

Static files accessed with `libraryResource`.

---

## 4. Configuring a Shared Library

Steps:

1. Manage Jenkins
2. Configure System
3. Global Pipeline Libraries
4. Add Library
5. Provide:

   * Name
   * Default version (branch/tag)
   * Retrieval method (Git URL)

Example Git hosting:

* GitHub
* GitLab
* Bitbucket

---

## 5. Loading the Library

### Automatically (recommended)

Configured globally and available to all pipelines.

Usage:

```groovy
@Library('ci-lib') _
buildApp(config: [:])
```

### Dynamically

```groovy
library identifier: 'ci-lib@main',
        retriever: modernSCM([
            $class: 'GitSCMSource',
            remote: 'https://github.com/org/ci-lib.git'
        ])
```

---

## 6. Global Variables (vars/)

Each file in `vars/`:

* Must define `def call(...)`
* Becomes a custom pipeline step

Example usage:

```groovy
buildApp(
    branch: 'main',
    environment: 'dev'
)
```

Treat these as domain-specific language (DSL) extensions.

---

## 7. Using Classes from src/

Import normally:

```groovy
import com.company.Utils

def u = new Utils()
echo u.sayHello()
```

Rules:

* Must implement `Serializable`
* Avoid heavy state retention
* Avoid non-serializable objects

---

## 8. Versioning Strategy

Always version libraries.

Options:

* Branch (main, develop)
* Tag (v1.0.0)
* Commit SHA

Example:

```groovy
@Library('ci-lib@v1.2.0') _
```

Pin versions for production stability.

---

## 9. Trusted vs Untrusted Libraries

### Trusted

* Full script approval bypass
* Runs with admin privileges
* High risk if compromised

### Untrusted

* Subject to sandbox restrictions
* Safer for multi-team environments

Default to untrusted unless control is strict.

---

## 10. Accessing Resources

```groovy
def yaml = libraryResource 'config.yaml'
writeFile file: 'config.yaml', text: yaml
```

Used for:

* Deployment templates
* Configuration
* Helm values
* JSON schemas

---

## 11. Enterprise Design Pattern

Central CI Team:

* Maintains shared library
* Defines standard build, test, deploy stages

Application Teams:

* Minimal Jenkinsfile
* Delegate logic to library

Example minimal Jenkinsfile:

```groovy
@Library('enterprise-ci@v3') _
appPipeline(
    language: 'java',
    deploy: true
)
```

All complexity moves to controlled library.

---

## 12. Anti-Patterns

* Putting entire pipeline logic in Jenkinsfile
* Copy-paste across repos
* Using shared library without version pinning
* Embedding secrets in library
* Allowing uncontrolled library updates

---

## 13. Mental Model

Jenkinsfiles define intent.

Shared Libraries define implementation.

This separation creates:

* Governance
* Standardization
* Version control of CI logic
* Reduced duplication
* Controlled evolution

Treat pipeline logic as software.

Version it.
Test it.
Review it.
Control it.

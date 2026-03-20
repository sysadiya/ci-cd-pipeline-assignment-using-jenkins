# SonarQube Integration Guide for Jenkins Pipeline

## ✅ Solution: Fixed SonarQube Maven Plugin "Not Found" Error

The issue was that the **sonar-maven-plugin should NOT be defined in pom.xml**. Instead, it should be executed as a separate Maven goal during the Jenkins pipeline.

### Why This Approach?
- The sonar-maven-plugin is a CLI tool, not a build dependency
- It doesn't need to be in the POM to work
- This avoids repository issues and authentication problems
- More flexible - you can control when it runs in the pipeline

---

## How to Use SonarQube in Your Jenkins Pipeline

### Prerequisites
1. SonarQube Server running (e.g., `http://localhost:9000` or your cloud instance)
2. SonarQube authentication token (generate from SonarQube UI under Administration > Security > Users)
3. Java 11+ installed on Jenkins agent

### Jenkins Pipeline Stage Example

Add this stage to your `Jenkinsfile`:

```groovy
stage('Build & Tests') {
    steps {
        script {
            sh '''
                cd customer-order-api
                mvn clean package -DskipTests
            '''
        }
    }
}

stage('Run Tests with Coverage') {
    steps {
        script {
            sh '''
                cd customer-order-api
                mvn test
            '''
        }
    }
}

stage('SonarQube Analysis') {
    environment {
        SONAR_HOST_URL = credentials('sonar-server-url')  // e.g., http://sonarqube:9000
        SONAR_LOGIN = credentials('sonar-token')           // Authentication token
    }
    steps {
        script {
            sh '''
                cd customer-order-api
                mvn sonar:sonar \
                  -Dsonar.projectKey=customer-order-api \
                  -Dsonar.projectName="Customer Order API" \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=${SONAR_LOGIN}
            '''
        }
    }
}
```

---

## Declarative Pipeline Example

```groovy
pipeline {
    agent any
    
    environment {
        SONAR_HOST = 'http://sonarqube:9000'
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Package') {
            steps {
                sh '''
                    cd customer-order-api
                    mvn clean package -DskipTests
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    cd customer-order-api
                    mvn test
                '''
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                sh '''
                    cd customer-order-api
                    mvn sonar:sonar \
                      -Dsonar.projectKey=customer-order-api \
                      -Dsonar.projectName="Customer Order API" \
                      -Dsonar.host.url=$SONAR_HOST \
                      -Dsonar.login=$SONAR_TOKEN \
                      -Dsonar.java.coveragePlugin=jacoco \
                      -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                '''
            }
        }
    }
    
    post {
        always {
            junit 'customer-order-api/target/surefire-reports/**/*.xml'
            publishHTML([
                reportDir: 'customer-order-api/target/site/jacoco',
                reportFiles: 'index.html',
                reportName: 'JaCoCo Coverage Report'
            ])
        }
    }
}
```

---

## Jenkins Credentials Setup

You need to create two Jenkins credentials:

### 1. SonarQube Server URL (String)
- **ID**: `sonar-server-url`
- **Secret**: `http://your-sonarqube-server:9000`

### 2. SonarQube Authentication Token (Secret Text)
- **ID**: `sonar-token`
- **Secret**: Generate token from SonarQube:
  - Login to SonarQube → Avatar (top right) → My Account → Security → Generate Tokens
  - Copy the generated token

---

## Maven Properties in pom.xml ✅

Your `pom.xml` already has the correct SonarQube properties:

```xml
<properties>
    <java.version>17</java.version>
    <sonar.projectKey>customer-order-api</sonar.projectKey>
    <sonar.projectName>Customer Order API</sonar.projectName>
    <sonar.java.coveragePlugin>jacoco</sonar.java.coveragePlugin>
    <sonar.coverage.jacoco.xmlReportPaths>${project.build.directory}/site/jacoco/jacoco.xml</sonar.coverage.jacoco.xmlReportPaths>
</properties>
```

This ensures JaCoCo coverage reports are properly linked to SonarQube analysis.

---

## Common SonarQube Parameters

| Parameter | Description |
|-----------|-------------|
| `sonar.projectKey` | Unique project identifier in SonarQube |
| `sonar.projectName` | Human-readable project name |
| `sonar.host.url` | SonarQube server URL |
| `sonar.login` | Authentication token |
| `sonar.java.coveragePlugin` | Coverage tool (jacoco, cobertura, etc.) |
| `sonar.coverage.jacoco.xmlReportPaths` | Path to JaCoCo XML report |
| `sonar.exclusions` | Patterns to exclude from analysis |

---

## Troubleshooting

### Issue: "Cannot connect to SonarQube"
- **Solution**: Check `sonar.host.url` is correct and SonarQube is running
- Verify network connectivity from Jenkins to SonarQube

### Issue: "Authentication Failed"
- **Solution**: Verify the token is valid and hasn't expired
- Generate a new token from SonarQube UI

### Issue: "No coverage report found"
- **Solution**: Ensure JaCoCo plugin runs before SonarQube analysis
- Run `mvn test` first to generate coverage data
- Verify `target/site/jacoco/jacoco.xml` exists

### Issue: "Cannot find sonar:sonar goal"
- **Solution**: This is correct! The goal is provided by Maven plugin at runtime
- Just run `mvn sonar:sonar` - Maven will download the plugin automatically

---

## Next Steps

1. **Configure Jenkins Credentials** (see above)
2. **Update your Jenkinsfile** with the pipeline stage
3. **Run the pipeline** - SonarQube will analyze after tests pass
4. **View Results** - Check SonarQube dashboard for code quality metrics

---

## Files Modified
- ✅ `pom.xml` - Removed sonar-maven-plugin (not needed in pom.xml)
- ✅ Kept JaCoCo plugin for coverage reporting
- ✅ Sonar properties configured correctly

Your project is now ready for SonarQube integration! 🚀


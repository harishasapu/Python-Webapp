pipeline{
    agent any
    tools{
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages{
        stage("GitCheckout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/harishasapu/Python-Webapp.git'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Trivy FS Scan"){
            steps{
                sh "trivy fs . > trivyfs.xt"
            }
        }
        stage("SonarQube Analysis"){
            steps{
                withSonarQubeEnv('SonarQube'){
                   sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=python-webapp -Dsonar.projectKey=python-webapp -Dsonar.java.binaries=. "
                }
            }
        }
        stage("Quality Gate"){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
            }
        }
        stage("Docker Build % Tag"){
            steps{
                sh "make image"
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image --format table harishasapu/python-webapp:latest"
            }
        }
        stage("Docker push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
                        sh "make push"
                    }
                }
            }
        }
        stage("Docker Deploy"){
            steps{
                sh "docker run -d --name webapp -p 5000:5000 harishasapu/python-webapp:latest"
            }
        }
    }
}

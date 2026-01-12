pipeline {
    agent any

    parameters {
        string(name: 'phase1', defaultValue: 'clean test', description: 'mvn phase')
        string(name: 'phase2', defaultValue: 'package', description: 'mvn phase')
        string(name: 'version', defaultValue: '3.4.0-SNAPSHOT', description: 'snap version')
    }

    environment {
        ARTIFACT_ID  = "spring-petclinic"
        SNAP_VERSION = "${params.version}"
        IMAGE_NAME   = "devopswithdayanand/spring-pet-project-m2:${SNAP_VERSION}"
        MAIL_FROM    = "22053590@kiit.ac.in"
        MAIL_TO      = "22053590@kiit.ac.in"
    }

    stages {

        stage('Notify Start') {
            steps {
                emailext(
                    from: "${MAIL_FROM}",
                    to: "${MAIL_TO}",
                    subject: "[Jenkins] ${JOB_NAME} - Build Started",
                    body: """
Hi Team,

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}
Status: STARTED

Regards,
Jenkins
"""
                )
            }
        }

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Testing') {
            steps {
                sh "mvn ${params.phase1}"
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonar-cred') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Package') {
            steps {
                sh "mvn ${params.phase2}"
                archiveArtifacts artifacts: "target/*.jar", fingerprint: true
            }
        }

        stage('Nexus Upload') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'spring-petclinic',
                        classifier: '',
                        file: "target/spring-petclinic-${SNAP_VERSION}.jar",
                        type: 'jar'
                    ]],
                    credentialsId: 'Nexus_cred',
                    groupId: 'org.springframework.samples',
                    nexusUrl: '172.31.39.206:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'spring-petclinic',
                    version: "${SNAP_VERSION}"
                )
            }
        }

        stage('Docker Image Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Image Push') {
            steps {
                sh "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh "aws eks --region ap-northeast-1 update-kubeconfig --name demo-cluster"
                sh "helm upgrade --install petclinic ./petclinic-helm"
            }
        }
    }

    post {

        success {
            emailext(
                from: "${MAIL_FROM}",
                to: "${MAIL_TO}",
                subject: "[Jenkins] ${JOB_NAME} #${BUILD_NUMBER} - SUCCESS",
                body: """
Hi Team,

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}
Status: SUCCESS
Branch: ${GIT_BRANCH}
Commit: ${GIT_COMMIT}

Logs: ${BUILD_URL}console
Artifacts: ${BUILD_URL}artifact/

Regards,
Jenkins
"""
            )
        }

        failure {
            emailext(
                from: "${MAIL_FROM}",
                to: "${MAIL_TO}",
                subject: "[Jenkins] ${JOB_NAME} #${BUILD_NUMBER} - FAILED",
                body: """
Hi Team,

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}
Status: FAILED
Branch: ${GIT_BRANCH}
Commit: ${GIT_COMMIT}

Logs: ${BUILD_URL}console
Artifacts: ${BUILD_URL}artifact/

Regards,
Jenkins
"""
            )
        }
    }
}

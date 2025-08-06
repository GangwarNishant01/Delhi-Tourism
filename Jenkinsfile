@Library('Shared') _
pipeline {
    agent any
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'DOCKER_IMAGE_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.DOCKER_IMAGE_TAG == '') {
                        error("DOCKER_IMAGE_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/GangwarNishant01/Delhi-Tourism.git","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","tourism-node-app","nishant-key")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('Delhi-Tourism'){
                            docker_build("tourism-node-app","${params.DOCKER_IMAGE_TAG}","gangwarnishant")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("tourism-node-app","${params.DOCKER_IMAGE_TAG}","gangwarnishant") 
                }
            }
        }
    }
    post{
        success{
            build job: "Delhi-Tourism-CD", parameters: [
                string(name: 'DOCKER_IMAGE_TAG', value: "${params.DOCKER_IMAGE_TAG}"),
            ]
        }
    }
}
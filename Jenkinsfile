pipeline {

    agent any
    //escenarios -> escenario -> pasos
    environment{
        NPM_CONFIG_CACHE="${WORKSPACE}/.npm"
        dockerimagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = 'gcp-registry'
    }
    stages{
        stage ("proceso de build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }

            stages {
                stage("instalacion de dependencias"){
                    steps{
                        sh 'npm ci'
                    }
                }
                stage("ejecucion de pruebas"){
                    steps{
                        sh 'npm run test:cov'
                    }
                }
                stage("construccion de la aplicacion"){
                    steps{
                        sh 'npm run build'
                    }
                }

            }

        }
        stage ("build y push de imagen docker"){
            steps{
                script{
                    docker.withRegistry("${registry}", registryCredentials){
                        sh "docker build -t backend-nest-test-MLM ."
                        sh "docker tag backend-nest-test-MLM ${dockerimagePrefix}/backend-nest-test-MLM"
                        sh "docker tag backend-nest-test-MLM ${dockerimagePrefix}/backend-nest-test-MLM:${BUILD_NUMBER}"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-MLM"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-MLM:${BUILD_NUMBER}"
                    }
                }

            }
        }
        stage ("actualizacion de kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh 'kubectl -n lab-mlabbe set image deployments/backend-nest-test-MLM backend-nest-test-MLM=${dockerimagePrefix}/backend-nest-test-MLM:${BUILD_NUMBER}'
                }
            }
        }
    }
}
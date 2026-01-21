pipeline{
    agent any // decided which node to run

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "iamswapnil98/itkannadigaru-blogpost:${GIT_COMMIT}"
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "itkannadigaru-cluster"
        NAMESPACE = "microdegree"
    }

    stages{
        stage('git-checkout'){
            steps{
                git url: 'https://github.com/swap1998-nam/ITKannadigaru-Java-based-app.git', branch: 'prod'
            }
            
        }

        stage('Compile'){
            steps{
                sh '''
                    mvn compile
                '''
            }
        }
        stage('packaging'){
            steps{
                sh '''
                    mvn clean package
                '''
            }
        }
        stage('docker-build'){
            steps{
                sh '''
                    printenv
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }
        // stage('Docker-testing'){
        //     steps{
        //         sh '''
        //             docker kill itkannadigaru-blogpost-test
        //             docker rm itkannadigaru-blogpost-test
        //             docker run -it -d --name itkannadigaru-blogpost-test -p 9000:8080 ${IMAGE_NAME}
        //         '''
        //     }
        // }   

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Login to Docker Hub
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }  

        stage('Push to dockerhub'){
            steps{
                sh '''
                    docker push ${IMAGE_NAME}
                '''
            }
        }

        stage('update the k8 cluster'){
            steps{
                script{
                   sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"     
                }
            }
        }

        stage('Deploy to EKS cluster'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'itkannadigaru-cluster', contextName: '', credentialsId: 'kube', namespace: 'itkannadigaru', restrictKubeConfigAccess: false, serverUrl: 'https://CCEC902FBD545A5B11F9A59D4867CF6A.yl4.ap-south-1.eks.amazonaws.com'){
                    sh " sed -i 's|replace|${IMAGE_NAME}|g' deployment.yml "
                    sh " kubectl apply -f deployment.yml -n ${NAMESPACE}"
                }
            }
        }
        stage('verify'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'itkannadigaru-cluster', contextName: '', credentialsId: 'kube', namespace: 'itkannadigaru', restrictKubeConfigAccess: false, serverUrl: 'https://CCEC902FBD545A5B11F9A59D4867CF6A.yl4.ap-south-1.eks.amazonaws.com'){
                    sh " kubectl get pods -n ${NAMESPACE}"
                    sh " kubectl get svc -n ${NAMESPACE}"
                }
            }
        }
    }
}

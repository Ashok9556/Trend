echo pipeline { > Jenkinsfile
echo     agent any >> Jenkinsfile
echo     environment { >> Jenkinsfile
echo         DOCKERHUB_CREDS = credentials('dockerhub-creds') >> Jenkinsfile
echo         IMAGE_NAME      = "ashok9951/trend-app" >> Jenkinsfile
echo         IMAGE_TAG       = "${env.BUILD_NUMBER}" >> Jenkinsfile
echo         AWS_REGION      = "us-east-1" >> Jenkinsfile
echo         EKS_CLUSTER     = "trend-devops-eks" >> Jenkinsfile
echo     } >> Jenkinsfile
echo     stages { >> Jenkinsfile
echo         stage('Checkout') { >> Jenkinsfile
echo             steps { >> Jenkinsfile
echo                 checkout scm >> Jenkinsfile
echo             } >> Jenkinsfile
echo         } >> Jenkinsfile
echo         stage('Build Docker Image') { >> Jenkinsfile
echo             steps { >> Jenkinsfile
echo                 sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ." >> Jenkinsfile
echo             } >> Jenkinsfile
echo         } >> Jenkinsfile
echo         stage('Push to DockerHub') { >> Jenkinsfile
echo             steps { >> Jenkinsfile
echo                 sh "echo \$DOCKERHUB_CREDS_PSW | docker login -u \$DOCKERHUB_CREDS_USR --password-stdin" >> Jenkinsfile
echo                 sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}" >> Jenkinsfile
echo                 sh "docker push ${IMAGE_NAME}:latest" >> Jenkinsfile
echo                 sh "docker logout" >> Jenkinsfile
echo             } >> Jenkinsfile
echo         } >> Jenkinsfile
echo         stage('Configure kubectl') { >> Jenkinsfile
echo             steps { >> Jenkinsfile
echo                 sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}" >> Jenkinsfile
echo                 sh "kubectl get nodes" >> Jenkinsfile
echo             } >> Jenkinsfile
echo         } >> Jenkinsfile
echo         stage('Deploy to K8s') { >> Jenkinsfile
echo             steps { >> Jenkinsfile
echo                 sh "kubectl apply -f k8s/deployment.yaml" >> Jenkinsfile
echo                 sh "kubectl apply -f k8s/service.yaml" >> Jenkinsfile
echo                 sh "kubectl rollout status deployment/trend-app --timeout=180s" >> Jenkinsfile
echo             } >> Jenkinsfile
echo         } >> Jenkinsfile
echo     } >> Jenkinsfile
echo } >> Jenkinsfile
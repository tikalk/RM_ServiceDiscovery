node('linux-host-slave') {
    stages {
        stage('Build & Push Image') {            
            environment {
                AWS_ECR_LOGIN='true'
                AWS_ECR_LOGIN_REGISTRY_IDS='329054710135'
                AWS_DEFAULT_REGION='eu-west-2'
                AWS_REGION='eu-west-2'
            }
            steps {
                sh "\$(\${HOME}/.local/bin/aws ecr get-login --no-include-email &> /dev/null)"
                sh "cp \${HOME}/.docker/config.json \${HOME}/.dockercfg"          
                def mvnHome = tool 'M3'
                sh "${mvnHome}/bin/mvn clean package docker:build"
                withDockerRegistry([credentialsId: 'ecr:eu-west-2:k8s-aws-ecr', url: "${registry}"]) {
                    git 'https://github.com/tikalk/RM_ServiceDiscovery.git'
                    def image = docker.build("329054710135.dkr.ecr.eu-west-2.amazonaws.com/discovery_service:${BUILD_NUMBER}")                
                    image.push()
                }
           }
        }
        
        stage ('Deploy to K8S') {
        sh(script: """
        sed -i 's/BUILDNUMBER/${BUILD_NUMBER}/g' deployment.yaml
        ./kubectl apply -f deployment.yaml --kubeconfig=\$(pwd)/kconfig --namespace fuze
        ./kubectl get pods --namespace fuze -l app=RM_ServiceDiscovery &> /dev/null
        """, returnStatus: false, returnStdout: false)
      }

    }
}

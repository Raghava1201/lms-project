pipeline{
    agent{
        node{
            label 'slave-prod'
        }
    }
     environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        EKS_CLUSTER_NAME = 'lms-eks-cluster'
        KUBECONFIG = credentials('aws-credentials')
     }
     stages {
        stage('slack-notification') {
            steps {
                slackSend channel: 'devops',
                    color: '439FE0',
                    message: "started ${JOB_NAME} ${BUILD_NUMBER} (<${BUILD_URL}|Open>)",
                    teamDomain: 'raghava-world',
                    tokenCredentialId: 'slack3',
                    username: 'jenkins'
            }
        }
        stage('Approval') {
            steps {
                emailext subject: "Deployment Approval for lms service",
                    body: "<a href='${JENKINS_URL}/job/${JOB_NAME}/${BUILD_NUMBER}/input'>click to approve</a>",
                    to: 'machirajuraghavarao@gmail.com',
                    mimeType: 'text/html',
                    attachLog: true
                script {
                    def Delay = input id: 'Deploy',
                        message: sh(script: '''echo "You are DEPLOYING -->$deployBranch<-- IN PRODUCTION"''', returnStdout: true).trim(),
                        submitter: 'userID1, userID2',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition',
                                choices: ['no', 'yes'].join('\n'),
                                name: 'input',
                                description: 'Please Select "YES" to Build or "NO" to Abort']
                        ]
                    echo "The answer is: ${Delay}"
                    if ("${Delay}" == "yes") {
                        sh '''echo "Deploying in prod"'''
                    } else {
                        sh """
                        echo "exiting not production ready branch"
                        exit 1
                        """
                    }
                }
            }
        }
        stage('Configure AWS CLI') {
            steps {
                sh 'aws eks update-kubeconfig --name lms-eks-cluster --region us-east-1'
             }
        }
        stage('minikube'){
           steps{
               sh 'pwd'
               sh 'minikube start'
           }
       }
    stage('Deploy to eks'){
        steps{
            dir('k8s'){
               sh 'ls -al /home/ubuntu/workspace/lms-project_prod/k8s/'
               sh 'kubectl apply -f pg-secret.yml --validate=false'
               sh 'kubectl apply -f pg-deployment.yml'
               sh 'kubectl apply -f pg-service.yml'
               sh 'kubectl apply -f be-configmap.yml'
               sh 'kubectl apply -f be-deployment.yml'
               sh 'kubectl apply -f be-service.yml'
               sh 'kubectl apply -f fe-deployment.yml'
               sh 'kubectl apply -f fe-service.yml'
            }
         }
      }
   }
}    

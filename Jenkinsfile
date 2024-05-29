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
        stage('scanning-backend') {
            steps {
               sh 'cd api && docker run --rm -e SONAR_HOST_URL="http://35.175.129.95:9000" -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=lms-project" -e SONAR_TOKEN="sqp_25e6c9448bb34bbe2d97888de21fba6eee5d9d5e" -v ".:/usr/src" sonarsource/sonar-scanner-cli'
          }
        }
        stage('Approval') {
            steps {
                emailext subject: "Deployment Approval for $deployBranch branch and $deployService service",
                    body: "<a href='${JENKINS_URL}/job/${JOB_NAME}/${BUILD_NUMBER}/input'>click to approve</a>",
                    to: 'machirajuraghavarao@gmail .com',
                    mimeType: 'text/html',
                    attachLog: true
                script {
                    def Delay = input id: 'Deploy',
                        message: sh(script: '''echo "You are DEPLOYING -->$deployBranch<-- IN PRODUCTION"''', returnStdout: true).trim(),
                        submitter: 'uaserID1, userID2',
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
     }
}   

node {
    
    stage('Clean') {
        sh 'rm *.zip || true'

    }


    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/neilcar/lambda_sample.git']]])
    }

    stage('Build Initial ZIP') {
        zip dir: '', glob: 'main.py', zipFile: 'lambda.zip'

    }

    stage('Scan function') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'        
            sh './twistcli serverless scan --details -address https://$TL_CONSOLE -u $TL_USER -p $TL_PASS lambda.zip'
        } 
    }
    
    stage('Embed Serverless Defender') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
            sh './twistcli serverless embed --address https://$TL_CONSOLE \
          --console-host $TL_CONSOLE --handler main.handler --function neil_test \
          --runtime python3.6  -u $TL_USER -p $TL_PASS lambda.zip'
        }
    }
    stage('Publish Function') {
        withAWS(credentialsId: 'AWS_acct') {
            deployLambda([useInstanceCredentials: true, alias: '', artifactLocation: 'twistlock_lambda.zip', awsRegion: 'us-east-1', deadLetterQueueArn: '', description: 'Neil is building in Jenkins', environmentConfiguration: [kmsArn: ''], functionName: 'neilcar_jenkins_test', handler: 'main.handler', memorySize: '128', role: 'arn:aws:iam::aws:policy/AWSLambdaFullAccess', runtime: 'python3.6', securityGroups: '', subnets: '', timeout: '30', updateMode: 'full', useInstanceCredentials: true])
    }

        

    }

}

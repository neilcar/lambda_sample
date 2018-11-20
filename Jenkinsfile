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

    stage('Embed Serverless Defender') {
        sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
        sh 'sudo chmod a+x ./twistcli'
        sh './twistcli serverless embed --address https://$TL_CONSOLE \
       --console-host $TL_CONSOLE --handler main.handler --function neil_test \
       --runtime python3.6  -u $TL_USER -p $TL_PASS lambda.zip'

    }
    stage('Publish Function') {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_acct', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
            deployLambda([alias: '', artifactLocation: 'twistlock_lambda.zip', awsAccessKeyId: '${AWS_ACCESS_KEY_ID}', awsRegion: 'us-east-1', awsSecretKey: '${AWS_SECRET_ACCESS_KEY}', deadLetterQueueArn: '', description: 'Neil is building in Jenkins', environmentConfiguration: [kmsArn: ''], functionName: 'neilcar_jenkins_test', handler: 'main.handler', memorySize: '128', role: 'arn:aws:iam::113505086193:role/lambda_basic_execution', runtime: 'python3.6', securityGroups: '', subnets: '', timeout: '30', updateMode: 'full'])
}

        

    }

}

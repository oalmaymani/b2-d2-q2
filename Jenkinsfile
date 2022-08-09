pipeline {
    agent any

    environment {

        AWS_ACCESS_KEY_ID     = credentials('osaid-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('osaid-aws-secret-access-key.')

        AWS_S3_BUCKET         = "osaid-belt2d2-artifacts-123456"

        ARTIFACT_NAME         = "spring-boot-rest-services-0.0.1-SNAPSHOT.jar"
        AWS_EB_APP_NAME       = "osaid-Belt2D2-artifacts-123456"
        AWS_EB_APP_VERSION    = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT    = "Osaidbelt2d2artifacts123456-env"

        SONAR_IP              = "52.23.193.18"
        SONAR_TOKEN           = "sqp_8215f91480971e55fb7f4c66af82e8ce911a2879"

    }

    stages {
        stage('Validate') {
            steps {                
                sh "mvn validate"
                sh "mvn clean"
            }
        }

         stage('Build') {
            steps {                
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {                
                sh "mvn test"
            }

            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=onsite-Osaid-B2D2 \
                    -Dsonar.host.url=http://$SONAR_IP \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Package') {
            steps {                
                sh "mvn package"
            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false                   
                }
            }
        }

        stage('Publish artefacts to S3 Bucket') {
            steps {
                sh "aws configure set region eu-west-3"
                sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"                
            }
        }

        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
        }        
    }
}
    

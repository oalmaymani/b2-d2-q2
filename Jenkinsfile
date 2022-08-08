pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('osaid-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('osaid-aws-secret-access-key.')
        ARTIFACT_NAME = 'hello-world.war'
        AWS_S3_BUCKET = 'osaid-belt2d2-artifacts-123456'
        AWS_EB_APP_NAME = 'osaid-Belt2D2-artifacts-123456'
        AWS_EB_ENVIRONMENT = 'Osaidbelt2d2artifacts123456-env'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
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

        stage('Package') {
            steps {
                
                sh "mvn package"

            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false
                    sh 'aws configure set region us-east-1'
                    sh 'aws s3 cp ./target/**.war s3://$AWS_S3_BUCKET/$hello-world.jar'
                }
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
    

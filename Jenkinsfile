def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {

    agent any

    tools {
        maven "MAVEN3.9.9"
        jdk "JDK17"
    }

    environment {
        // registryCredential = '<registry_type:region:aws_credentials_ID>'
        registryCredential = 'ecr:us-east-1:awscreds'
        // imageName = '<URI>'
        imageName = "266735847828.dkr.ecr.us-east-1.amazonaws.com/cicdproject"
        // registry = 'https://<URI without image_name>'
        registry = "https://266735847828.dkr.ecr.us-east-1.amazonaws.com"

        // Create ECS cluster containing service. Service is a task which will run your container fetching image from ECR
        // cluster = '<cluster_name>'
        cluster = 'cicdprojectcluster'
        // service = '<service_name>'
        service = 'cicdprojectsvc'
    }

    stages {

        stage('Fetch code') {
            steps{
                git branch: 'main', 
                url: 'https://github.com/notirawap/CICDproject.git'
            }
        }
 
        stage('Build') {
            steps{
                sh 'mvn clean install -DskipTests'
            }
            post {
                success{
                    echo "************************ Archiving Artifact ************************"
                    archiveArtifacts artifacts: '**/*.war' 
                }
            }
        }

        stage('Unit Test') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Code analysis') {
            environment {
                scannerHome = tool 'SONAR'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=CICDproject \
                        -Dsonar.projectName=CICDproject \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    // Run docker plugin with build function: docker.build("URI"+":<image_tag>", "<Dockerfile path from the Github source code>")
                    dockerImage = docker.build(imageName + ":$BUILD_NUMBER", ".")
                }
            }
    
        }

        stage('Upload App Image') {
            steps{
                script {
                    // Run docker plugin with withRegistry function: withRegistry("https://<URI without image_name>'", "<registry_name:region:credentials_ID>") for Docker plugin to login to registry
                    docker.withRegistry(registry, registryCredential) {
                        // Push Docker image with the tags
                        dockerImage.push("$BUILD_NUMBER") // tag1
                        dockerImage.push('latest') // tag2
                    }
                }
            }
        }

        stage('Remove Container Images'){
            steps{
                sh 'docker rmi -f $(docker images -a -q)'
            }
        }

        stage('Deploy to ECS'){
            steps{
                // withAWS(credentials: '<aws_credentials_ID>', region: '<region>')
                withAWS(credentials:'awscreds', region: 'us-east-1'){
                    // AWS CLI to remove the old container with the old image tag, fetch new image with the latest tags, and run it on the ECS cluster.
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                }
            }
        }
    }

    post {
		always {
				echo 'Slack Notifications'
				slackSend channel: '#devopscicd',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
		}
    }
}

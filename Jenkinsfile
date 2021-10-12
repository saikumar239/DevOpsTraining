pipeline {
    agent any
	environment {
		PROJECT_ID = 'leafy-market-327511'
                CLUSTER_NAME = 'devops'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'Kubernetes'		
	}
    stages {
        stage('Build') {
            steps {

                sh 'mvn -B -DskipTests clean package'
                sh 'mvn deploy -s settings.xml'
            }
        }//end build
	    
        stage('Test') {
            steps {
                sh 'mvn test'
                junit 'target/surefire-reports/*.xml'
				}
			}//end of test
	    
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh 'mvn sonar:sonar' 
              }
            }
        }//end of sonar
	    
	stage("Sonar Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }//end of Sonar Quality gate
	    
	stage('Docker Build') {
	    steps {
		withDockerRegistry([ credentialsId: "Artifactory", url: "https://devopscicd.jfrog.io/" ]) {
		sh 'docker build -t "devops:${BUILD_NUMBER}" .'
		sh 'docker tag "devops:${BUILD_NUMBER}" devopscicd.jfrog.io/default-docker-local/"devops:${BUILD_NUMBER}"'
		}
	    }
	}//end of Docker Build
	    
	stage('Docker Push') {
	    steps {
		withDockerRegistry([ credentialsId: "Artifactory", url: "https://devopscicd.jfrog.io/" ]) {
		sh 'docker push devopscicd.jfrog.io/default-docker-local/"devops:${BUILD_NUMBER}"'
		}
	     }
	}//end of Docker Push	
	stage('Deploy to GKE K8s') {
		    steps{
			script {
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_NUMBER}/g' serviceLB.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		       }
	            }
		}//closed Deploy to GKE K8
      }//end stages
    }//end pipeline

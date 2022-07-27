pipeline {
    agent any
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'geometric-hull-355406'
                CLUSTER_NAME = 'k8s-cluster'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'kubernetes'
		
                
						
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				   myimage = docker.build("gcr.io/$PROJECT_ID/image:${env.BUILD_ID}")
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    docker.withRegistry('gcr.io/geometric-hull-355406', 'gcr:kubernetes')
				    echo "Push Docker Image"
				    myimage.push("${env.BUILD_ID}")
		        }
			
			}
		}	
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' serviceLB.yaml"
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Start deployment of deployment.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
    }
}

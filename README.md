#Begining my learn path of Jenkins

**Metadata for ubuntu quick instalation** 
#!/bin/bash
sudo apt update
sudo apt install openjdk-8-jdk -y 
sudo apt install maven -y 
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y 
sudo systemctl start jenkins 
sudo systemctl enable jenkins 

- Open jenkins console, on Global tool configuration we can configure JDK path /usr/lib/jvm/java-1.8.0-openjdk-amd64 and maven 3 


- Example of pipeline

pipeline {
	agent any
	stages{
	  stage('Fetch code'){
	    steps { 
          git branch: 'main', url: 'https://github.com/Vinicius-Sa/Jenkins.git'
	    }
	  }
	  stage('build'){
	    steps {
	      sh 'sudo apt install maven'
	    }
	  }
	  stage('Test'){
	    steps{
	      sh 'sudo touch /test'
	    }
	  }
	}
}


Getting **"sudo: a terminal is required to read the password"** aded Jenkins user to sudoers with no passwd required to solve this **jenkins ALL= NOPASSWD: ALL** 

![Captura de tela 2023-03-05 210747](https://user-images.githubusercontent.com/95035624/222995291-cee76488-6494-47c3-8942-0cb2e941bdb6.png)



- Using environment variables

BUILD_ID
The current build ID, identical to BUILD_NUMBER for builds created in Jenkins versions 1.597+

BUILD_NUMBER
The current build number, such as "153"

BUILD_TAG
String of jenkins-${JOB_NAME}-${BUILD_NUMBER}. Convenient to put into a resource file, a jar file, etc for easier identification

Example of how to use these enrionment variables 
stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}" }}}


- Build 

Jenkins has a number of plugins for invoking practically any build tool in general use, but this example will simply invoke make from a shell step (sh). The sh step assumes the system is Unix/Linux-based, for Windows-based systems the bat could be used instead.

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' 
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
            }
        }
    }
}

- Test 

In the example below, if tests fail, the Pipeline is marked "unstable", as denoted by a yellow ball in the web UI. Based on the recorded test reports, Jenkins can also provide historical trend analysis and visualization

pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true' 
                junit '**/target/*.xml' 
            }
        }
    }
}

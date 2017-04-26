pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr:'5'))
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }  
        
        stage('build') {
            
            when {
                branch 'dev'
            }
            
            steps {
                withMaven(
                    maven: 'default', // Tools declared in the Jenkins "Global Tool Configuration"
                    jdk: 'default'){
                    sh 'mvn -U clean deploy'
                } // withMaven will discover the generated Maven artifacts, JUnit reports and FindBugs reports
            }
        }
    } 
    
    post {
        success {
			sh '$JENKINS_HOME/scripts/updateGithubIssue.sh'
        }
        failure {
            mail(from: "jenkins@boxy.cismet.de", 
				to: "pascal@cismet.de", 
				subject: "Build failed in Jenkins: ${currentBuild.fullDisplayName}",
                body: """FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':
                Check console output at '${env.BUILD_URL}'""")
        }
        unstable {
            mail(from: "jenkins@boxy.cismet.de", 
				to: "pascal@cismet.de", 
				subject: "Jenkins build became unstable: ${currentBuild.fullDisplayName}",
                body: """<p>UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':
				Check console output at '${env.BUILD_URL}'""")
        }
        changed {
            mail(from: "jenkins@boxy.cismet.de", 
                to: "dev@cismet.de", 
                subject: "Jenkins build passed: ${currentBuild.fullDisplayName}",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':
				Check console output at '${env.BUILD_URL}'""")
        }
    }
}
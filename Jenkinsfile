/**
 * Pipelines are a series of steps that allow you to orchestrate the work required to build, 
 * test and deploy applications. Pipelines are defined in a file called Jenkinsfile 
 * that is stored in the root of your project’s source repository.
 **/
pipeline {

    agent { 
        label 'master' 
    }
       
    tools {
        maven "default"
        jdk "default"
    }
    

    environment  {
        FOO = 'BAR'
    }
    
    // The options directive is for configuration that applies to the whole job.
    options {
        // For example, we'd like to make sure we only keep 10 builds at a time, so
        // we don't fill up our storage!
        buildDiscarder(logRotator(numToKeepStr:'5'))
    
        // And we'd really like to be sure that this build doesn't hang forever, so
        // let's time it out after an hour.
        timeout(time: 60, unit: 'MINUTES')
        
        disableConcurrentBuilds()
        
        //pipelineTriggers([githubPush()])
    }

    stages {

        stage('prepare') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "JAVA_HOME = ${JAVA_HOME}"
                '''
            }
        }
        
        
        stage('checkout') {
            steps {
                // git url: 'https://github.com/p-a-s-c-a-l/cids-server-rest-types-releasetest.git'
                // git branch: 'dev', url: 'git@github.com:p-a-s-c-a-l/cids-server-rest-types-releasetest.git'
                checkout scm
            }
        }  
        
        stage('build') {
            
            when {
                // Only run if the branch matches this Ant-style pattern
                branch 'dev'
            }
            
            /**
             * Think of a step as like a single command that does a single action. 
             * When a step succeeds it moves onto the next step. 
             * When a step fails to execute correctly the Pipeline will fail.
             **/
            steps {
                withMaven(
                    maven: 'default', // Maven installation declared in the Jenkins "Global Tool Configuration"
                    //mavenSettingsConfig: 'my-maven-settings', // Maven settings.xml file defined with the Jenkins Config File Provider Plugin
                    //mavenLocalRepo: '.repository'
                    jdk: 'default'){
                    // Run the maven build
                    sh 'mvn -U clean deploy'
                } // withMaven will discover the generated Maven artifacts, JUnit reports and FindBugs reports
            }
        }
    } 
    
    post {
        always {
            echo 'This will always run'
            sh '$JENKINS_HOME/scripts/updateGithubIssue.sh'
        }
        
        success {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/**/*.xml'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
            
            mail(from: "jenkins@boxy.cismet.de", 
                to: "pascal@cismet.de", 
                subject: "${currentBuild.fullDisplayName} passed.",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""")
            
        }
        failure {
            echo 'This will run only if failed'
            mail to: "pascal@cismet.de", subject: "${currentBuild.fullDisplayName} Build Failed",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
            mail to: "pascal@cismet.de", subject: "${currentBuild.fullDisplayName} Build Unstable",
                body: """<p>UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, the Pipeline was previously failing but is now successful'
        }
    }
}
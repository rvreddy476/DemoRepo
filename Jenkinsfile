/*
    Name:       myaccount-ui
    Template:   Jenkinsfile Template for Node.js + Angular
    Version:    v1.2
*/

// OpenShift project names
def aspire_code = '2f82'
def openshift_project_dev = "${aspire_code}-d"
def openshift_project_val = "${aspire_code}-v"
def openshift_project_preprod = "${aspire_code}-q"
def openshift_project_prod = "${aspire_code}-p"

// designated branch names
def branch_dev = 'dev'
def branch_val = 'val'
def branch_preprod = 'pre-prod'
def branch_prod = 'master'

// the name of the microservice in OpenShift
def microservice = 'myaccount-ui'
def buildName = 'myaccount-ui'
def buildRuntimeName = 'myaccount-ui-runtime'
def deploymentName = 'myaccount-ui-runtime'

pipeline {
    agent {
        node {
            label 'nodejs-12' 
        }
    }
   
    stages {
        stage ('checkout git') {
            steps{
                 checkout scm
                  sh 'git branch -a'
            }
        }
        stage ('install modules') {
            when {
                not {
                    anyOf {
                        branch branch_dev;
                        branch branch_val;
                        branch branch_preprod;
                        branch branch_prod;
                    }
                }
            }
            steps{
                   configFileProvider([configFile(fileId: '5f007d03-0ea8-4109-9b7c-e2cd667431e0', variable: 'npmrc')]) {
                      sh "cp $npmrc ./"
                      sh '''
                            cp $npmrc ~/.npmrc
                            npm install --verbose
                         '''
            }
         }
        }
        stage ('build Angular') {
            when {
                not {
                    anyOf {
                        branch branch_dev;
                        branch branch_val;
                        branch branch_preprod;
                        branch branch_prod
                    }
                }
            }
            steps{
                sh '$(npm bin)/ng build --prod'
            }
        }
       /*stage ('code quality'){
            when {
                not {
                    anyOf {
                        branch branch_dev;
                        branch branch_val;
                        branch branch_preprod;
                        branch branch_prod;
                    }
                }
            }
            steps{
                sh '$(npm bin)/ng lint'
            }
        }*/
        
        
        stage ('unit tests') { 
            when {
                not {
                    anyOf {
                        branch branch_dev;
                        branch branch_val;
                        branch branch_preprod;
                        branch branch_prod
                    }
                }
            }                 
                steps{                    
                             sh 'npm run test:unit:ci'  
                     }
             post {
                    always {          
                           junit 'reports/*.xml'
                            }
                }
            }

        stage ('Build on OpenShift in Dev') {
            when {
                branch branch_dev
            }
            steps{
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_dev) {
                            def bc = openshift.selector("bc", buildName).startBuild("--wait")
                            def bcRuntime = openshift.selector("bc", buildRuntimeName).startBuild("--wait")
                        }
                    }
                }
            }
        }
        stage('Deploy to Dev') {
            when {
                branch branch_dev
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_dev) {
                            def dc = openshift.selector("dc", deploymentName)
                            dc.rollout().latest()
                            dc.rollout().status()
                        }
                    }
                }
            }
        }
        stage('Build on OpenShift in Val') {
            when {
                branch branch_val
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_val) {
                            def bc = openshift.selector("bc", buildName).startBuild("--wait")
                            def bcRuntime = openshift.selector("bc", buildRuntimeName).startBuild("--wait")
                        }
                    }
                }
            }
        }
        stage('Deploy to OpenShift in Val') {
            when {
                branch branch_val
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_val) {
                            def dc = openshift.selector("dc", deploymentName)
                            dc.rollout().latest()
                            dc.rollout().status()
                        }
                    }
                }
            }
        }
         stage('Build on OpenShift in Pre-Prod') {
            when {
                branch branch_preprod
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_preprod) {
                            def bc = openshift.selector("bc", buildName).startBuild("--wait")
                            def bcRuntime = openshift.selector("bc", buildRuntimeName).startBuild("--wait")
                        }
                    }
                }
            }
        }
        stage('Deploy to OpenShift in Pre-Prod') {
            when {
                branch branch_preprod
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_preprod) {
                            def dc = openshift.selector("dc", deploymentName)
                            dc.rollout().latest()
                            dc.rollout().status()
                        }
                    }
                }
            }
        }
        stage('Build on OpenShift in Prod') {
            when {
                branch branch_prod
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_prod) {
                            def bc = openshift.selector("bc", buildName).startBuild("--wait")
                            def bcRuntime = openshift.selector("bc", buildRuntimeName).startBuild("--wait")
                        }
                    }
                }
            }
        }
        stage('Promote to Prod?') {
            when {
                branch branch_prod
            }
            steps {
                mail to: 'abhishek.reddy@airbus.com',
	            cc : 'dl-iam-selfservice@airbus.com',
                subject: "INPUT: Build ${env.JOB_NAME}", 
                body: "Awaiting for your input ${env.JOB_NAME} build no: ${env.BUILD_NUMBER} .Click below to promote to production\n${env.JENKINS_URL}job/${env.JOB_NAME}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
                timeout(time: 60, unit: 'MINUTES'){
                    input message: "Promote to Production?", ok: "Promote"
                }
            }
        }

        stage('Deploy to OpenShift in Prod') {
            when {
                branch branch_prod
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(openshift_project_prod) {
                            def dc = openshift.selector("dc", deploymentName)
                            dc.rollout().latest()
                            dc.rollout().status()
                        }
                    }
                }
            }
        }
        // in case your package is a library uncomment the following to publish to Artifactory
        /*
        stage ('Publish to Artifactory'){
            when {
                branch branch_prod
            }
            steps {
                sh 'npm publish'
            }
        }
        */
    }
    // if you have unit tests defined uncomment the following to have them show up in the Jenkins UI
    // post {
    //    always {
    //        junit 'build/reports/**/*.xml'
    //    }
    // }
      post {
        failure {
            mail to: 'dl-iam-selfservice@airbus.com',            
                subject: "FAILED: Build ${env.JOB_NAME}", 
                body: "Build failed ${env.JOB_NAME} build no: ${env.BUILD_NUMBER}.\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
	        }
	
	 aborted{
	        mail to: 'dl-iam-selfservice@airbus.com',                
                subject: "ABORTED: Build ${env.JOB_NAME}", 
                body: "Build was aborted ${env.JOB_NAME} build no: ${env.BUILD_NUMBER}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
                }
	}
}
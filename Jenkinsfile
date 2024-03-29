/*
 * Normal Jenkinsfile that will build and do Policy and SCA scans
 */

pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'Verademo'      // App Name in the Veracode Platform
    }

    // this is optional on Linux, if jenkins does not have access to your locally installed docker
    //tools {
    // these match up with 'Manage Jenkins -> Global Tool Config'
    //'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker-latest' 
    //}

    options {
        // only keep the last x build logs and artifacts (for space saving)
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    }

    // First Stage is to Verify that the Environmental Variables do check out and Git Repo is aligned with the scope
    
    stages{
        stage ('environment verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }
        // This Next Stage is a 'Build' stage where pom.xml file for Maven to compile the application using pom.xml within app/ directory
        stage ('build') {
            steps {
                withMaven(maven:'maven-3') {
                    script {
                        if(isUnix() == true) {
                            sh 'mvn clean package -f app/pom.xml'
                        }
                        else {
                            bat 'mvn clean package -f app/pom.xml'
                        }
                    }
                }
            }
        }
        /*
        stage ('Veracode scan') {
            steps {
                script {
                    if(isUnix() == true) {
                        env.HOST_OS = 'Unix'
                    }
                    else {
                        env.HOST_OS = 'Windows'
                    }
                }

                echo 'Veracode scanning' */
                withCredentials([ usernamePassword (
                        credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                    // Commented out Static Policy Scan 
                    /*
                    veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}-${env.HOST_OS}", uploadExcludesPattern: '', uploadIncludesPattern: 'target/verademo.war', vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"
                    */
                    // wait for scan to complete (timeout: x)
                    //veracode applicationName: '${VERACODE_APP_NAME}'', criticality: 'VeryHigh', debug: true, timeout: 20, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'target/verademo.war', vid: '${VERACODE_API_ID}', vkey: '${VERACODE_API_KEY}'
                }
            }
        }
        // This Stage marks the beginning of SCA for Veracode for 3rd party analysis.
        stage ('Veracode SCA') {
            steps {
                echo 'Veracode SCA'
                withCredentials([ string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                    withMaven(maven:'maven-3') {
                        script {
                            sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan . --update-advisor --allow-dirty"
                            }
                        }
                    }
                }
            }
        }
        // Next Stage of the Pipeline, we have the Pipeline Scan that is focused for developers to fix flaws faster, scan faster under same scan engine. 
        stage ('Veracode pipeline scan') {
            steps {
                echo 'Veracode Pipeline scanning'
                withCredentials([ usernamePassword (
                        credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                    script {

                        // this try-catch block will show the flaws in the Jenkins log, and yet not
                        // fail the build due to any flaws reported in the pipeline scan
                        // alternately, you could add --fail_on_severity '', but that would not show the
                        // flaws in the Jenkins log

                        // issue_details true: add flaw details to the results.json file
                        try {
                            if(isUnix() == true) {
                                sh """
                                        curl -sO https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip
                                        unzip pipeline-scan-LATEST.zip pipeline-scan.jar
                                        java -jar pipeline-scan.jar --veracode_api_id '${VERACODE_API_ID}' \
                                            --veracode_api_key '${VERACODE_API_KEY}' \
                                            --file target/verademo.war --issue_details true --verbose true --fail_on_severity="Very High, High" --fail_on_criteria="79,80,113"
                                        """
                            }
                            else {
                                powershell """
                                            curl  https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip -o pipeline-scan.zip
                                            Expand-Archive -Path pipeline-scan.zip -DestinationPath veracode_scanner
                                            java -jar veracode_scanner\\pipeline-scan.jar --veracode_api_id '${VERACODE_API_ID}' \
                                            --veracode_api_key '${VERACODE_API_KEY}' \
                                            --file target/verademo.war --issue_details true --verbose true --fail_on_severity="Very High, High" --fail_on_criteria="79,80,113"
                                            """
                            }
                        } catch (err) {
                            echo 'Pipeline err: ' + err
                        }
                    }
                }
            }
        }
    }
}

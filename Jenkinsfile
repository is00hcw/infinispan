#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                script {
                    sh returnStdout: true, script: 'cleanup.sh'
                }
            }
        }
        
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                configFileProvider([configFile(fileId: 'maven-settings-with-deploy-snapshot', variable: 'MAVEN_SETTINGS')]) {
                    script {
                        def mvnHome = tool 'Maven'
                        sh "${mvnHome}/bin/mvn clean install -s $MAVEN_SETTINGS -Dmaven.test.failure.ignore=true"
                        junit testDataPublishers: [[$class: 'ClaimTestDataPublisher']], testResults: '**/target/*-reports/*.xml'
                    }
                }
            }
        }

        stage('X-Site tests') {
            when {
                branch 'master'
            }
            steps {
                configFileProvider([configFile(fileId: 'maven-settings-with-deploy-snapshot', variable: 'MAVEN_SETTINGS')]) {
                    script {
                        def mvnHome = tool 'Maven'
                        sh "${mvnHome}/bin/mvn clean install -s $MAVEN_SETTINGS -pl core -Ptest-xsite -Dinfinispan.module-suffix=xsite -Dmaven.test.failure.ignore=true"
                        junit testDataPublishers: [[$class: 'ClaimTestDataPublisher']], testResults: '**/target/*-reports/*.xml'
                    }
                }
            }
        }

        stage('Stress tests') {
            when {
                branch 'master'
            }
            steps {
                configFileProvider([configFile(fileId: 'maven-settings-with-deploy-snapshot', variable: 'MAVEN_SETTINGS')]) {
                    script {
                        def mvnHome = tool 'Maven'
                        sh "${mvnHome}/bin/mvn clean install -s $MAVEN_SETTINGS -Ptest-CI,mongodb,nonParallel -Dinfinispan.test.jta.tm=jbosstm -DdefaultTestGroup=stress -DdefaultExcludedTestGroup=unstable,unstable_xsite -Dinfinispan.test.parallel.threads=1  -Dmaven.test.failure.ignore=true"
                        junit testDataPublishers: [[$class: 'ClaimTestDataPublisher']], testResults: '**/target/*-reports/*.xml'
                    }
                }
            }
        }
        
        stage('Deploy SNAPSHOT') {
            when {
                branch 'master'
            }
            steps {
                configFileProvider([configFile(fileId: 'maven-settings-with-deploy-snapshot', variable: 'MAVEN_SETTINGS')]) {
                    script {
                        def mvnHome = tool 'Maven'
                        sh "${mvnHome}/bin/mvn deploy -s $MAVEN_SETTINGS -DskipTests"
                    }
                }
            }
        }
    }
}

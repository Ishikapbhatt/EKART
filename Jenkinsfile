pipeline {
    agent any

    tools {
        maven 'maven'  // Name of Maven tool in Jenkins
        jdk 'jdk17'    // Name of JDK tool in Jenkins
    }

    environment {
        SCANNER_HOME = tool 'sonar'  // SonarQube scanner
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ishikapbhatt/EKART.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=.
                    """
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                // Use Jenkins credentials
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds', 
                    usernameVariable: 'NEXUS_USER', 
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    // Use Managed Maven settings file
                    configFileProvider([configFile(
                        fileId: 'maven-settings',  // ID of Maven settings.xml in Jenkins
                        variable: 'MAVEN_SETTINGS'
                    )]) {
                        sh """
                            mvn clean deploy -DskipTests -s $MAVEN_SETTINGS
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished!"
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}

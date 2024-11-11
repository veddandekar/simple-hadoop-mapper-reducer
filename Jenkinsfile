pipeline {
    agent any 
    tools {
        maven "Maven 3.9.2"
    }

    stages {
        stage('Static Code Analysis') {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh 'mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=HadoopMapReducer \
                  -Dsonar.projectName="HadoopMapReducer" \
                  -Dsonar.host.url=http://34.45.111.212 \
                  -Dsonar.token=sqp_8587c9ff58b04e0550fdc9dc4dd077e6e21ed59c'
                }
            }
        }
        stage("Quality Gate") {
            steps {
                  timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                  }
            }
        }


        stage('Build Hadoop Job') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy and generate result') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh 'cat data.txt'
                sshagent(['hadoop-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no data.txt root@34.139.13.169:/tmp/input.txt
                    """
                }
                sshagent(['hadoop-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/wordcount-1.0-SNAPSHOT.jar root@34.139.13.169:/tmp/
                    """
                }
                sshagent(['hadoop-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no root@34.139.13.169 'hadoop jar /tmp/wordcount-1.0-SNAPSHOT.jar /tmp/input.txt /tmp/output.txt'
                    """
                }
                sshagent(['hadoop-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no root@34.139.13.169:/tmp/output.txt /tmp/output.txt
                    """
                }
                sh 'cat /tmp/output.txt'
            }
        }

    }
}

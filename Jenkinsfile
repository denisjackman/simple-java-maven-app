pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Toolchain') {
            steps {
                sh '''
                    if ! command -v mvn >/dev/null 2>&1; then
                        sudo apt-get update && sudo apt-get install -y maven
                    fi
                    if ! java -version 2>&1 | grep -q '"21' ; then
                        sudo apt-get update && sudo apt-get install -y openjdk-21-jdk
                    fi
                '''
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Report') {
            steps {
                sh 'mvn -B org.apache.maven.plugins:maven-surefire-report-plugin:3.5.2:report-only'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    mkdir -p /var/www/html/simple-java-maven-app
                    REPORT_FILE=$(find target \\( -iname "surefire.html" -o -iname "surefire-report.html" \\) | head -n1)
                    if [ -z "$REPORT_FILE" ]; then
                        echo "surefire report not found; target contents:"
                        find target
                        exit 1
                    fi
                    REPORT_DIR=$(dirname "$REPORT_FILE")
                    rsync -a --delete "$REPORT_DIR/" /var/www/html/simple-java-maven-app/
                    cp "/var/www/html/simple-java-maven-app/$(basename "$REPORT_FILE")" /var/www/html/simple-java-maven-app/index.html
                '''
            }
        }
    }
}

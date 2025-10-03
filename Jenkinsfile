pipeline
{
    agent any
 
    environment {
        SONARQUBE = 'sonarqube'
        SCANNER = 'SonarScanner'
        SONAR_PROJECT_KEY = 'hello-python'
        SONAR_API_TOKEN =
credentials('sonar-token')  // Jenkins
secret
        SONAR_HOST_URL = 'http://136.114.113.205:9000' //Jenkin
VM IP
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ebrahimgamdiwala/hello-python.git'  //github username
            }
        }
 
        stage('Install & Run Tests') {
            steps {
                sh '''
                  echo "Installing
dependencies..."
                  python3 -m pip install
--upgrade pip
                  pip3 install --user -r
requirements.txt
                  
                  echo " Running
tests..."
PYTHONPATH=$PWD python3 -m pytest -q 
                '''
            }
        }
 
        stage('SonarQube Analysis (Async)') {
            steps {
               withSonarQubeEnv("${SONARQUBE}") {
                    withEnv(["PATH+SONAR=${tool
SCANNER}/bin"]) {
                        sh '''
                          echo "Running
SonarQube scan asynchronously..."
                          sonar-scanner \
                           -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                            -Dsonar.sources=. \
                           -Dsonar.host.url=$SONAR_HOST_URL \
                           -Dsonar.login=$SONAR_API_TOKEN \
                            -Dsonar.python.version=3.10
                        '''
                    }
                }
            }
        }
 
        stage('Deploy to App VM') {
            steps {
                sshagent(credentials:
['gce-ssh']) {
                    sh '''
                        echo "Deploying
with systemd..."
 
                        ssh -o
StrictHostKeyChecking=no ebrahimgamdiwala@34.29.89.164 "mkdir -p /home/ebrahimgamdiwala/app"    //appvm (to check IP address whoami)
                        scp -o
StrictHostKeyChecking=no -r * ebrahimgamdiwala@34.29.89.164:/home/ebrahimgamdiwala/app/
 
                        ssh -o
StrictHostKeyChecking=no ebrahimgamdiwala@34.29.89.164 "
                          sudo systemctl
daemon-reload &&
                          sudo systemctl restart
flaskapp &&
                          sudo systemctl enable
flaskapp
                        "
 
                        echo "Deployment
complete. Check: curl http://34.29.89.164:8080"
                    '''
                }
            }
        }
 
        stage('Optional: Check SonarQube
Quality Gate') {
            steps {
                script {
                    echo "Fetching
SonarQube Quality Gate result (non-blocking)..."
                    sh """
                      curl -s -u
$SONAR_API_TOKEN: \
                       "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY"
\
                        | jq
'.projectStatus.status'
                    """
                    echo "You can manually
inspect the SonarQube dashboard for full details."
                }
            }
        }
    }
 
    post {
        success {
            echo " Pipeline
Succeeded"
        }
        failure {
            echo " Pipeline Failed"
        }
    }
}

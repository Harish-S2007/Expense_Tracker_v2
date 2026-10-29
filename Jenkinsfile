
pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
                echo 'V2 source code checked out successfully.'
            }
        }

        stage('2. Check Tools') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $PYTHON = "C:\\Users\\hp\\expense_tracker_v2\\.venv\\Scripts\\python.exe"
                    $DOCKER = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"
                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"

                    foreach ($tool in @($PYTHON, $DOCKER, $KUBECTL)) {
                        if (-not (Test-Path $tool)) {
                            throw "Tool not found: $tool"
                        }
                    }

                    & $PYTHON --version
                    if ($LASTEXITCODE -ne 0) { throw "Python check failed" }

                    & $DOCKER --version
                    if ($LASTEXITCODE -ne 0) { throw "Docker check failed" }

                    & $KUBECTL version --client
                    if ($LASTEXITCODE -ne 0) { throw "kubectl check failed" }

                    Write-Host "V2 tools checked successfully."
                '''
            }
        }

        stage('3. Install Dependencies') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $PYTHON = "C:\\Users\\hp\\expense_tracker_v2\\.venv\\Scripts\\python.exe"

                    if (-not (Test-Path "requirements.txt")) {
                        throw "requirements.txt not found"
                    }

                    & $PYTHON -m pip install -r requirements.txt
                    if ($LASTEXITCODE -ne 0) {
                        throw "Dependency installation failed"
                    }
                '''
            }
        }

        stage('4. Test Application') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $PYTHON = "C:\\Users\\hp\\expense_tracker_v2\\.venv\\Scripts\\python.exe"

                    & $PYTHON -c "from app import app; print('V2 Flask application loaded successfully')"
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 application test failed"
                    }
                '''
            }
        }

        stage('5. Build Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $DOCKER = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"

                    if (-not (Test-Path "Dockerfile")) {
                        throw "Dockerfile not found"
                    }

                    Write-Host "Building Expense Tracker V2 image..."
                    & $DOCKER build -t expense-tracker:v2 .
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 Docker build failed"
                    }

                    & $DOCKER image inspect expense-tracker:v2
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 image verification failed"
                    }
                '''
            }
        }

        stage('6. Deploy V2 to Kubernetes') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    $env:KUBECONFIG = "C:\\Users\\hp\\.kube\\config"

                    & $KUBECTL config use-context docker-desktop
                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not select docker-desktop context"
                    }

                    & $KUBECTL get nodes
                    if ($LASTEXITCODE -ne 0) {
                        throw "Cannot connect to Kubernetes"
                    }

                    if (-not (Test-Path "deployment.yaml")) {
                        throw "deployment.yaml not found"
                    }

                    Write-Host "Applying V2 Kubernetes configuration..."
                    & $KUBECTL apply -f deployment.yaml
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 Kubernetes configuration failed"
                    }

                    & $KUBECTL rollout status deployment/expense-v2 --timeout=180s
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 rollout failed"
                    }
                '''
            }
        }

        stage('7. Verify V2 Deployment') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    $env:KUBECONFIG = "C:\\Users\\hp\\.kube\\config"

                    & $KUBECTL get deployment expense-v2
                    if ($LASTEXITCODE -ne 0) {
                        throw "V2 deployment not found"
                    }

                    & $KUBECTL get pods
                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not list pods"
                    }

                    & $KUBECTL get services
                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not list services"
                    }

                    & $KUBECTL get deployment expense-tracker
                    if ($LASTEXITCODE -ne 0) {
                        Write-Host "Warning: V1 deployment was not found."
                    }

                    Write-Host "V2 verification completed."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Expense Tracker V2 pipeline completed.'
        }
        failure {
            echo 'FAILURE: Check Console Output for the failed stage.'
        }
    }
}

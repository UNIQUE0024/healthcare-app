pipeline {
    agent any
    
    tools {
        maven 'Maven'
        jdk 'JDK-17'
    }
    
    environment {
        // Docker Hub
        DOCKER_IMAGE = 'unique/healthcare-app'
        DOCKER_TAG = "v1.0.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
        
        // Kubernetes
        K8S_NAMESPACE = 'healthcare'
    }
    
    stages {
        stage('1️⃣ Git Clone') {
            steps {
                echo '═══════════════════════════════════════'
                echo '     📥 Cloning Code from GitHub      '
                echo '═══════════════════════════════════════'
                git branch: 'main',
                    url: 'https://github.com/yourusername/healthcare-app.git'
                echo '✅ Code cloned successfully!'
            }
        }
        
        stage('2️⃣ Maven Build') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      🔨 Building with Maven           '
                echo '═══════════════════════════════════════'
                sh 'mvn clean compile'
                echo '✅ Build completed!'
            }
        }
        
        stage('3️⃣ Unit Tests') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      🧪 Running Unit Tests            '
                echo '═══════════════════════════════════════'
                sh 'mvn test'
                echo '✅ All tests passed!'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('4️⃣ Package WAR') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      📦 Creating WAR File             '
                echo '═══════════════════════════════════════'
                sh 'mvn package -DskipTests'
                echo '✅ WAR file created: target/healthcare.war'
            }
        }
        
        stage('5️⃣ Build Docker Image') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      🐳 Building Docker Image         '
                echo '═══════════════════════════════════════'
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
                echo "✅ Image built: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
        
        stage('6️⃣ Push to Docker Hub') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      ☁️  Pushing to Docker Hub        '
                echo '═══════════════════════════════════════'
                script {
                    sh """
                        echo ${DOCKER_CREDENTIALS_PSW} | docker login -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    """
                }
                echo '✅ Image available worldwide!'
                echo "   📦 ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
        
        stage('7️⃣ Deploy MySQL to K8s') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      🗄️  Deploying MySQL Database     '
                echo '═══════════════════════════════════════'
                script {
                    sh """
                        kubectl apply -f k8s/namespace.yml
                        kubectl apply -f k8s/mysql-secret.yml
                        kubectl apply -f k8s/mysql-pv.yml
                        kubectl apply -f k8s/mysql-pvc.yml
                        kubectl apply -f k8s/mysql-configmap.yml
                        kubectl apply -f k8s/mysql-deployment.yml
                        kubectl apply -f k8s/mysql-service.yml
                        
                        echo "⏳ Waiting for MySQL to be ready..."
                        kubectl wait --for=condition=ready pod -l app=mysql -n ${K8S_NAMESPACE} --timeout=120s
                    """
                }
                echo '✅ MySQL deployed and running!'
            }
        }
        
        stage('8️⃣ Deploy App to K8s') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      🚀 Deploying Healthcare App      '
                echo '═══════════════════════════════════════'
                script {
                    sh """
                        kubectl apply -f k8s/app-configmap.yml
                        kubectl apply -f k8s/app-deployment.yml
                        kubectl apply -f k8s/app-service.yml
                        
                        kubectl set image deployment/healthcare-app \
                          healthcare-app=${DOCKER_IMAGE}:${DOCKER_TAG} \
                          -n ${K8S_NAMESPACE}
                        
                        echo "⏳ Waiting for deployment rollout..."
                        kubectl rollout status deployment/healthcare-app -n ${K8S_NAMESPACE} --timeout=300s
                    """
                }
                echo '✅ Application deployed successfully!'
            }
        }
        
        stage('9️⃣ Verify Deployment') {
            steps {
                echo '═══════════════════════════════════════'
                echo '      ✔️  Verifying Deployment         '
                echo '═══════════════════════════════════════'
                script {
                    sh """
                        echo "📋 Checking Pods:"
                        kubectl get pods -n ${K8S_NAMESPACE}
                        
                        echo ""
                        echo "📋 Checking Services:"
                        kubectl get svc -n ${K8S_NAMESPACE}
                        
                        echo ""
                        echo "📋 Checking Persistent Volumes:"
                        kubectl get pv
                        
                        echo ""
                        echo "📋 Application URL:"
                        kubectl get svc healthcare-service -n ${K8S_NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
                    """
                }
                echo '✅ Verification complete!'
            }
        }
    }
    
    post {
        success {
            echo ''
            echo '╔═══════════════════════════════════════════════════╗'
            echo '║                                                   ║'
            echo '║         🎉 DEPLOYMENT SUCCESSFUL! 🎉              ║'
            echo '║                                                   ║'
            echo '╚═══════════════════════════════════════════════════╝'
            echo ''
            echo '📊 Deployment Summary:'
            echo '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━'
            echo "   🏷️  Build: #${BUILD_NUMBER}"
            echo "   🐳 Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "   ☸️  Namespace: ${K8S_NAMESPACE}"
            echo "   🌐 Check: kubectl get all -n ${K8S_NAMESPACE}"
            echo '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━'
        }
        
        failure {
            echo ''
            echo '╔═══════════════════════════════════════════════════╗'
            echo '║                                                   ║'
            echo '║           ❌ DEPLOYMENT FAILED! ❌                ║'
            echo '║                                                   ║'
            echo '╚═══════════════════════════════════════════════════╝'
            echo ''
            echo "❌ Check logs: ${BUILD_URL}console"
        }
        
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
    }
}

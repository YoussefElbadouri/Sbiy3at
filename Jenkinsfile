pipeline {
    agent any

    parameters {
        string(name: 'GITHUB_REPO', defaultValue: 'https://github.com/YoussefElbadouri/testt.git', description: 'Lien du projet GitHub')
    }

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'  // Remplace par le nom de ton serveur SonarQube dans Jenkins
        ZAP_API_KEY = "your_api_key"  // Remplace par ta vraie API Key OWASP ZAP
        ZAP_HOST = "localhost"
        ZAP_PORT = "9090"
        TARGET_URL = "http://localhost:5000"  // Met l'URL de l’application après déploiement
        REPORTS_DIR = "~/reports"
    }

    stages {
        stage('1️⃣ Récupération du Code') {
            steps {
                git branch: 'main', url: params.GITHUB_REPO
            }
        }

        stage('2️⃣ Analyse Statique - SonarQube') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=testt -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.token=sqp_b707df6bc1c65ca9d89e97d85f89490be3183cae"  //hada bdlo darori
                    }
                }
            }
        }

        stage('3️⃣ Scan des Dépendances') {
            steps {
                sh """
                    mkdir -p ${REPORTS_DIR}
                    echo "🔍 Analyse des dépendances avec Trivy..."
                    /usr/local/bin/trivy fs --severity HIGH,CRITICAL --no-progress --exit-code 1 . | tee ${REPORTS_DIR}/trivy-dependencies.txt
                    
                    echo "🔍 Analyse npm audit..."
                    export PATH=\$PATH:/usr/local/bin
                    npm audit --json > ${REPORTS_DIR}/npm-audit.json || true
                """

            }
        }

        stage('4️⃣ Build Docker Image') {
            steps {
                sh 'docker build -t my-app .'
            }
        }

        stage('5️⃣ Scan des Containers - Trivy') {
            steps {
                sh """
                    echo "🔍 Scan de l'image Docker avec Trivy..."
                    trivy image --severity HIGH,CRITICAL --no-progress --exit-code 1 my-app | tee ${REPORTS_DIR}/trivy-container.txt
                """
            }
        }

        stage('6️⃣ Déploiement de l’Application') {
            steps {
                sh 'docker run -d -p 5000:5000 --name my-app-container my-app'
            }
        }

        stage('7️⃣ Analyse Dynamique - OWASP ZAP') {
            steps {
                sh """
                    echo "🔍 Lancement du scan OWASP ZAP..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/spider/action/scan/?url=${TARGET_URL}&apikey=${ZAP_API_KEY}"
                    sleep 30
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/ascan/action/scan/?url=${TARGET_URL}&apikey=${ZAP_API_KEY}"
                    sleep 60
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/OTHER/core/other/htmlreport/?apikey=${ZAP_API_KEY}" > ${REPORTS_DIR}/zap-report.html
                """
            }
        }

        stage('8️⃣ Génération des Rapports') {
            steps {
                sh """
                    echo "📝 Génération des rapports..."
                    zip -r security-reports.zip ${REPORTS_DIR}
                """
                archiveArtifacts artifacts: 'security-reports.zip', fingerprint: true
            }
        }

        
    }
}

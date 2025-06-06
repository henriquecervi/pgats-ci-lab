pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
        YARN_CACHE_FOLDER = "${WORKSPACE}/.yarn-cache"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        skipStagesAfterUnstable()
    }
    
    triggers {
        // Execução automática em mudanças no SCM
        scm('H/5 * * * *')  // Verifica mudanças a cada 5 minutos
        
        // Execução agendada diária
        cron('H 2 * * *')  // Diariamente às 2h
    }
    
    stages {
        stage('🏗️ Setup e Preparação') {
            steps {
                script {
                    echo "=== INICIANDO PIPELINE PGATS-CI ==="
                    echo "Build: ${env.BUILD_NUMBER}"
                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Workspace: ${env.WORKSPACE}"
                }
                
                // Limpar workspace anterior
                cleanWs()
                
                // Checkout do código
                checkout scm
                
                // Instalar Node.js usando NodeJS Plugin
                script {
                    def nodeHome = tool name: 'NodeJS-18', type: 'nodejs'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                
                // Verificar versões
                sh '''
                    echo "Verificando versões instaladas:"
                    node --version
                    npm --version
                '''
                
                // Instalar Yarn globalmente
                sh 'npm install -g yarn'
                
                // Verificar Yarn
                sh 'yarn --version'
            }
        }
        
        stage('📦 Instalar Dependências') {
            steps {
                // Cache do Yarn
                script {
                    if (!fileExists("${YARN_CACHE_FOLDER}")) {
                        sh "mkdir -p ${YARN_CACHE_FOLDER}"
                    }
                }
                
                // Instalar dependências
                sh '''
                    echo "Instalando dependências do projeto..."
                    yarn install --frozen-lockfile --cache-folder ${YARN_CACHE_FOLDER}
                '''
                
                // Instalar navegadores Playwright
                sh '''
                    echo "Instalando navegadores Playwright..."
                    yarn playwright install --with-deps
                '''
            }
        }
        
        stage('🔍 Verificação de Qualidade') {
            parallel {
                stage('Linting') {
                    steps {
                        script {
                            try {
                                sh 'yarn run lint'
                                echo "✅ Linting passou com sucesso!"
                            } catch (Exception e) {
                                echo "⚠️ Linting falhou, mas continuando..."
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                }
                
                stage('Verificar Estrutura') {
                    steps {
                        sh '''
                            echo "Verificando estrutura do projeto:"
                            ls -la
                            echo "Verificando package.json:"
                            cat package.json | grep -E "(name|version|scripts)"
                        '''
                    }
                }
            }
        }
        
        stage('🧪 Testes Unitários') {
            steps {
                sh '''
                    echo "Executando testes unitários com Jest..."
                    yarn run test
                '''
            }
            
            post {
                always {
                    // Publicar resultados dos testes
                    script {
                        if (fileExists('results.xml')) {
                            publishTestResults testResultsPattern: 'results.xml'
                        }
                    }
                    
                    // Publicar cobertura de código
                    script {
                        if (fileExists('reports/coverage/lcov-report/index.html')) {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'reports/coverage/lcov-report',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report',
                                reportTitles: 'Relatório de Cobertura de Código'
                            ])
                        }
                    }
                }
            }
        }
        
        stage('🔬 Testes de Mutação') {
            when {
                anyOf {
                    branch 'main'
                    environment name: 'RUN_MUTATION', value: 'true'
                }
            }
            
            steps {
                script {
                    try {
                        timeout(time: 30, unit: 'MINUTES') {
                            sh '''
                                echo "Executando testes de mutação com Stryker..."
                                yarn run test:mutation
                            '''
                        }
                    } catch (Exception e) {
                        echo "⚠️ Testes de mutação falharam ou timeout, mas continuando..."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
            
            post {
                always {
                    script {
                        if (fileExists('reports/mutation/index.html')) {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'reports/mutation',
                                reportFiles: 'index.html',
                                reportName: 'Mutation Test Report',
                                reportTitles: 'Relatório de Testes de Mutação'
                            ])
                        }
                    }
                }
            }
        }
        
        stage('🎭 Testes End-to-End') {
            steps {
                sh '''
                    echo "Executando testes E2E com Playwright..."
                    yarn run e2e
                '''
            }
            
            post {
                always {
                    // Publicar resultados E2E
                    script {
                        if (fileExists('test-results/results.xml')) {
                            publishTestResults testResultsPattern: 'test-results/results.xml'
                        }
                    }
                    
                    // Publicar relatório Playwright
                    script {
                        if (fileExists('playwright-report/index.html')) {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Report',
                                reportTitles: 'Relatório de Testes E2E'
                            ])
                        }
                    }
                    
                    // Arquivar screenshots e vídeos
                    script {
                        if (fileExists('test-results')) {
                            archiveArtifacts artifacts: 'test-results/**/*', 
                                           fingerprint: true, 
                                           allowEmptyArchive: true
                        }
                    }
                }
            }
        }
        
        stage('🚀 Build e Deploy') {
            when {
                anyOf {
                    branch 'main'
                }
            }
            
            steps {
                sh '''
                    echo "Preparando build da aplicação RoboCoasters..."
                    
                    # Criar diretório de build
                    mkdir -p dist
                    
                    # Copiar arquivos estáticos
                    cp -r css dist/ 2>/dev/null || echo "Diretório css não encontrado"
                    cp -r src dist/ 2>/dev/null || echo "Diretório src não encontrado"  
                    cp -r img dist/ 2>/dev/null || echo "Diretório img não encontrado"
                    cp index.html dist/ 2>/dev/null || echo "index.html não encontrado"
                    cp favicon.ico dist/ 2>/dev/null || echo "favicon.ico não encontrado"
                    
                    echo "Conteúdo do build:"
                    ls -la dist/
                '''
            }
            
            post {
                success {
                    // Arquivar artefatos da aplicação
                    archiveArtifacts artifacts: 'dist/**/*', 
                                   fingerprint: true,
                                   allowEmptyArchive: true
                    
                    echo "✅ Build da aplicação concluído com sucesso!"
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "=== RESUMO DA EXECUÇÃO PGATS-CI ==="
                echo "Build: ${env.BUILD_NUMBER}"
                echo "Status: ${currentBuild.result ?: 'SUCCESS'}"
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Duração: ${currentBuild.durationString}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "========================================="
            }
            
            // Limpar cache antigo
            script {
                sh "find ${YARN_CACHE_FOLDER} -type f -mtime +7 -delete 2>/dev/null || true"
            }
        }
        
        success {
            echo "🎉 Pipeline executado com SUCESSO!"
            script {
                if (env.BRANCH_NAME == 'main') {
                    echo "🚀 Deploy realizado na branch principal!"
                }
            }
        }
        
        failure {
            echo "❌ Pipeline FALHOU! Verifique os logs acima."
        }
        
        unstable {
            echo "⚠️ Pipeline INSTÁVEL! Alguns testes falharam."
        }
        
        cleanup {
            // Limpar workspace se necessário
            script {
                if (params.CLEAN_WORKSPACE == true) {
                    cleanWs()
                }
            }
        }
    }
} 
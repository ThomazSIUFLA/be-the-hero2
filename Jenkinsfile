pipeline {
docker {
      image 'cypress/base:10'
    }
agent any
    options { timestamps() }

    stages {
        stage('teste unitario - backend'){
            steps{
                dir('backend'){
                    nodejs(nodeJSInstallationName: 'node12') {
                        sh 'npm install'
                        sh 'npx knex migrate:latest'
                        sh 'npm test'
                    }
                }
            }
        }

        stage('build - frontend'){
            steps{
                dir('frontend'){
                    nodejs(nodeJSInstallationName: 'node14') {
                        sh 'npm install'
                        sh 'npm run build'
                    }
                }
            }
        }
        
        stage('analise de codigo - sonar'){
            steps{
                withSonarQubeEnv('sonar') {
                    nodejs(nodeJSInstallationName: 'node14') {
                        sh 'sonar-scanner -Dsonar.projectKey=be-the-hero -Dsonar.sources=. -Dsonar.host.url=http://10.5.0.3:9000 -Dsonar.login=dfceaadc37950594b87877a0cedf3ecfa3c2df98 -Dsonar.exclusions=tests/**'    
                    }
                    
                }
            }
        }
        
        stage('quality gate - sonar'){
            steps{
                sleep(5)
                timeout(time:1, unit:'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('deploy - backend'){
            steps{
                dir('backend'){
                    sh 'cp -r -f . /var/dados_back'
                }
            }
        }

        stage('deploy - frontend'){
            steps{
                dir('frontend'){
                    sh 'cp -r -f ./build /var/dados_front'
                }
            }
        }
        
        // stage('testes e2e - cypress'){
        //     steps{
        //         nodejs(nodeJSInstallationName: 'node14') {
        //             ansiColor('css') {
        //                 sh 'npm install'
        //                 sh 'npm install cypress'
        //                 sh 'npx cypress run'
        //             }
        //         }
        //     }
        // }

        stage('build and test') {
            environment {
                // we will be recording test results and video on Cypress dashboard
                // to record we need to set an environment variable
                // we can load the record key variable from credentials store
                // see https://jenkins.io/doc/book/using/using-credentials/
                CYPRESS_RECORD_KEY = credentials('cypress-example-kitchensink-record-key')
            }

            steps {
                sh 'npm ci'
                sh "npm run test:ci:record"
            }
    }
}
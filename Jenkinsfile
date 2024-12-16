pipeline {
    agent any
    
    stages {
        stage('Git') {
            steps {
                git branch: 'develop', url: 'https://github.com/jodelaenc/UNIR.git'
            }
        }
        stage('Build') {
            steps {
                echo 'No compila'
                bat "dir"
            }
        }
        stage('Tests'){
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                    }
                }
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                start start java -jar C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\Project\\helloworld\\test\\wiremock
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Results') {
            steps {
                junit 'result*.xml'
            }
        }
    }
}
pipeline {
    agent any

    stages {
        stage('Git') {
            steps {
                git branch: 'develop', url: 'https://github.com/jodelaenc/UNIR.git'
                stash includes: '**', name: 'source-code'
            }
        }
        stage('Build') {
            steps {
                echo 'No compila'
                bat "dir"
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit') {
                    agent {
                        label 'agent1'
                    }
                    steps {
                        unstash 'source-code'
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                        stash includes: 'result-unit.xml', name: 'unit-test-results'
                    }
                }
                stage('Rest') {
                    agent {
                        label 'agent2'
                    }
                    steps {
                        unstash 'source-code'
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                start java -jar C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\Project\\helloworld\\test\\wiremock
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                        stash includes: 'result-rest.xml', name: 'rest-test-results'
                    }
                }
            }
        }

        stage('Results') {
            steps {
                unstash 'unit-test-results'
                unstash 'rest-test-results'
                junit 'result*.xml'
            }
        }
    }
}
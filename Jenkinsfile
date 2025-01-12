pipeline {
    agent any

    stages {
        stage('Git') {
            steps {
                git branch: 'develop', url: 'https://github.com/jodelaenc/UNIR.git'
		echo WORKSPACE
		bat 'dir'
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
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
			    unstash 'source-code'4
                            sh '''
			        ls -la
                                export PYTHONPATH=${WORKSPACE}
                                home/agent1/.local/bin/pytest --junitxml=result-unit.xml test/unit
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
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
			    unstash 'source-code'
                            sh '''
                                export FLASK_APP=app\\api.py
                                home/agent2/.local/bin/flask run & sleep 4
                                java -jar /home/agent2/wiremock/wiremock-standalone-3.10.0.jar --port 9090 --root-dir /home/agent2/wiremock &  
                                export PYTHONPATH=${WORKSPACE}
				sleep 15
                                home/agent2/.local/bin/pytest --junitxml=result-rest.xml test/rest
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

pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                    git branch: 'develop', url: 'https://github.com/jodelaenc/UNIR.git'
                    bat 'dir'
                    echo WORKSPACE
                    stash includes: '**', name: 'code'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit') {
                    agent {
                        label 'agent1' 
                    }
                    steps {
                            // Comandos para obtener información del entorno
                            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                unstash 'code'  // Recuperar el código fuente
                                bat '''
                                    set PYTHONPATH=.
                                    pytest --junitxml=result-unit.xml test\\unit
                                '''
                                 junit 'result-unit.xml'  // Publicar resultados de las pruebas
                            }
                    }
                }

                stage('Rest') {
                    agent {
                        label 'agent2' 
                    }
                    steps {
                        script {
                            // Comandos para obtener información del entorno
                            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                unstash 'code'  // Recuperar el código fuente
                                bat '''
                                    set FLASK_APP=app\\api.py
                                    start flask run
                                    start java -jar C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir C:\\Users\\jodel\\Desktop\\UNIR\\Practicas\\CP1\\Project\\helloworld\\test\\wiremock
                                    sleep 15
                                    pytest --junitxml=result-rest.xml test/rest
                                '''
                                junit 'result-rest.xml'
                            }
                        }
                    }
                }
                stage('Static'){
                    agent {
                        label 'agent1' 
                    }
                    steps{
                        bat '''
                            flake8 --format=pylint --exit-zero app > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold:8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                    }
                }
                
                stage('Security Test'){
                    agent {
                        label 'agent1' 
                    }
                    steps{
                        bat '''
                            bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}“
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold:2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                    }
                }
            }
        }
        
        stage('Performance'){
                agent {
                    label 'agent2'  
                }
                steps{
                    bat '''
                        set FLASK_APP=app\\api.py
                        start flask run
                        sleep 15
                        C:\\Users\\jodel\\Desktop\\UNIR\\JMeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\jMeterFinal.jmx -f -l flask.jtl
                    '''
                    perfReport sourceDataFiles : 'flask.jtl'
                }
        }
        
        stage('Coverage'){
            agent {
                label 'agent3' 
            }
            steps{
                bat '''
                    coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                    coverage xml
                    type coverage.xml
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets:'80,80,90', lineCoverageTargets:'85,85,90'
                }
            }
        }
    }
    
    post {
        always {
            script {
                    echo "Limpiando el workspace..."
                    deleteDir()                
                }
        }
    }
    
}

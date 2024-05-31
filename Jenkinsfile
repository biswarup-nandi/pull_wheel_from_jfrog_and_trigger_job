def DBCLIPATH = "/usr/local/bin"

pipeline {
    agent any

    parameters {
        string defaultValue: 'dev', description: 'Deployment Environment', name: 'env'
        string defaultValue: 'pull_wheel_from_jfrog_and_trigger_job', description: 'Deployment Environment', name: 'job_name'
    }


    environment {
        ARTIFACTORY_URL = 'https://biswarupnandi.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'dbx-dbx-python'
        ARTIFACTORY_SERVER = 'jfrog-artifact-instance'
        PYTHON_VERSION = '3.10.12'
    }

    stages {
        stage('Setup Python') {
            steps {
                sh """
                    /usr/bin/python3 --version
                    /usr/bin/python3 -m venv venv
                    . venv/bin/activate
                    python --version
                    pip --version
                    pip install --upgrade pip
                """
            }
        }

        stage('Pull Wheel') {
            steps {
                script {
                    def server = Artifactory.server("${ARTIFACTORY_SERVER}")
                    def downloadSpec = """{
                        "files": [{
                            "pattern": "${ARTIFACTORY_REPO}/*.whl",
                            "target": "dist/",
                            "sortBy" : ["modified"],
                            "sortOrder": "desc",
                            "limit": "1"
                        }]
                    }"""
                    server.download(downloadSpec)
                }
                sh """
                    echo "Current Directory List --->"
                    ls -ltr
                    echo "Wheel(dist) Directory List --->"
                    ls -ltr dist/
                """
            }
        }

        stage('DAB Destroy (If Exist)') {
            steps {
                script {
                    try {
                        def command = "${DBCLIPATH}/databricks bundle destroy -t ${params.env} --auto-approve"
                        // Execute the command
                        sh(command)
                    } catch (Exception e) { 
                        echo "Error: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('DAB Validation') {
            steps {
                script {
                    try {
                        def command = "${DBCLIPATH}/databricks bundle validate -t ${params.env}"
                        // Execute the command
                        sh(command)
                    } catch (Exception e) { 
                        echo "Error: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('DAB Deployment') {
            steps {
                script {
                    try {
                        def command = "${DBCLIPATH}/databricks bundle deploy -t ${params.env}"
                        // Execute the command
                        sh(command)
                    } catch (Exception e) { 
                        echo "Error: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Job Run') {
            steps {
                script {
                    try {
                        def command = "${DBCLIPATH}/databricks bundle run -t ${params.env} ${params.job_name}"
                        // Execute the command
                        sh(command)
                    } catch (Exception e) { 
                        echo "Error: ${e.getMessage()}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

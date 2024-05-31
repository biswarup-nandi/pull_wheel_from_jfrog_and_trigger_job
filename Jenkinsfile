pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'https://biswarupnandi.jfrog.io/artifactory'
        ARTIFACTORY_REPO = 'dbx-dbx-python'
        ARTIFACTORY_SERVER = 'jfrog-artifact-instance'
        PYTHON_VERSION = '3.10.12'
        // DATABRICKS_HOST = 'https://accounts.cloud.databricks.com'
        // DATABRICKS_AUTH_TYPE = 'oauth-m2m'
        // DATABRICKS_REGION = 'us-east-1'
        // DATABRICKS_ACCOUNT_ID = credentials('databricks-account-id')
        // DATABRICKS_CLIENT_ID = credentials('databricks-client-id')
        // DATABRICKS_CLIENT_SECRET = credentials('databricks-client-secret')
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

        stage('Pull and Install Wheel') {
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
                    ls -ltr dist/
                """
            }
        }

        // stage('Run Main Function') {
        //     steps {
        //         sh """
        //             . venv/bin/activate
        //             pip list
        //             dbx-utility
        //         """
        //     }
        // }
    }

    post {
        always {
            cleanWs()
        }
    }
}

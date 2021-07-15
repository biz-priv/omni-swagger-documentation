pipeline {
    agent { label 'ecs' }
    stages {
        stage('Set parameters') {
            steps {
                script{
                    echo "GIT_BRANCH: ${GIT_BRANCH}"
                    echo sh(script: 'env|sort', returnStdout: true)
                    if ("${GIT_BRANCH}".contains("feature") || "${GIT_BRANCH}".contains("bugfix") || "${GIT_BRANCH}".contains("devint")) {
                        env.ENVIRONMENT=env.getProperty("environment_devint")
                    } else if ("${GIT_BRANCH}".contains("develop")) {
                        env.ENVIRONMENT=env.getProperty("environment_develop")
                    } else if ("${GIT_BRANCH}".contains("master")) {
                        env.ENVIRONMENT=env.getProperty("environment_prod")
                    }
                    sh """
                    echo "Environment: "${env.ENVIRONMENT}
                    """
                }
            }
        }
        stage('BizDev Deploy'){
            when {
                anyOf {
                    branch 'devint'
                }
                expression {
                    return true;
                }
            }
            steps {
                withAWS(credentials: 'bizdev-aws-creds'){
                    sh """
                    cd sceptre-project
                    sceptre launch ${env.ENVIRONMENT} --yes
                    cd ..
                    aws s3 cp swagger s3://omni-redoc-bucket-${env.ENVIRONMENT} --recursive
                    """
                }
            }
        }
        stage('Omni Deploy'){
            when {
                anyOf {
                    branch 'develop';
                    branch 'master';
                }
                expression {
                    return true;
                }
            }
            steps {
                withAWS(credentials: 'omni-aws-creds'){
                    sh """
                    cd sceptre-project
                    sceptre launch ${env.ENVIRONMENT} --yes
                    cd ..
                    aws s3 cp swagger s3://omni-redoc-bucket-${env.ENVIRONMENT} --recursive
                    """
                }
            }
        }
    }
}
def deployApp(String target, String webPort, String miniPort) {
    sh """
        rsync -rv --delete --exclude=.git --exclude=Jenkinsfile ./ /var/www/${target}-tamashaweb/
        rsync -rv --delete --exclude=.git --exclude=Jenkinsfile ./ /var/www/${target}-tamasha_mini/
        curl -fsS -o /dev/null http://127.0.0.1:${webPort}/
        curl -fsS -o /dev/null http://127.0.0.1:${miniPort}/
    """
}

pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Validate') {
            steps {
                sh '''
                    echo "Branch: $BRANCH_NAME"
                    test -f index.html
                    echo "Branch: $BRANCH_NAME | Build: $BUILD_NUMBER | Commit: $GIT_COMMIT" > version.txt
                '''
            }
        }

        // ---------- DEV: automatic ----------
        stage('Deploy to DEV') {
            when { branch 'dev' }
            steps { script { deployApp('dev', '8081', '9081') } }
        }

        // ---------- UAT: approval ke baad ----------
        stage('Approve UAT') {
            when { branch 'uat' }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'UAT par deploy karein?', ok: 'Deploy'
                }
            }
        }
        stage('Deploy to UAT') {
            when { branch 'uat' }
            steps { script { deployApp('uat', '8082', '9082') } }
        }

        // ---------- PREPROD: approval ke baad ----------
        stage('Approve PreProd') {
            when { branch 'preprod' }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'PreProd par deploy karein?', ok: 'Deploy'
                }
            }
        }
        stage('Deploy to PreProd') {
            when { branch 'preprod' }
            steps { script { deployApp('preprod', '8083', '9083') } }
        }
    }

    post {
        success { echo "Deployment kamyab: ${env.BRANCH_NAME}" }
        failure { echo "Deployment fail: ${env.BRANCH_NAME}" }
    }
}

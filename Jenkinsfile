node {
    docker.image('python:2-alpine').inside {
        stage('Build') { 
            sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
        }
    }
    docker.image('qnib/pytest').inside {
        try {
            stage('Test') { 
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
        finally{
            junit 'test-reports/results.xml'
        }
    }
    docker.image('cdrx/pyinstaller-linux:python2').inside {
        try {
            stage('Archive') {
                sh '/root/.pyenv/shims/pyinstaller --onefile sources/add2vals.py'
            }
            archiveArtifacts 'dist/add2vals'
            stage('Manual Approval') { 
                input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk Deploy)'
            }
        }
        catch (e) {
            echo "An error happened: $e"
        throw e
        }
    }
    docker.image('alpine:latest').inside {
        try {
            stage('Deploy') {
                withCredentials([sshUserPrivateKey(credentialsId: 'infra-ubuntu', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        cat "$SSH_KEY" > ssh_key
                        chmod 600 ssh_key
                        apk add openssh
                        scp -o StrictHostKeyChecking=no -i ssh_key dist/add2vals ubuntu@54.87.148.162:/home/ubuntu/deploy
                        ssh -o StrictHostKeyChecking=no -i ssh_key ubuntu@54.87.148.162 "sudo chmod +X /home/ubuntu/deploy/*"
                        '''
                }
            }
            sleep(time: 1, unit: 'MINUTES')
        }
        catch (e) {
            echo "An error happened: $e"
        throw e
        }
    }
}
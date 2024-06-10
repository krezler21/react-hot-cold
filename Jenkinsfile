pipeline {
agent any

    environment{
        DOCKERHUB_CREDENTIALS = credentials('DockerHub')
        GITHUB_CREDENTIALS = credentials('ce30c1a0-401d-4402-922a-0791cf0781d6')
        GITHUB_REPO = 'krezler21/react-hot-cold'
        NEXT_VERSION = nextVersion()
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        
        stage('Pull'){
            steps{
                echo "Pulling repo stage"
                git branch: 'main', url: 'https://github.com/krezler21/react-hot-cold'
     
            }
        }
        
        stage('Build') {
            steps {
                echo "Build stage"
                sh '''
                docker build -t react-hot-cold:latest  -f./building/Dockerfile .
                docker run --name build_container react-hot-cold:latest
                docker cp build_container:/react-hot-cold/build ./artefakty
                docker logs build_container > log_build.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo "Test stage"
                sh '''
                docker build -t react-hot-cold-test:latest -f ./test/Dockerfile .
                docker run --name test_container react-hot-cold-test:latest
                docker logs test_container > log_test.txt

                '''
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploy stage"
                sh '''
                
                docker build -t react-hot-cold-deploy:latest -f ./deploy/Dockerfile .
                docker run -p 3000:3000 -d --rm --name deploy_container react-hot-cold-deploy:latest
                
                '''
        }
        }

        stage('Publish') {
            steps {
                echo "Publish stage"
                sh '''
                TIMESTAMP=$(date +%Y%m%d%H%M%S)
                tar -czf artifact_$TIMESTAMP.tar.gz log_build.txt log_test.txt artefakty
                git config --global user.email "krezler21@gmail.com"
                git config --global user.name "krezler21"
                git tag -a ${NEXT_VERSION} -m "tag"
                git push https://${GITHUB_CREDENTIALS_PSW}@github.com/${GITHUB_REPO} ${NEXT_VERSION}

                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
              
                docker tag react-hot-cold-deploy:latest krezler21/react-hot-cold:${NEXT_VERSION}
                docker push krezler21/react-hot-cold:${NEXT_VERSION}
                docker logout

                echo

                '''
            } 
        }

    }
                post{
            always{
            echo "Archiving artifacts"

            archiveArtifacts artifacts: 'artifact_*.tar.gz', fingerprint: true
            sh '''
            chmod +x cleanup.sh
            ./cleanup.sh
            '''
        }
}


}
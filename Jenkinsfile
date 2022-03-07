pipeline
{
    agent 
    {
        label 'docker-node'
    }
    environment 
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages
    {
        stage('Downloading the project from Gitea') 
        {
            steps 
            {
                git branch: 'main', url: 'http://192.168.99.102:3000/rachko69/bgapp'
            }
        }
        stage('Compose the Project BGApp for test')
        {
            steps
            {
                sh 'docker-compose up -d'
            }
        }
        stage('Test')
        {
            steps
            {
                script 
                {
                    echo 'Test #1 - reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://192.168.99.102:8000) | grep 200'
                    
                    echo 'Test #2 - Бургас'
                    sh 'curl --write-out "%{http_code}" --silent http://192.168.99.102:8000 | grep София'
                }
            }
        }
        stage('Stopping the test application and removing the containers')
        {
            steps
            {
                echo 'CleanUp Testing App'
                sh 'docker container ls | grep bgapp* && docker container rm -f $(docker ps -f name=bgapp* --format "{{.ID}}")'
                sh 'docker network ls | grep bgapp* && docker network rm $(docker network ls -f name=bgapp* --format "{{.ID}}")'
            }
        }
        stage('Login to DockerHub') 
        {
            steps 
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push image to DockerHub') 
        {
            steps 
            {
                sh 'docker image tag bgapp_web rachko69/image-bgapp-web'
                sh 'docker image tag bgapp_db rachko69/image-bgapp-db'
                sh 'docker push rachko69/image-bgapp-web'
                sh 'docker push rachko69/image-bgapp-db'
            }
        }
        stage('Deploy to prod')
        {
            steps
            {
                sh 'docker-compose -f docker-compose-prod.yaml up -d'
            }
        }
    }
}

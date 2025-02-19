pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "mirael13" 
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any 
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker build -f movie-service/Dockerfile -t $DOCKER_ID/movie_service:$DOCKER_TAG movie-service/
				         docker build -f cast-service/Dockerfile -t $DOCKER_ID/cast_service:$DOCKER_TAG cast-service/
                sleep 6
                '''
                }
            }
        }
		
		
		
		stage(' Docker Push'){ // docker build image stage
		    environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
			
            steps {
                script {
                sh '''
                 docker login -u $DOCKER_ID -p $DOCKER_PASS
                 docker push $DOCKER_ID/movie_service:$DOCKER_TAG
				         docker push $DOCKER_ID/cast_service:$DOCKER_TAG
                '''
                }
            }
        }
		
		
		
		stage('Create Docker Registry Secret') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    kubectl create secret docker-registry regcred \
                        --docker-server=https://index.docker.io/v1/ \
                        --docker-username=$DOCKER_ID \
                        --docker-password=$DOCKER_PASS \
                        --docker-email=votre@email.com \
                        -n dev --dry-run=client -o yaml | kubectl apply -f -
                    kubectl create secret docker-registry regcred \
                        --docker-server=https://index.docker.io/v1/ \
                        --docker-username=$DOCKER_ID \
                        --docker-password=$DOCKER_PASS \
                        --docker-email=votre@email.com \
                        -n qa --dry-run=client -o yaml | kubectl apply -f -
                    kubectl create secret docker-registry regcred \
                        --docker-server=https://index.docker.io/v1/ \
                        --docker-username=$DOCKER_ID \
                        --docker-password=$DOCKER_PASS \
                        --docker-email=votre@email.com \
                        -n staging --dry-run=client -o yaml | kubectl apply -f -
                    kubectl create secret docker-registry regcred \
                        --docker-server=https://index.docker.io/v1/ \
                        --docker-username=$DOCKER_ID \
                        --docker-password=$DOCKER_PASS \
                        --docker-email=votre@email.com \
                        -n prod --dry-run=client -o yaml | kubectl apply -f -
                    '''
                }
            }
        }
        
    stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                sed -i "s/imagePullSecrets:.*/imagePullSecrets:\\n  - name: regcred/g" values.yml
                helm upgrade --install eval-jenkins-chart charts/ --values=values.yaml --namespace dev
                '''
                }
            }

        }
        
    stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                sed -i "s/imagePullSecrets:.*/imagePullSecrets:\\n  - name: regcred/g" values.yml
                helm upgrade --install eval-jenkins-chart charts/ --values=values.yaml --namespace staging
                '''
                }
            }

        }
        
        
   stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                sed -i "s/imagePullSecrets:.*/imagePullSecrets:\\n  - name: regcred/g" values.yml
                helm upgrade --install eval-jenkins-chart charts/ --values=values.yaml --namespace qa
                '''
                }
            }

        }
        
        
   stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
        when{
        branch 'master'
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp charts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                sed -i "s/imagePullSecrets:.*/imagePullSecrets:\\n  - name: regcred/g" values.yml
                helm upgrade --install eval-jenkins-chart charts/ --values=values.yaml --namespace prod
                '''
                }
            }

        }
		
		
		

}
}






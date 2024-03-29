pipeline {
    environment {
        /*DOCKERHUB_CREDENTIALS = credentials('fd264a96-f98f-4122-bb6f-a19b3ad27e58')*/
	registry = "mikeangelo37/irssi"
	registryCredential = 'fd264a96-f98f-4122-bb6f-a19b3ad27e58'
    }
    parameters {
        string(name: 'VERSION', defaultValue: '0.0.0', description: '')
        booleanParam(name: 'PROMOTE', defaultValue: true, description: '')
    }
    agent any
    stages {
        stage('Prepare') {
            steps {
                sh 'DOCKER_TLS_VERIFY=0 docker rm -f buffer'
                sh 'docker volume prune -f'
                sh 'docker volume create volin'
                sh 'docker run -v volin:/data --name buffer ubuntu'
                sh 'cd ~/ && find irssi || git clone https://github.com/irssi/irssi.git/'
                sh 'docker cp ~/irssi buffer:/data'
                sh 'docker rm buffer'
                echo 'Cloning...'
            }
        }
        stage('Build') {
            steps {
	      checkout([$class: 'GitSCM', 
              branches: [[name: '*/main']], 
              userRemoteConfigs: [[url: 'https://github.com/MikeAngelo37/MDO_LAB-5-6-7']]])
	      	sh 'rm -f /var/jenkins_home/workspace/MDOL6/log-*.txt'
       		sh 'docker system prune -f'
	      	git 'https://github.com/MikeAngelo37/irssi'
        	sh 'docker build -t irssibld . -f DockerfileBuild'
        	sh 'docker volume create volout'
        	sh 'docker run --mount type=volume,src="volin",dst=/app --mount type=volume,src="volout",dst=/app/result irssibld bash -c "ls -l && cd irssi && 		meson setup build && ninja -C build; cp -r ../irssi ../result" > log-build.txt'
        	echo 'Building...'
		archiveArtifacts artifacts: "log-build.txt"
               
            }
            post {
                failure {
                    echo 'Build - fail.'
                }
                success {
                    echo 'Build - success.'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'docker build -t irssitst . -f DockerfileTest'
                sh 'docker run -t --mount type=volume,src="volin",dst=/app irssitst bash -c "cd irssi/build && meson test" > log-test.txt'
		archiveArtifacts artifacts: "log-test.txt"
            }
             post {
                failure {
                    echo 'Test - fail.'
                }
                success {
                    echo 'Test - success.'
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
		sh 'docker rm -f deploybuffer || true'
    		sh 'docker run -dit --name deploybuffer --mount type=volume,src="volout",dst=/app/result ubuntu > log-deploy.txt'
    		sh 'docker container exec deploybuffer sh -c "apt-get update" >> log-deploy.txt'
		sh 'docker container exec deploybuffer sh -c "DEBIAN_FRONTEND="noninteractive" apt-get install -y libglib2.0" >> log-deploy.txt'
		sh "docker container exec deploybuffer sh -c 'apt-get install -y libutf8proc-dev' >> log-deploy.txt"
		sh "docker container exec deploybuffer sh -c 'cd /app/result/irssi/build/src/fe-text && ./irssi -v' >> log-deploy.txt"
		sh "docker container kill deploybuffer"
		sh 'docker rm -f deploybuffer'
		archiveArtifacts artifacts: "log-deploy.txt"
            }
            
        }
        stage('Publish') {
            when {
                expression {return params.PROMOTE}
            }
            steps {
		echo 'Publishing...'
		script {
		    dockerImage = docker.build registry + ":$BUILD_NUMBER"
		    docker.withRegistry( '', registryCredential ) {
			dockerImage.push()
		    }
		}
		/*sh 'docker login'
		sh 'docker tag irssibld mikeangelo37/irssi:latest'
		sh 'docker push mikeangelo37/irssi:latest'*/
		archiveArtifacts artifacts: "log-publish.txt"
            }
        }
    }
}
/* fd264a96-f98f-4122-bb6f-a19b3ad27e58 */

		/*echo 'Publishing...'
		sh 'docker tag irssi mikeangelo37/irrsi:latest'
		sh 'docker push mikeangelo37/irrsi:latest'
		archiveArtifacts artifacts: "log-publish.txt"
                /*echo 'Publishing...'
		sh 'docker rm -f publishbuffer || true'
    		sh 'find /var/jenkins_home/workspace -name "artifacts" || mkdir /var/jenkins_home/workspace/artifacts'
    		sh 'docker run -d --rm --name publishbuffer --mount type=volume,src="volout",dst=/app/result --mount type=bind,source=/var/jenkins_home/workspace/artifacts,target=/usr/local/copy ubuntu  bash -c "chmod -R 777 /app && cp -r /app/. /usr/local/copy" > log-publish.txt'
		sh "rm -r /var/jenkins_home/workspace/tmp/*"
		sh "cp -R * /var/jenkins_home/workspace/tmp"
		sh "cd /var/jenkins_home/workspace/tmp"
		sh "tar -zcvf irssi-ver${params.VERSION}.tar.gz -C /var/jenkins_home/workspace/artifacts . >> log-publish.txt"
		sh "cd /var/jenkins_home/workspace/artifacts"
		archiveArtifacts artifacts: "irssi-ver${params.VERSION}.tar.gz"
		archiveArtifacts artifacts: "log-build.txt"
		archiveArtifacts artifacts: "log-test.txt"
		archiveArtifacts artifacts: "log-deploy.txt"
		archiveArtifacts artifacts: "log-publish.txt"
		sh 'docker rm -f publishbuffer'*/

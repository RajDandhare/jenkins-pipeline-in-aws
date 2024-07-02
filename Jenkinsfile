pipeline{
	agent any
	stages {
		stage('fetch-code'){
			steps {
				sh '''
				cd /git-files/jenkins-docker-webpage/
				sudo git pull
				'''
			}
		}
		stage('build-image'){
			steps{
				sh '''
				cd /git-files/jenkins-docker-webpage/
				sudo ansible-playbook docker-build.yml
				'''
			}
		}
		stage('deploy-image'){
			steps{
				sh '''
				cd /git-files/jenkins-docker-webpage/
				sudo ansible-playbook docker-deploy.yml
				'''
			}
		}
	}
}

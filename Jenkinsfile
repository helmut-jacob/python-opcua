pipeline {
	agent {
		docker {
			image 'python:3.6-alpine'
		}
	}
	options {
		timeout(time: 10, unit: 'MINUTES')
		timestamps ()
	}
	stages {
		stage('Dependencies') {
			steps {
				withCredentials([usernamePassword(credentialsId: 'artifactory-cloud-build', usernameVariable: 'ART_USR', passwordVariable: 'ART_PSW')]){
					sh 'pip install -i "https://$ART_USR:$ART_PSW@rommelag.jfrog.io/rommelag/api/pypi/pypi-latest/simple" setuptools==38.5.2'
				}
			}
		}
		stage('Build') {
			steps {
				sh 'python setup.py build sdist'
			}
		}
		stage('Upload') {
			when{
				branch 'ilabs'
			}
			steps {
				sh '''
					VER=$(grep "version=" setup.py | sed -e "s/.*\\"\\(.*\\)\\".*/\\1/g")
					cd dist
					tar xf opcua-${VER}.tar.gz
					mv opcua-${VER} opcua-${VER}+ilabs.${BUILD_NUMBER}
					sed -i -e "s/version=\".*\"/version=\\"${VER}+ilabs.${BUILD_NUMBER}\\",/" opcua-${VER}+ilabs.*/setup.py
					tar cfz opcua-${VER}+ilabs.${BUILD_NUMBER}.tar.gz opcua-${VER}+ilabs.${BUILD_NUMBER}/
                    apk update #hackedy hack hack hack
				'''
				pythonUpload 'dist/opcua-*+ilabs.*.tar.gz'
			}
		}
	}
}


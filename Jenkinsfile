pipeline {
  agent {
    kubernetes {
      containerTemplate {
        name 'gradle'
        image 'gradle:4.5.1-jdk9'
        ttyEnabled true
        command 'cat'
      }
    }
  }
  
  triggers {
	  pollSCM 'H/2 * * * *'
	}
  

	
	options {
		buildDiscarder(logRotator(numToKeepStr:'3'))
        timeout(time: 30, unit: 'MINUTES')
    }
  
  stages {
    stage('Config') {
      steps {
      	echo "Rama: ${env.BRANCH_NAME},  codigo de construccion: ${env.BUILD_ID} en ${env.JENKINS_URL}"
		echo "Iniciando limpieza"
	sh 'gradle clean -x check -x test'

      }
    }
    
    stage('Build') {
			steps {
				echo "Iniciando construccion"
				script {
					if ( env.BRANCH_NAME == 'master' ) {
						echo "Iniciando construccion master"
					}else {
						echo "Iniciando construccion develop"
					}
				}
			}
	}
	
	stage('Test') {
			parallel {
				
				stage('Integration Test') {
					steps {
						script {
							try {
								echo "Iniciando analisis de calidad del repositorio de la rama ${env.BRANCH_NAME}"								
							} catch(Exception e) {
								echo "Error en el analisis de calidad del repositorio de la rama ${env.BRANCH_NAME}"
							}
						}
					}
				}
				
				stage('SonarQube') {
					steps {
						echo "Iniciando Despliegue del repositorio de la rama ${env.BRANCH_NAME}"						
					}
				}
				
				stage('Unit Test') {
					stages {
					
						stage('Build') {
							steps {
								echo "Iniciando la compilacion docker de la rama ${env.BRANCH_NAME}"
							}
						}
						
						stage('Upload') {
							steps {
								echo "Push docker de la rama ${env.BRANCH_NAME}"								
							}
						}
					}
				}
			}
		}
	}
	
	post {
		always {
			echo "Pipeline finalizado del repositorio de la rama ${env.BRANCH_NAME} con el codigo de construccion ${env.BUILD_ID} en ${env.JENKINS_URL}"
		}
		success {
            echo 'La linea de construccion finalizo exitosamente'			
		}
        
        failure {
            echo "La linea de construccion finalizo con errores ${currentBuild.fullDisplayName} with ${env.BUILD_URL}"
        }
		
        unstable {
            echo 'La linea de construccion finalizo de forma inestable'
        }
	}
}

pipeline {
	agent {
		kubernetes {
			containerTemplate {
				name 'gradle'
				image 'gradle:4.10.0-jdk8'
				ttyEnabled true
				command 'cat'
			}
		}
	}
  
  /*triggers {
	pollSCM 'H/2 * * * *'
  }*/
  	
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
					sh 'gradle build -x check -x test'
				}else {
					echo "Iniciando construccion develop"
					sh 'gradle build -x check -x test'
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
								sh 'gradle clean test --no-daemon' //run a gradle task							
							} catch(Exception e) {
								echo "Error en el analisis de calidad del repositorio de la rama ${env.BRANCH_NAME}"
							} finally {
								junit '**/build/test-results/test/*.xml' //make the junit test results available in any case (success & failure)
							}
						}
					}
				}
				
				stage('SonarQube') {
					steps {
						echo "Iniciando Despliegue del repositorio de la rama ${env.BRANCH_NAME}"						
					}
				}
				
				/*stage('Unit Test') {
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
				}*/
			}
		}

		stage('Pre Build Docker') {
			steps {
				echo "Rama: ${env.BRANCH_NAME},  codigo de construccion: ${env.BUILD_ID} en ${env.JENKINS_URL}"
				echo "Iniciando limpieza"
			}		
		}

		stage('Build Docker') {
			steps {
				echo "Rama: ${env.BRANCH_NAME},  codigo de construccion: ${env.BUILD_ID} en ${env.JENKINS_URL}"
				echo "Iniciando limpieza "
			}		
		}

		stage('Deploy registry') {
			steps {
				echo "Rama: ${env.BRANCH_NAME},  codigo de construccion: ${env.BUILD_ID} en ${env.JENKINS_URL}"
				echo "Iniciando limpieza "
			}			
		}

		stage("Deploy stagging enviroment") {
            input {
                message "Should we deploy the project?"
            }         
            steps {
                echo "Desplegando entorno"
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

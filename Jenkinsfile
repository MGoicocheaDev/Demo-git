#!groovy
import groovy.json.JsonOutput
import java.time.*

/********* INICIO:: VARIABLES GLOBALES *********/
def projectProperties = [[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '15', artifactNumToKeepStr: '10', daysToKeepStr: '15', numToKeepStr: '10']],disableConcurrentBuilds()]


/********* INICIO:: EJECUCION **********/
node('windows-slave') {
	properties(projectProperties)
	try {
	    stage('Test'){
			bat "echo prueba pipelince from scm"
		}

	}
	catch(Exception ex){

	}
	finally {
	
	}
}

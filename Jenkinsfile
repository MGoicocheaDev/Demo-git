#!groovy
import groovy.json.JsonOutput
import java.time.*

/********* INICIO:: VARIABLES GLOBALES *********/
def projectProperties = [[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '15', artifactNumToKeepStr: '10', daysToKeepStr: '15', numToKeepStr: '10']],disableConcurrentBuilds()]
/***/
env.GIT_URL = 'https://steps.everis.com/git/MNTOREPSOL/Everilion-app0211.git'
def teamsChannel = '#git'

def msBuildPath = (/"C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\"/)
def	openCoverPath = (/"C:\Herramientas\OpenCover.4.6.519\tools\"/)
def nUnitpath = (/"C:\Herramientas\NUnit.ConsoleRunner.3.6.1\tools"/)
def reportPath = (/"C:\Herramientas\ReportGenerator.2.5.6\tools"/)


/** PROYECTO **/
def sq_project_code = 'app0211'
def solution_name = 'TPVBackOfficeDev.sln'
def version_solution = '15.0'
def sq_project_name = 'app0211-TPVBackOfficeDev'
def sq_project_name_ini = 'Peru-Repsol-RECOSAC-EVERILION-'
def nunit_project = ''
/***/
/** SONAR **/
def sq_project_name_full = sq_project_name_ini + sq_project_name
def SONAR_HOST_URL = 'http://52.36.35.246:9005/'
def credentialSonar = 'bc04db4217ba9e506974992e9aff1fdfc44f193a'
def sonarVersion = '4.5.0.1761';
/***/
def timeZone = TimeZone.getTimeZone('America/Lima')
def startDateTime = new Date().format( 'dd/MM/yyyy hh:mm:ss', timeZone )
def statusName = 'started'
def errorDescription = ''
def sq_project_version = new Date().format( 'yyyyMMddhhmmss', timeZone )
def branch_name = env.BRANCH_NAME
def path_workspace_project = "C:\\jenkins_slv\\workspace\\Peru\\Repsol\\Everilion\\${sq_project_code}\\${branch_name.replaceAll('/','-')}"
def filter_Coverage = "+[*]* -[*.Test*]*"
env.GIT_COMMIT_DESC = ""
env.context = ""
/********* FIN:: VARIABLES GLOBALES *********/

/********* INICIO:: EJECUCION **********/
node('windows-slave'){
	properties(projectProperties)
	try {
		stage('Checkout code'){
			env.context = 'Checkout code'
			checkout(path_workspace_project,branch_name)
			populateGlobalVariables (path_workspace_project)
		}
		stage("Notify-MSTeams-Start") {			
	        notifyTeams("${sq_project_code} -${solution_name}", 
	        	"${env.BUILD_NUMBER}", 
	        	statusColor("${statusName}"),
	        	"${env.BUILD_URL}",[
	        	[
	               name:"Branch",
	               value:"${branch_name}"
	            ],
	            [
	               name:"Status",
	               value:"${statusName}"
	            ],
	            [
	               name:"Start time",
	               value: startDateTime
	            ],
	            [
	               name:"Completion time",
	               value:"-"
	            ],
	            [
	               name:"Remarks",
	               value:"${env.GIT_COMMIT.substring(0,8)} - ${env.GIT_COMMIT_DESC}"
	            ],
	            [
	               name:"Developer",
	               value:"${env.GIT_AUTHOR_NAME}"
	            ]
	          ])
	    }
		stage('Build'){
			env.context = 'Build'
			build(path_workspace_project,solution_name,version_solution,msBuildPath)
		}
		/*/
		stage('Pruebas Unitarias'){
			env.context = 'Pruebas Unitarias'
			parallel 'Nunit':{
			    unitTest(nUnitpath,openCoverPath,reportPath,nunit_project,path_workspace_project)
			}, 'Coverage':{
			    coverage(nUnitpath,openCoverPath,reportPath,nunit_project,filter_Coverage,path_workspace_project)
			}
		}/*/
		stage('Sonar'){
			env.context = 'Sonar'
			sonarPreview(credentialSonar,
				sonarVersion,
				sq_project_name_full,
				sq_project_version,
				SONAR_HOST_URL,
				path_workspace_project,
				branch_name,
				solution_name,msBuildPath)
		}
		stage('Deploy'){
			env.context = 'Deploy'
			if(branch_name == 'develop'){
				develop(solution_name,path_workspace_project)
			}else if(branch_name == 'branch-Certificacion'){
				certificacion(solution_name,path_workspace_project)
			}
		}

		statusName = 'success'
		errorDescription = "Pipeline: ${env.JOB_NAME} ha finializado con exito"

	}
	catch(Exception ex){
		statusName = 'error'
		//errorDescription = ex.getClass()
		errorDescription = "Pipeline: ${env.JOB_NAME} ha presentado un error en el stage ${env.context}"
		//throw any
	}
	finally {
		/** Limpieza **/
		if(path_workspace_project != ''){
			bat "rd ${path_workspace_project} /S /Q"	
		}
		
		//cleanWS() --> no funciona

		/** Notifica Fin de ejecucion **/
		notifyTeams("${sq_project_code} -${solution_name}", 
	        	"${env.BUILD_NUMBER}", 
	        	statusColor("${statusName}"),
	        	"${env.BUILD_URL}",[
	        	[
	               name:"Branch",
	               value:"${branch_name}"
	            ],
	            [
	               name:"Status",
	               value:"success"
	            ],
	            [
	               name:"Start time",
	               value: startDateTime
	            ],
	            [
	               name:"Completion time",
	               value: new Date().format( 'dd/MM/yyyy hh:mm:ss', timeZone )
	            ],
	            [
	               name:"Remarks",
	               value:"${env.GIT_COMMIT.substring(0,8)} - ${env.GIT_COMMIT_DESC}"
	            ],
	            [
	               name:"Developer",
	               value:"${env.GIT_AUTHOR_NAME}"
	            ],
	            [
	               name:"Result",
	               value:"${errorDescription}"
	            ]
	          ])
	}
}
/********* INICIO:: EJECUCION **********/

/********* INICIO:: METODOS **********/

/******* INICIO:: CHECKOUT **********/
def checkout (path_workspace_project, branch_name) {
	echo 'checkout'

		dir(path_workspace_project){
			checkout([$class: 'GitSCM', 
			branches: [[name: '*/'+branch_name]], 
			doGenerateSubmoduleConfigurations: false, 
			extensions: [], 
			submoduleCfg: [], 
			userRemoteConfigs: [[
				credentialsId: 'genericoMaintMNTOREPSOL', 
				url: env.GIT_URL ]]])
		}
	
}
/******* FIN:: CHECKOUT **********/

/******* INICIO:: BUILD **********/
def build(path_workspace_project,solution_name,version_solution,msBuildPath){
	
	dir(path_workspace_project){
		env.msbuild14_path = "${msBuildPath}"
		bat "nuget restore ${solution_name}"
		withEnv(["PATH+msbuild_path=$msbuild14_path"]){
			bat "MSBuild.exe ${solution_name} "+
			"/p:VisualStudioVersion=${version_solution} " +
			"/p:Configuration=Debug " +
			"/p:Platform=\"Any CPU\" " +
			"/t:Rebuild"
			//bat "MSBuild.exe ${path_workspace_project}\\${solution_name} /p:VisualStudioVersion=${version_solution} /p:Configuration=Debug /p:Platform=\"Any CPU\" /t:Rebuild"
		}
	}
}
/******* FIN:: BUILD **********/

/******* INICIO::  NUNIT **********/
def uniTest(nunit,opencover_bin,reporte,nunit_project,path_workspace_project){
	
		env.opencover_bin = "${opencover_bin}"
		env.nunit = "${nunit}"
		env.reporte = "${reporte}"
	dir(path_workspace_project){
		withEnv(["PATH+nunit=$nunit", "PATH+opencover_bin=$opencover_bin", "PATH+reporte=$reporte"]) {
			bat "nunit3-console.exe Test\\${nunit_project}\\bin\\Debug\\${nunit_project}.dll " +
			"--result=${path_workspace_project}\\Nunit.Result.xml"
		}
	}
}
/******* FIN::  NUNIT **********/

/******* INICIO::  COVERAGE **********/
def coverage(nunit,opencover_bin,reporte,nunit_project,filter_Coverage,path_workspace_project){	
	
		env.opencover_bin = "${opencover_bin}"
		env.nunit = "${nunit}"
		env.reporte = "${reporte}"
	dir(path_workspace_project){
		withEnv(["PATH+nunit=$nunit", "PATH+opencover_bin=$opencover_bin", "PATH+reporte=$reporte"]) {
			bat "OpenCover.Console.exe " +
			"-target:nunit3-console.exe " +
			"-targetargs:\"Test\\${nunit_project}\\bin\\Debug\\${nunit_project}.dll " +
			"--result:${path_workspace_project}\\Nunit.Result.xml\" " +
			"-register:Path64 " +
			"-output:${path_workspace_project}\\Coverage.Result.xml " +
			"-filter:\"${filter_Coverage}\" "

			bat "ReportGenerator.exe "+
			"-reports:${path_workspace_project}\\Coverage.Result.xml "+
			"-targetdir:.build"

			publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: ".build", reportFiles: 'index.htm', reportName: 'Reporte de Cobertura', reportTitles: ''])
		}
	}
}
/******* FIN::  COVERAGE **********/

/******* INICIO::  SONAR **********/
def sonarPreview(sq_credentialSonar,
	sq_version_sonar,
	sq_project_name_full,
	sq_project_version,
	SONAR_HOST_URL,
	path_workspace_project,
	branch_name,
	solution_name,
	msBuildPath){
	dir(path_workspace_project){
		withSonarQubeEnv('SonarQube-IT&S-Enterprise') {
	        def sqScannerMsBuildHome = tool name: sq_version_sonar, type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'

	        bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe begin " + 
	        "/k:" + sq_project_name_full + 
	        " /n:" + sq_project_name_full + 
	        " /v:"+sq_project_version + 
	        " /d:sonar.log.level=DEBUG "+
	        " /d:sonar.host.url=%SONAR_HOST_URL% "+
	        " /d:sonar.sourceEncoding=UTF-8 " + 
	        " /d:sonar.profile=\"Sonar way\" "+
	        " /d:sonar.login=${sq_credentialSonar} "  + 
	        " /d:sonar.branch.name=${branch_name} " +
	        " /d:sonar.cs.opencover.reportsPaths=${path_workspace_project}\\Coverage.Result.xml" +
	        " /d:sonar.cs.nunit.reportsPaths=${path_workspace_project}\\Nunit.Result.xml"
	        bat "${msBuildPath}MSBuild.exe ${solution_name} /t:Rebuild"	        
	        bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe end /d:sonar.login=${sq_credentialSonar} "	        
	    }
	}
}
/******* FIN::  SONAR **********/

/******* INICIO:: DEPLOY *********/
def develop(solution_name,path_workspace_project){		
		precompiledProject(path_workspace_project,solution_name)
		bat "${path_workspace_project}\\CI-Deploy\\Deploy.exe ${path_workspace_project} develop"
}

def preproduction(branch_name){
	/** despliegue automatizado en desarrollo */
	/** branch => INTEGRACION **/
	env.context = "stage : Deploy to preproduction"

	if(branch_name == 'INTEGRACION'){
		stage 'Deploy-PreProduction'
		bat 'echo DESPLEGANDO EN PREPRODUCCION'
	}
}

def certificacion(solution_name,path_workspace_project){		
		precompiledProject(path_workspace_project,solution_name)
		bat "${path_workspace_project}\\CI-Deploy\\Deploy.exe ${path_workspace_project} certification"
}

def production(branch_name){
	/** despliegue automatizado en desarrollo */
	/** branch => master **/

	env.context = "stage : Deploy to production"

	if(branch_name == 'master'){
		stage 'Deploy-Production'
		
		bat 'echo DESPLEGANDO EN PRODUCCION'
	}
}

def precompiledProject(path_workspace_project,solution_name)
{
	bat "echo ---- Iniciando Precompilación del proyecto: ${solution_name}"
	withEnv(["PATH+msbuild_path=$msbuild14_path"]){
		bat("MSBuild.exe /t:Build ${path_workspace_project}\\${solution_name} /p:DeployOnBuild=true /p:PublishProfile=JenkinsCompiled-NotEdit.pubxml /p:Configuration=Release")
	}	
	bat "echo ---- Finalizando Precompilación del proyecto: ${solution_name}"
}

/******* FIN:: DEPLOY *********/

/******** INICIO:: GIT INFO ********/
def getGitAuthor(path_workspace_project) {
	dir(path_workspace_project){
		String GITCOMMIT = bat( script: '@echo off && git rev-parse HEAD', returnStdout: true).trim().tokenize(' ').last().trim()
		env.GIT_COMMIT=GITCOMMIT
		String GITAUTHOR = bat(script: "@echo off && git --no-pager show -s --pretty=\"format:%%an\" ${env.GIT_COMMIT}" ,returnStdout: true).trim()
		env.GIT_AUTHOR_NAME = GITAUTHOR.toString()
	}	
}

def getLastCommitMessage (path_workspace_project) {
	dir(path_workspace_project){
		String GITMESSAGE = bat(returnStdout: true, script: "@echo off && git log -1 --pretty=\"format:%%B\" ").trim()
		env.GIT_COMMIT_DESC = GITMESSAGE.toString()
		env.GIT_COMMIT_DESC = env.GIT_COMMIT_DESC.replaceAll('/','-')
		env.GIT_COMMIT_DESC = env.GIT_COMMIT_DESC.replaceAll('\\\\','_')
		//env.GIT_COMMIT_DESC = env.GIT_COMMIT_DESC.replaceAll (/\'/,/\\\'/)
		env.GIT_COMMIT_DESC = env.GIT_COMMIT_DESC.replaceAll("'","*")
	}
}

def populateGlobalVariables (path_workspace_project) {
    getLastCommitMessage(path_workspace_project)
    getGitAuthor(path_workspace_project)
    //testSummary = getTestSummary()
}
/******** FIN:: GIT INFO ********/

/******* INICIO: NOTIFICACION MS TEAMS *********/
def notifyTeams(codeProject, build, status,url,facts) {
    def teamsURL = 'https://outlook.office.com/webhook/ca1fbf99-086d-47fb-860e-4b900216f640@3048dc87-43f0-4100-9acb-ae1971c79395/JenkinsCI/b1e73466e1d84229b84e1057c1930ff9/40ff0248-9ede-4f0e-9727-05ea3a4d8c4c'
    
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payloadTeams = JsonOutput.toJson([
        type: "MessageCard",
        context: "http://schema.org/extensions",
        summary:"Integración Continua - Repsol Peru",   
        themeColor: status,
        title: "",
        sections : [
          [
            activityTitle : codeProject,
            activitySubtitle : "Latest status of build <strong>#${build}</strong>"
          ],
          [
            title: "Details:",
            facts: facts
          ]
        ],
        potentialAction: [
          [
            "@type": "ViewAction",
            "name":"View Build",
            "target":[
              url
            ]
          ]
        ]

      ])
   powershell "Invoke-RestMethod -Uri \'${teamsURL}\' -Method Post -Body \'${payloadTeams}\' -ContentType \'application/json\'"
}

def statusColor(status){
  def color = "";
    if(status == "started"){
        color = "428bca"
    }
    else if(status == "warning"){
      color = "fd7e14"
    }
    else if(status == "error"){
      color = "A50203"
    }
    else if(status == "success"){
      color = "5cb85c"
    }

  return color;
}
/******* FIN: NOTIFICACION MS TEAMS *********/

/********* FIN:: METODOS **********/

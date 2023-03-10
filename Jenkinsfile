@Library('flexy') _

// rename build
def private_ip_address = ""
def install = null
def tier_repo = "SachinNinganure"
def cluster_bld = ""
def scenario = "907272"
def userId = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId
def kraken_job = ""
def cerberus_job = ""
if (userId) {
  currentBuild.displayName = userId
}

pipeline {
  agent none

  parameters {
        string(name: 'BUILD_NUMBER', defaultValue: '', description: 'Build number of job that has installed the cluster.')
        string(name: 'JENKINS_AGENT_LABEL',defaultValue:'oc410 || oc411 || oc412')
        string(name: 'EGRESS_REPO', defaultValue:'https://github.com/SachinNinganure/Egress-Load-test', description:'You can change this to point to your fork if needed.')
        string(name: 'EGRESS_REPO_BRANCH', defaultValue:'main', description:'You can change this to point to a branch on your fork if needed.')
        text(name: 'ENV_VARS', defaultValue: '', description:'''<p>
               Enter list of additional (optional) Env Vars you'd want to pass to the script, one pair on each line. <br>
               e.g.<br>
               SOMEVAR1='env-test'<br>
               SOMEVAR2='env2-test'<br>
               ...<br>
               SOMEVARn='envn-test'<br>
               </p>'''
        )
        booleanParam(name: 'DEBUG', defaultValue: false)
        booleanParam(name: 'IPECHO', defaultValue: false)
	booleanParam(name: '4Projects', defaultValue: false)
	booleanParam(name: '200Projects', defaultValue: false)
        
        choice(
          choices: ["application-outages","container-scenarios","namespace-scenarios","network-scenarios","node-scenarios","pod-scenarios","node-cpu-hog","node-io-hog", "node-memory-hog", "power-outages","pvc-scenario","time-scenarios","zone-outages"], 
          name: 'KRAKEN_SCENARIO', 
          description: '''Type of kraken scenario to run'''
        )
        choice(
          choices: ["python","pod"], 
          name: 'KRAKEN_RUN_TYPE', 
          description: '''Type of way to run chaos scenario'''
        )
        string(
          name: 'ITERATIONS', 
          defaultValue: '', 
          description: 'Number of iterations to run of kraken scenario.'
        )
        string(
          name: 'KRAKEN_REPO', 
          defaultValue:'https://github.com/redhat-chaos/krkn', 
          description:'You can change this to point to your fork if needed.'
       )
        string(
          name: 'KRAKN_REPO_BRANCH', 
          defaultValue:'main', 
          description:'You can change this to point to a branch on your fork if needed.'
         )
        string(
          name: 'KRAKEN_HUB_REPO', 
          defaultValue:'https://github.com/redhat-chaos/krkn-hub', 
          description:'You can change this to point to your fork if needed.'
        )
       string(
        name: 'KRAKN_HUB_REPO_BRANCH', 
        defaultValue:'main', 
        description:'You can change this to point to a branch on your fork if needed.'
        )
         string(
          name: "PAUSE_TIME",
          defaultValue: "5",
          description: 'Amount of time to pause before running chaos scenarios'
        )
       string(
            name: "CERBERUS_WATCH_NAMESPACES", 
            defaultValue: "[^.*\$]",
            description: "Which specific namespaces you want to watch any failing components, use [^.*\$] if you want to watch all namespaces"
        )
       string(
            name: "CERBERUS_IGNORE_PODS",
            defaultValue: "[^installer*, ^kube-burner*, ^redhat-operators*, ^certified-operators*, ^collect-profiles*]", 
            description: "Which specific pod names regex patterns you want to ignore in the namespaces you defined above"
        )
       string(
            name: 'CERBERUS_ITERATIONS', 
            defaultValue: '', 
            description: 'Number of iterations to run of cerberus.'
        )
         
       text(
        name: 'ENV_VARS', 
        defaultValue: '',
        description:'''<p>
               Enter list of additional (optional) Env Vars you'd want to pass to the script, one pair on each line. <br>
               See https://github.com/redhat-chaos/krkn-hub/blob/main/docs/cerberus.md for list of variables to pass <br>
               e.g.<br>
               SOMEVAR1='env-test'<br>
               SOMEVAR2='env2-test'<br>
               ...<br>
               SOMEVARn='envn-test'<br>
               </p>'''
          )
       string(
            name: 'CERBERUS_REPO',
            defaultValue: 'https://github.com/redhat-chaos/cerberus',
            description: 'You can change this to point to your fork if needed.'
        )
       string(
            name: 'CERBERUS_REPO_BRANCH',
            defaultValue: 'main', 
            description: 'You can change this to point to a branch on your fork if needed.'
        ) 
    }
      


  stages {
    stage('Sequential'){
      agent { label params['JENKINS_AGENT_LABEL'] }
      stages{
        stage('Copy artifacts'){
          steps{
            copyArtifacts(
                filter: '',
                fingerprintArtifacts: true,
                projectName: 'ocp-common/Flexy-install',
                selector: specific(params.BUILD_NUMBER),
                target: 'flexy-artifacts'
            )
            script {
              buildinfo = readYaml file: "flexy-artifacts/BUILDINFO.yml"
              currentBuild.displayName = "${currentBuild.displayName}-${params.BUILD_NUMBER}-${currentBuild.number}"
              currentBuild.description = "Copying Artifact from Flexy-install build <a href=\"${buildinfo.buildUrl}\">Flexy-install#${params.BUILD_NUMBER}</a>"
              buildinfo.params.each { env.setProperty(it.key, it.value) }
            }
            script {   if (fileExists("flexy-artifacts/workdir/install-dir/cluster_info.json")){ 
                println "private_ip_address"
		private_ip_address = "cat flexy-artifacts/workdir/install-dir/cluster_info.json"
		println private_ip_address
                ENV_VARS += '\n' + private_ip_address
		sh label: '', script: '''
		echo "$ENV_VARS" > .env_override
		set -a && source .env_override && set +a
		private_ip_address=`grep INT_SVC_INSTANCE_INTERNAL_IP flexy-artifacts/workdir/install-dir/cluster_info.json|cut -d "," -f3-|cut -d ":" -f2-|sed 's/}//g'|cut -d "." -f1 |sed 's/-/./g'|cut -d "." -f2-`
                echo "$private_ip_address"
                echo "I am at the last of the logic, $private_ip_address"
		echo $private_ip_address >flexy-artifacts/workdir/install-dir/ipfile.txt
                '''
		private_ip_address = sh returnStdout: true, script: 'cat flexy-artifacts/workdir/install-dir/ipfile.txt'
		println private_ip_address
		println "now copying ip to ENV variable"
                println "printing the ENV variable "
                }
              }
          }
        }
                stage('ginko tests'){
		  when {
            		environment name: 'IPECHO', value: 'true'
         		 }

		  steps{
 	 	   script{	
			cluster_bld=BUILD_NUMBER
			install = build job:"ocp-common/ginkgo-test/", propagate: false, parameters:[
                        string(name: "SCENARIO", value: scenario),
                        string(name: "FLEXY_BUILD", value: cluster_bld),
                        string(name: "TIERN_REPO_OWNER", value: tier_repo),
 		]
                      }
                   }
                 }		
        stage('Checkout repo'){
          steps{
            dir('Egress-Load-test'){
              git branch: params.EGRESS_REPO_BRANCH, url: params.EGRESS_REPO
            }
          }
        }
        stage('Debug info'){
          when {
            environment name: 'DEBUG', value: 'true'
          }
          steps{
            ansiColor('xterm') {
              sh label: '', script: '''
              # Get ENV VARS Supplied by the user to this job and store in .env_override
              echo "$ENV_VARS" > .env_override
              # Export those env vars so they could be used by CI Job
              set -a && source .env_override && set +a
              mkdir -p ~/.kube
              cp $WORKSPACE/flexy-artifacts/workdir/install-dir/auth/kubeconfig ~/.kube/config
              ls -ls ~/.kube/
              env
              oc version
              oc project default
              ansible --version
              python --version
              python3 --version
              whoami
              '''
            }
          }
        }

       stage("Run parallel tests") {	
	parallel {
 	    stage("Check cluster health") {
                agent { label params['JENKINS_AGENT_LABEL'] }
                steps {
                    script {
                           cluster_bld=BUILD_NUMBER
			   cerberus_job = build job: 'scale-ci/e2e-benchmarking-multibranch-pipeline/cerberus',
                            parameters: [
                                string(name: 'BUILD_NUMBER', value: cluster_bld),text(name: "ENV_VARS", value: ENV_VARS),
                                string(name: "CERBERUS_WATCH_NAMESPACES", value: CERBERUS_WATCH_NAMESPACES),
                                string(name: 'CERBERUS_IGNORE_PODS', value: CERBERUS_IGNORE_PODS),string(name: 'CERBERUS_ITERATIONS', value: CERBERUS_ITERATIONS),
                                string(name: 'JENKINS_AGENT_LABEL', value: JENKINS_AGENT_LABEL),booleanParam(name: "INSPECT_COMPONENTS", value: false),
                                string(name: "ENV_VARS", value: ENV_VARS)
                            ],
                            propagate: false
                        currentBuild.description += """
                            <b>Cerberus Job: </b> <a href="${cerberus_job.absoluteUrl}"> ${params.CERBERUS_ITERATIONS} iterations were ran</a> <br/>
                        """
                    }
                }
            }             



	   stage('Run egress script to test the egress functionality- Projects=4'){
              when {
                        environment name: '4Projects', value: 'true'
                         }
	     steps{
               ansiColor('xterm') {
               sh label: '', script: '''
               # Get ENV VARS Supplied by the user to this job and store in .env_override
               echo "$ENV_VARS" > .env_override
               # Export those env vars so they could be used by CI Job
               set -a && source .env_override && set +a
               mkdir -p ~/.kube
               cp $WORKSPACE/flexy-artifacts/workdir/install-dir/auth/kubeconfig ~/.kube/config
               ls -la
               cd Egress-Load-test 
	       pwd
               ./run.sh_4p
               '''
             }
           }
         }

           stage('Run egress script to test the egress functionality- Projects=200'){
              when {
                        environment name: '200Projects', value: 'true'
                         }
	      steps{
                 ansiColor('xterm') {
                 sh label: '', script: '''
                 # Get ENV VARS Supplied by the user to this job and store in .env_override
                 echo "$ENV_VARS" > .env_override
                 # Export those env vars so they could be used by CI Job
                 set -a && source .env_override && set +a
                 mkdir -p ~/.kube
                 cp $WORKSPACE/flexy-artifacts/workdir/install-dir/auth/kubeconfig ~/.kube/config
                 ls -la
                 cd Egress-Load-test
                 pwd
                 ./run.sh_200p
                 '''
            }
          }
        }
	   
           stage("Start Kraken run") {
                agent { label params['JENKINS_AGENT_LABEL'] }
                steps {
                    script {
                            cluster_bld=BUILD_NUMBER
			    kraken_job = build job: 'scale-ci/e2e-benchmarking-multibranch-pipeline/kraken',
                            parameters: [
                                string(name: 'BUILD_NUMBER', value: cluster_bld),text(name: "ENV_VARS", value: ENV_VARS),
                                string(name: "KRAKEN_REPO", value: KRAKEN_REPO),string(name: "KRAKN_REPO_BRANCH", value: KRAKN_REPO_BRANCH),
                                string(name: "KRAKEN_HUB_REPO", value: KRAKEN_HUB_REPO),string(name: "KRAKN_HUB_REPO_BRANCH", value: KRAKN_HUB_REPO_BRANCH),
                                string(name: 'KRAKEN_SCENARIO', value: KRAKEN_SCENARIO),string(name: "KRAKEN_RUN_TYPE", value: KRAKEN_RUN_TYPE),
                                string(name: 'JENKINS_AGENT_LABEL', value: JENKINS_AGENT_LABEL),string(name: "PAUSE_TIME", value: PAUSE_TIME),
                                string(name: "ITERATIONS", value: ITERATIONS),string(name: "ENV_VARS", value: ENV_VARS)
                            ],
                            propagate: false
                            currentBuild.description += """
                            <b>Kraken Job: </b> <a href="${kraken_job.absoluteUrl}"> ${KRAKEN_SCENARIO} </a> <br/>
                        """
                    }
                }
            }

           stage('Set status') {
                agent { label params['JENKINS_AGENT_LABEL'] }
                steps {
                    script{
                    def status = ""
                if ( kraken_job != "" ) {
                    if (status != "" ) {
                        status += ","
                    }
                if ( kraken_job.result.toString() != "SUCCESS" ){
                        status += "Kraken Failed"
                        currentBuild.result = "FAILURE"
                    } else {
                        status += "Kraken Passed"
                                          }
                                      }
            if ( cerberus_job != "" ) {
                    if (status != "" ) {
                        status += ","
                    }
                    if ( cerberus_job.result.toString() != "SUCCESS" ) {
                        if (status != "" ) {
                        status += ","
                        }
                        status += "Cerberus Failed"
                        currentBuild.result = "FAILURE"
                    } else { 
                        status += "Cerberus Passed"
                    }
                }
                currentBuild.description += """
                    <b>Final Status: </b> ${status} <br/>
                """
                                     }
                         }
                 }


        }
     }




        stage('Archive Artifacts') {
          steps {
                  sh 'pwd'
		  archiveArtifacts artifacts: 'Egress-Load-test/*', fingerprint: true
                }
            }

        }
      }
    }
}

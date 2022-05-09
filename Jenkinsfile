
def aws = null
def install = null
def scale_up = null
def loaded_ci = null
def health_check = null
def upgrade_ci = null
def destroy_ci = null
def must_gather = null
def build_string = "DEFAULT"
def load_result = "SUCCESS"
def loaded_url = ""
def upgrade_url = ""
def must_gather_url = ""
def proxy_settings = ""
def status = "PASS"
def VERSION = ""
def global_scale_num = 0

def userId = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId
if (userId) {
  currentBuild.displayName = "${userId}-${currentBuild.displayName}"
}

pipeline{
    agent any

    parameters {
        separator(name: "PRE_BUILT_FLEXY_ENV", sectionHeader: "Pre Built Flexy", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'BUILD_NUMBER', defaultValue: '', description: 'Build number of job that has installed the cluster.')

        separator(name: "BUILD_FLEXY_COMMON_PARAMS", sectionHeader: "Build Flexy Parameters Common", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'OCP_PREFIX', defaultValue: '', description: 'Name of ocp cluster you want to build')
        string(name: 'OCP_VERSION', defaultValue: '', description: 'Build version to install the cluster.')
        separator(name: "BUILD_FLEXY_FROM_PROFILE", sectionHeader: "Build Flexy From Profile", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'CI_PROFILE', defaultValue: '', description: 'Name of ci profile to build for the cluster you want to build')
        choice(choices: ['extra-small','small','medium',''], name: 'PROFILE_SCALE_SIZE', description: 'Size of cluster to scale to; will be ignored if SCALE_UP is set')
        separator(name: "BUILD_FLEXY", sectionHeader: "Build Flexy Parameters", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        choice(choices: ['','aws', 'azure', 'gcp', 'osp', 'alicloud', 'ibmcloud', 'vsphere', 'ash'], name: 'CLOUD_TYPE', description: '''Cloud type (As seen on https://gitlab.cee.redhat.com/aosqe/flexy-templates/-/tree/master/functionality-testing/aos-4_9, after ""-on-") <br/>
        Will be ignored if BUILD_NUMBER is set''')
        choice(choices: ['','ovn', 'sdn'], name: 'NETWORK_TYPE', description: 'Network type, will be ignored if BUILD_NUMBER is set')
        choice(choices: ['','ipi', 'upi', 'sno'], name: 'INSTALL_TYPE', description: '''Type of installation (set to SNO for sno cluster type),  <br/>
        will be ignored if BUILD_NUMBER is set''')
        string(name: 'MASTER_COUNT', defaultValue: '3', description: 'Number of master nodes in your cluster to create.')
        string(name: "WORKER_COUNT", defaultValue: '3', description: 'Number of worker nodes in your cluster to create.')

        separator(name: "SCALE_UP_JOB_INFO", sectionHeader: "Scale Up Job Options", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
	    booleanParam(name: 'INFRA_WORKLOAD_INSTALL', defaultValue: false, description: 'Install workload and infrastructure nodes even if less than 50 nodes. <br> Checking this parameter box is valid only when SCALE_UP is greater than 0.')
        string(name: 'SCALE_UP', defaultValue: '0', description: 'If value is set to anything greater than 0, cluster will be scaled up before executing the workload.')
        string(name: 'SCALE_DOWN', defaultValue: '0', description:
        '''If value is set to anything greater than 0, cluster will be scaled down after the execution of the workload is complete,<br>
        if the build fails, scale down may not happen, user should review and decide if cluster is ready for scale down or re-run the job on same cluster.'''
        )
        separator(name: "SCALE_CI_JOB_INFO", sectionHeader: "Scale-CI Job Options", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        choice(choices: ["","cluster-density","pod-density","node-density","node-density-heavy","etcd-perf","max-namespaces","max-services","concurrent-builds","pod-network-policy-test","router-perf","network-perf-hostnetwork-network-test","network-perf-pod-network-test","network-perf-serviceip-network-test"], name: 'CI_TYPE', description: '''Type of scale-ci job to run. Can be left blank to not run ci job <br>
        Router-perf tests will use all defaults if selected, all parameters in this section below will be ignored ''')


        string(name: 'VARIABLE', defaultValue: '1000', description: '''
        This variable configures parameter needed for each type of workload. By default 1000. <br>
        pod-density: This will export PODS env variable; set to 200 * num_workers, work up to 250 * num_workers. Creates as many "sleep" pods as configured in this environment variable. <br>
        cluster-density: This will export JOB_ITERATIONS env variable; set to 4 * num_workers. This variable sets the number of iterations to perform (1 namespace per iteration). <br>
        max-namespaces: This will export NAMESPACE_COUNT env variable; set to ~30 * num_workers. The number of namespaces created by Kube-burner. <br>
        max-services: This will export SERVICE_COUNT env variable; set to 200 * num_workers, work up to 250 * num_workers. Creates n-replicas of an application deployment (hello-openshift) and a service in a single namespace. <br>
        node-density: This will export PODS_PER_NODE env variable; set to 200, work up to 250. Creates as many "sleep" pods as configured in this variable - existing number of pods on node. <br>
        node-density-heavy: This will export PODS_PER_NODE env variable; set to 200, work up to 250. Creates this number of applications proportional to the calculated number of pods / 2 <br>
        Read here for detail of each variable: <br>
        https://github.com/cloud-bulldozer/e2e-benchmarking/blob/master/workloads/kube-burner/README.md <br>
        ''')

        separator(name: "NODE_DENSITY_JOB_INFO", sectionHeader: "Node Density Job Options", sectionHeaderStyle: """
				font-size: 14px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'NODE_COUNT', defaultValue: '3', description: 'Number of worker nodes to be used in your cluster for this workload.')
        separator(name: "CONCURRENT_BUILDS_JOB_INFO", sectionHeader: "Concurrent Builds Job Options", sectionHeaderStyle: """
				font-size: 14px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'BUILD_LIST', defaultValue: "1 8 15 30 45 60 75", description: 'Number of concurrent builds to run at a time; will run 2 iterations of each number in this list')
        string(name: 'APP_LIST', defaultValue: 'cakephp eap django nodejs', description: 'Applications to build, will run each of the concurrent builds against each application. Best to run one application at a time')
        separator(name: "NETWORK_PERF_INFO", sectionHeader: "Network-Perf Job Options", sectionHeaderStyle: """
				font-size: 14px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        choice(choices: ['smoke', 'pod2pod', 'hostnet', 'pod2svc'], name: 'WORKLOAD_TYPE', description: 'Workload type')
        booleanParam(name: "NETWORK_POLICY", defaultValue: false, description: "If enabled, benchmark-operator will create a network policy to allow ingress trafic in uperf server pods")

        separator(name: "UPGRADE_INFO", sectionHeader: "Upgrade Options", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")
        string(name: 'UPGRADE_VERSION', description: 'This variable sets the version number you want to upgrade your OpenShift cluster to (can list multiple by separating with comma, no spaces).')
        booleanParam(name: 'EUS_UPGRADE', defaultValue: false, description: '''This variable will perform an EUS type upgrade <br>
        See "https://docs.google.com/document/d/1396VAUFLmhj8ePt9NfJl0mfHD7pUT7ii30AO7jhDp0g/edit#heading=h.bv3v69eaalsw" for how to run
        ''')
        choice(choices: ['fast', 'eus', 'candidate', 'stable'], name: 'EUS_CHANNEL', description: 'EUS Channel type, will be ignored if EUS_UPGRADE is not set to true')

        booleanParam(name: 'ENABLE_FORCE', defaultValue: true, description: 'This variable will force the upgrade or not')
        booleanParam(name: 'SCALE', defaultValue: false, description: 'This variable will scale the cluster up one node at the end up the upgrade')
        string(name: 'MAX_UNAVAILABLE', defaultValue: "1", description: 'This variable will set the max number of unavailable nodes during the upgrade')

        separator(name: "GENERAL_BUILD_INFO", sectionHeader: "General Options", sectionHeaderStyle: """
				font-size: 18px;
				font-weight: bold;
				font-family: 'Orienta', sans-serif;
			""")

        booleanParam(name: 'WRITE_TO_FILE', defaultValue: true, description: 'Value to write to google sheet (will run https://mastern-jenkins-csb-openshift-qe.apps.ocp4.prod.psi.redhat.com/job/scale-ci/job/paige-e2e-multibranch/job/write-to_sheet)')
        booleanParam(name: 'DESTROY_WHEN_DONE', defaultValue: 'False', description: 'If you want to destroy the cluster created at the end of your run ')
        string(name:'JENKINS_AGENT_LABEL',defaultValue:'oc45',description:
        '''
        scale-ci-static: for static agent that is specific to scale-ci, useful when the jenkins dynamic agent isn't stable<br>
        4.y: oc4y || mac-installer || rhel8-installer-4y <br/>
            e.g, for 4.8, use oc48 || mac-installer || rhel8-installer-48 <br/>
        3.11: ansible-2.6 <br/>
        3.9~3.10: ansible-2.4 <br/>
        3.4~3.7: ansible-2.4-extra || ansible-2.3 <br/>
        '''
        )
        text(name: 'ENV_VARS', defaultValue: '', description:'''<p>
               Enter list of additional (optional) Env Vars you'd want to pass to the script, one pair on each line. <br>
               e.g.<br>
               SOMEVAR1='env-test'<br>
               SOMEVAR2='env2-test'<br>
               ...<br>
               SOMEVARn='envn-test'<br>
               </p>'''
            )
        string(name: 'E2E_BENCHMARKING_REPO', defaultValue:'https://github.com/cloud-bulldozer/e2e-benchmarking', description:'You can change this to point to your fork if needed.')
        string(name: 'E2E_BENCHMARKING_REPO_BRANCH', defaultValue:'master', description:'You can change this to point to a branch on your fork if needed.')
        string(name: "CI_PROFILES_URL",defaultValue: "https://gitlab.cee.redhat.com/aosqe/ci-profiles.git/",description:"Owner of ci-profiles repo to checkout, will look at folder 'scale-ci/\${major_v}.\${minor_v}'")
        string(name: "CI_PROFILES_REPO_BRANCH", defaultValue: "master", description: "Branch of ci-profiles repo to checkout" )
    }


    stages{
        stage("Build Flexy Clusters") {
           agent { label params['JENKINS_AGENT_LABEL'] }
           steps {
                // checkout CI profile repo from GitLab
                checkout changelog: false,
                    poll: false,
                    scm: [
                        $class: 'GitSCM',
                        branches: [[name: "${params.CI_PROFILES_REPO_BRANCH}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [
                            [$class: 'CloneOption', noTags: true, reference: '', shallow: true],
                            [$class: 'PruneStaleBranch'],
                            [$class: 'CleanCheckout'],
                            [$class: 'IgnoreNotifyCommit'],
                            [$class: 'RelativeTargetDirectory', relativeTargetDir: 'local-ci-profiles']
                        ],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            name: 'origin',
                            refspec: "+refs/heads/${params.CI_PROFILES_REPO_BRANCH}:refs/remotes/origin/${params.CI_PROFILES_REPO_BRANCH}",
                            url: "${params.CI_PROFILES_URL}"
                        ]]
                    ]
                script{
                    def install_type_custom = params.INSTALL_TYPE
                    def custom_cloud_type = params.CLOUD_TYPE
                    def custom_jenkins_label = JENKINS_AGENT_LABEL
                    if (params.SCALE_UP.toInteger() > 0 ) {
                        global_scale_num = params.SCALE_UP.toInteger()
                    }
                    println "global num $global_scale_num"
                     if(params.BUILD_NUMBER == "") {
                        VERSION = params.OCP_VERSION
                        println "${VERSION}"
                        def version_list = VERSION.tokenize(".")
                        println "version ${version_list}"
                        def major_v = version_list[0]
                        def minor_v = version_list[1]
                        def extra_launcher_vars = ''
                        println "major ${major_v} minor ${minor_v}"
                        def var_loc = ""
                        if(params.CI_PROFILE != "") {
                            installData = readYaml(file: "local-ci-profiles/scale-ci/${major_v}.${minor_v}/${params.CI_PROFILE}.install.yaml")
                            installData.install.flexy.each { env.setProperty(it.key, it.value) }
                            println "PROFILE_SCALE_SIZE ${params.PROFILE_SCALE_SIZE}"
                            // loop through install data keys to make sure scale is one of them
                            for (data in installData) {
                                if (data.key == "scale" ) {
                                    data.value.get(params.PROFILE_SCALE_SIZE).each { env.setProperty(it.key, it.value) }
                                    break
                                }
                            }

                            var_loc = env.VARIABLES_LOCATION
                            if (env.EXTRA_LAUNCHER_VARS != null ) {
                                extra_launcher_vars = env.EXTRA_LAUNCHER_VARS  + "\n"
                            }
                            println "extra lanch vars ${extra_launcher_vars}"
                            println "env scale ${env.SCALE_UP}"
                            println "scale ${SCALE_UP}"
                            if (global_scale_num.toInteger() == 0 && env.SCALE_UP.toInteger() > 0 ) {
                                global_scale_num = env.SCALE_UP.toInteger()
                            }

                            currentBuild.description = """
                                <b>CI Profile:</b> ${params.CI_PROFILE}<br/>
                            """
                        }
                         else {
                          def network_ending = ""

                          if (params.CLOUD_TYPE == "vsphere") {
                                network_ending = "-vmc7"
                          }
                           if (params.NETWORK_TYPE != "sdn") {
                            if (params.CLOUD_TYPE == "alicloud") {
                                network_ending = "-fips-${params.NETWORK_TYPE}-ci"
                            } else if (params.CLOUD_TYPE != "ash" ) {
                                network_ending += "-${params.NETWORK_TYPE}"
                            }
                           }
                            def worker_type = ""
                                if (params.CLOUD_TYPE == "aws") {
                                if (params.INSTALL_TYPE == "sno") {
                                    extra_launcher_vars = "master_worker_AllInOne: 'true'\nnum_masters: 1\nnum_workers: 0\nvm_type: 'm5.4xlarge'\n"
                                    install_type_custom = "ipi"
                                } else {
                                extra_launcher_vars = "vm_type_workers: 'm5.xlarge'\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                                }
                            }
                            else if (params.CLOUD_TYPE == "azure") {
                                extra_launcher_vars = "vm_type_workers: 'Standard_D8s_v3'\nregion: centralus\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                            }
                            else if (params.CLOUD_TYPE == "gcp") {
                                extra_launcher_vars = "vm_type_workers: 'n1-standard-4'\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                                if (params.NETWORK_TYPE != "sdn") {
                                 network_ending = network_ending + "-ci"
                                }
                            }
                            else if (params.CLOUD_TYPE == "osp") {
                                extra_launcher_vars = "vm_type_workers: 'ci.m1.xlarge'\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                            }
                            else if (params.CLOUD_TYPE == "alicloud") {

                                extra_launcher_vars = "vm_type_workers: 'ecs.g6.xlarge'\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                            }
                            else if (params.CLOUD_TYPE == "ibmcloud") {
                                extra_launcher_vars = "vm_type_workers: 'bx2d-4x16'\nregion: 'jp-tok'\nnum_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                            }
                            else if (params.CLOUD_TYPE == "vsphere") {
                                extra_launcher_vars = " num_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                            } else if (params.CLOUD_TYPE == "ash") {
                                custom_cloud_type = "azure"
                                extra_launcher_vars = " num_workers: " + WORKER_COUNT + "\nnum_masters: " + MASTER_COUNT + "\n"
                                if (params.NETWORK_TYPE != "sdn") {
                                extra_launcher_vars += 'networkType: "OVNKubernetes"\n'
                                }
                                network_ending = "-ash_wwt"
                                custom_jenkins_label = "fedora-installer-wwt"
                            }
                             var_loc = "private-templates/functionality-testing/aos-${major_v}_${minor_v}/${install_type_custom}-on-${custom_cloud_type}/versioned-installer${network_ending}"

                             currentBuild.description = """
                                    <b>Cluster Built:</b> ${install_type_custom} ${custom_cloud_type} ${params.NETWORK_TYPE}<br/>
                                """
                            }

                            install = build job:"ocp-common/Flexy-install", propagate: false, parameters:[
                                string(name: "INSTANCE_NAME_PREFIX", value: OCP_PREFIX),string(name: "VARIABLES_LOCATION", value: "${var_loc}"),
                                string(name: "JENKINS_AGENT_LABEL", value: custom_jenkins_label),text(name: "LAUNCHER_VARS",
                                value: "${extra_launcher_vars}installer_payload_image: 'registry.ci.openshift.org/ocp/release:${VERSION}'"),
                                text(name: "BUSHSLICER_CONFIG", value: ''),text(name: 'REPOSITORIES', value: '''
GIT_PRIVATE_URI=git@gitlab.cee.redhat.com:aosqe/cucushift-internal.git
GIT_PRIVATE_TEMPLATES_URI=https://gitlab.cee.redhat.com/aosqe/flexy-templates.git'''),
text(name: 'CREDENTIALS', value: '''
DYNECT_CREDENTIALS=b1666c61-4a76-40b7-950f-a3d40f721e59
REG_STAGE=41c2dd39-aad7-4f07-afec-efc052b450f5
REG_QUAY=c1802784-0f74-4b35-99fb-32dfa9a207ad
REG_CLOUD=fba37700-62f8-4883-8905-53f86461ba5b
REG_CONNECT=819c9e9f-1e9c-4d2b-9dc0-fc630674bc9b
REG_REDHAT=819c9e9f-1e9c-4d2b-9dc0-fc630674bc9b
REG_BREW_OSBS=brew-registry-osbs-mirror
REG_NIGHTLY_BUILDS=9a9187c6-a54c-452a-866f-bea36caea6f9
REG_CI_BUILDS=registry.ci.openshift.org
GIT_FLEXY_SSH_KEY=e2f7029f-ab8d-4987-8950-39feb80d5fbd
GIT_PRIVATE_SSH_KEY=1d2207b6-15c0-4cb0-913a-637788d12257
REG_SVC_CI=9a9187c6-a54c-452a-866f-bea36caea6f9
                                ''' )
                        ]
                        if( install.result.toString()  != "SUCCESS") {
                           println "build failed"
                           currentBuild.result = "FAILURE"
                           status = "Install failed"
                        }
                    } else {
                     copyArtifacts(
                        filter: '',
                        fingerprintArtifacts: true,
                        projectName: 'ocp-common/Flexy-install',
                        selector: specific(params.BUILD_NUMBER),
                        target: 'flexy-artifacts'
                       )
                        installData = readYaml(file: "flexy-artifacts/workdir/install-dir/cluster_info.yaml")
                        VERSION = installData.INSTALLER.VERSION
                        currentBuild.description = """
                            <b>Using Pre-Built Flexy</b> <br/>
                        """
                     }

                    if (params.BUILD_NUMBER != "") {
                            build_string = params.BUILD_NUMBER
                    } else if( install.result.toString() == "SUCCESS" ) {
                            build_string = install.number.toString()
                    }
                    currentBuild.description += """
                        <b>Version:</b> ${VERSION}<br/>
                        <b>Flexy-install Build:</b> ${build_string}<br/>
                    """
                 if ( build_string != "DEFAULT") {
                     copyArtifacts(
                        fingerprintArtifacts: true,
                        projectName: 'ocp-common/Flexy-install',
                        selector: specific(build_string),
                        filter: "workdir/install-dir/",
                        target: 'flexy-artifacts'
                       )
                    if (fileExists("flexy-artifacts/workdir/install-dir/client_proxy_setting.sh")) {
                     println "yes"
                     proxy_settings = sh returnStdout: true, script: 'cat flexy-artifacts/workdir/install-dir/client_proxy_setting.sh'
                     proxy_settings = proxy_settings.replace('export ', '')
                    }

                    ENV_VARS += '\n' + proxy_settings
                    println "$ENV_VARS $global_scale_num"

                 }
               }
            }
        }
        stage("Scale Up Cluster") {
           agent { label params['JENKINS_AGENT_LABEL'] }
            when {
                expression { build_string != "DEFAULT" && status == "PASS" && global_scale_num.toInteger() > 0}
            }
           steps {
            script{

                scale_up = build job: 'scale-ci/e2e-benchmarking-multibranch-pipeline/cluster-workers-scaling/', parameters: [
                    string(name: 'BUILD_NUMBER', value: "${build_string}"), text(name: "ENV_VARS", value: ENV_VARS),
                    booleanParam(name: 'INFRA_WORKLOAD_INSTALL', value: INFRA_WORKLOAD_INSTALL),
                    string(name: 'WORKER_COUNT', value: global_scale_num.toString()), string(name: 'JENKINS_AGENT_LABEL', value: JENKINS_AGENT_LABEL)
                ]
                currentBuild.description += """
                    <b>Scaled Up:</b>  <a href="${scale_up.absoluteUrl}"> ${global_scale_num} </a><br/>
                """
                if( scale_up != null && scale_up.result.toString() != "SUCCESS") {
                   status = "Scale Up Failed"
                   currentBuild.result = "FAILURE"
                }
            }
          }
        }
        stage("Perf Testing"){
            agent { label params['JENKINS_AGENT_LABEL'] }
            when {
                expression { build_string != "DEFAULT" && status == "PASS" }
            }
            steps{
                script {
                  if( ["cluster-density","pod-density","node-density","node-density-heavy", "max-namespaces","max-services","concurrent-builds","pod-density-heavy"].contains(params.CI_TYPE) ) {
                    loaded_ci = build job: "scale-ci/e2e-benchmarking-multibranch-pipeline/kube-burner", propagate: false, parameters:[
                        string(name: "BUILD_NUMBER", value: "${build_string}"),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),
                        string(name: "WORKLOAD", value: CI_TYPE),string(name: "VARIABLE", value: VARIABLE),string(name: 'NODE_COUNT', value: NODE_COUNT),
                        string(name: "BUILD_LIST", value: BUILD_LIST),string(name: 'APP_LIST', value: APP_LIST),
                        text(name: "ENV_VARS", value: ENV_VARS),booleanParam(name: "WRITE_TO_FILE", value: WRITE_TO_FILE),
                        string(name: "E2E_BENCHMARKING_REPO", value: E2E_BENCHMARKING_REPO),
                        string(name: "E2E_BENCHMARKING_REPO_BRANCH", value: E2E_BENCHMARKING_REPO_BRANCH)
                    ]
                    currentBuild.description += """
                        <b>Scale-Ci: Kube-burner </b> ${CI_TYPE}- ${VARIABLE} <br/>
                        <b>Scale-CI Job: </b> <a href="${loaded_ci.absoluteUrl}"> ${loaded_ci.getNumber()} </a> <br/>
                    """
                   } else if (params.CI_TYPE == "etcd-perf") {
                       loaded_ci = build job: "scale-ci/e2e-benchmarking-multibranch-pipeline/etcd-perf", propagate: false, parameters:[
                            string(name: "BUILD_NUMBER", value: "${build_string}"),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),
                            text(name: "ENV_VARS", value: ENV_VARS),string(name: "E2E_BENCHMARKING_REPO", value: E2E_BENCHMARKING_REPO),
                            string(name: "E2E_BENCHMARKING_REPO_BRANCH", value: E2E_BENCHMARKING_REPO_BRANCH),
                            booleanParam(name: "WRITE_TO_FILE", value: WRITE_TO_FILE)
                       ]
                       currentBuild.description += """
                            <b>Scale-Ci: </b> etcd-perf <br/>
                            <b>Scale-CI Job: </b> <a href="${loaded_ci.absoluteUrl}"> ${loaded_ci.getNumber()} </a> <br/>
                        """
                   } else if (params.CI_TYPE == "router-perf") {
                       loaded_ci = build job: "scale-ci/e2e-benchmarking-multibranch-pipeline/router-perf",propagate: false, parameters:[
                            string(name: "BUILD_NUMBER", value: "${build_string}"),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),
                            text(name: "ENV_VARS", value: ENV_VARS),string(name: "E2E_BENCHMARKING_REPO", value: E2E_BENCHMARKING_REPO),
                            string(name: "E2E_BENCHMARKING_REPO_BRANCH", value: E2E_BENCHMARKING_REPO_BRANCH),
                            booleanParam(name: "WRITE_TO_FILE", value: WRITE_TO_FILE)
                       ]
                        currentBuild.description += """
                            <b>Scale-Ci: </b> router-perf <br/>
                            <b>Scale-CI Job: </b> <a href="${loaded_ci.absoluteUrl}"> ${loaded_ci.getNumber()} </a> <br/>
                        """
                   }else if ( ["network-perf-pod-network-test","network-perf-serviceip-network-test","network-perf-hostnetwork-network-test"].contains(params.CI_TYPE) ) {
                       loaded_ci = build job: "scale-ci/paige-e2e-multibranch/network-perf", propagate: false, parameters:[
                            string(name: "BUILD_NUMBER", value: "${build_string}"),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),
                            string(name: "WORKLOAD_TYPE", value: WORKLOAD_TYPE),booleanParam(name: "NETWORK_POLICY", value: NETWORK_POLICY),
                            text(name: "ENV_VARS", value: ENV_VARS),string(name: "E2E_BENCHMARKING_REPO", value: E2E_BENCHMARKING_REPO),
                            string(name: "E2E_BENCHMARKING_REPO_BRANCH", value: E2E_BENCHMARKING_REPO_BRANCH),
                            booleanParam(name: "WRITE_TO_FILE", value: WRITE_TO_FILE)
                         ]
                       currentBuild.description += """
                            <b>Scale-Ci: Network Perf </b> ${WORKLOAD_TYPE} ${NETWORK_POLICY} <br/>
                            <b>Scale-CI Job: </b> <a href="${loaded_ci.absoluteUrl}"> ${loaded_ci.getNumber()} </a> <br/>
                       """
                    }else{
                        println "No Scale-ci Job"
                        currentBuild.description += """
                            <b>No Scale-Ci Run</b><br/>
                        """
                    }

                    if( loaded_ci != null ) {
                      if( loaded_ci.result.toString() != "SUCCESS") {
                           status = "Scale CI Job Failed"
                           currentBuild.result = "FAILURE"
                        }
                    }

                }
            }
        }
        stage('Upgrade'){
            agent { label params['JENKINS_AGENT_LABEL'] }
            when {
                expression { build_string != "DEFAULT" && status == "PASS" && UPGRADE_VERSION != ""  }
            }
            steps{
                script{
                    currentBuild.description += """
                        <b>Upgrade to: </b> ${UPGRADE_VERSION} <br/>
                    """
                    upgrade_ci = build job: "scale-ci/e2e-benchmarking-multibranch-pipeline/upgrade", propagate: false,parameters:[
                        string(name: "BUILD_NUMBER", value: build_string),string(name: "MAX_UNAVAILABLE", value: MAX_UNAVAILABLE),
                        string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),string(name: "UPGRADE_VERSION", value: UPGRADE_VERSION),
                        booleanParam(name: "EUS_UPGRADE", value: EUS_UPGRADE),string(name: "EUS_CHANNEL", value: EUS_CHANNEL),
                        booleanParam(name: "WRITE_TO_FILE", value: WRITE_TO_FILE),booleanParam(name: "ENABLE_FORCE", value: ENABLE_FORCE),
                        booleanParam(name: "SCALE", value: SCALE),text(name: "ENV_VARS", value: ENV_VARS)
                    ]
                    currentBuild.description += """
                        <b>Upgrade Job: </b> <a href="${upgrade_ci.absoluteUrl}"> ${upgrade_ci.getNumber()} </a> <br/>
                    """
                    if( upgrade_ci.result.toString() != "SUCCESS") {
                       status = "Upgrade Failed"
                       currentBuild.result = "FAILURE"
                    }
                }
            }
        }

        stage("Write out results") {
            agent { label params['JENKINS_AGENT_LABEL'] }
            when {
                expression { params.WRITE_TO_FILE == true }
            }
            steps{
                script{
                    println "write to file $loaded_ci "

                    if(loaded_ci != null ) {
                        loaded_url = loaded_ci.absoluteUrl
                    }
                    if(upgrade_ci != null ) {
                        upgrade_url = upgrade_ci.absoluteUrl
                    }
                    if ( must_gather != null ) {
                        must_gather_url = must_gather.absoluteUrl
                    }
                    build job: 'scale-ci/e2e-benchmarking-multibranch-pipeline/write-scale-ci-results', parameters: [
                        string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL),string(name: "BUILD_NUMBER", value: build_string),
                        string(name: 'CI_STATUS', value: "${status}"), string(name: 'UPGRADE_JOB_URL', value: upgrade_url),text(name: "ENV_VARS", value: ENV_VARS),
                        string(name: 'CI_JOB_URL', value: loaded_url), booleanParam(name: 'ENABLE_FORCE', value: ENABLE_FORCE),booleanParam(name: 'SCALE', value: SCALE),
                        string(name: 'LOADED_JOB_URL', value: BUILD_URL), string(name: 'JOB', value: "loaded-upgrade")
                    ], propagate: false

                }
              }
        }
        stage('Destroy Flexy Cluster'){
            agent { label params['JENKINS_AGENT_LABEL'] }
            steps{
                script{
                    if(install != null && (install.result.toString() != "SUCCESS" || params.DESTROY_WHEN_DONE == true)) {
                        destroy_ci = build job: 'ocp-common/Flexy-destroy', parameters: [
                            string(name: "BUILD_NUMBER", value: install.number.toString()),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL)
                        ]
                    } else if(install == null && params.DESTROY_WHEN_DONE == true) {
                        destroy_ci = build job: 'ocp-common/Flexy-destroy', parameters: [
                            string(name: "BUILD_NUMBER", value: build_string),string(name: "JENKINS_AGENT_LABEL", value: JENKINS_AGENT_LABEL)
                        ]
                    }

                    if( destroy_ci != null) {
                        println "destroy not null"
                        if( destroy_ci.result.toString() != "SUCCESS") {
                            println "destroy failed"
                            currentBuild.result = "FAILURE"
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                currentBuild.description += """
                    <b>Final Status:</b> ${status}<br/>
                """
            }
        }
    }
}
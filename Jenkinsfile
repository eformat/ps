#!groovy

try {
    timeout(time: 20, unit: 'MINUTES') {

        def project = "${env.PROJECT_NAME}"
        def branch = "${env.BRANCH_NAME}"
        branch = branch.toLowerCase()

        echo "Project is: ${env.PROJECT_NAME}"
        echo "Build Number is: ${env.BUILD_NUMBER}"
        echo "Branch name is: ${env.BRANCH_NAME}"
        echo "Job Name is: ${env.JOB_NAME}"
        def commit_id, source, origin_url, name

        node('maven') {
            stage('Initialise') {
                // Checkout code from repository - we want commit id and name
                checkout scm
                dir("${WORKSPACE}") {
                    commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    echo "Git Commit is: ${commit_id}"
                    def cmd0 = $/name=$(git config --local remote.origin.url); name=$${name##*/}; echo $${name%%.git}/$
                    name = sh(returnStdout: true, script: cmd0).trim()
                    name = "${name}-${branch}"
                    echo "Name is: ${name}"
                }
                origin_url = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
                source = "${origin_url}#${commit_id}"
                echo "Source URL is: ${source}"
            }

            stage('Build') {
                // Start Build or Create initial app if doesn't exist
                if (getBuildName(name)) {
                    echo 'Building image'
                    def build = getBuildName(name)
                    try {
                        sh "oc start-build ${build} --from-file=deployments/ROOT.war?raw=true --follow"
                    } catch (e) {
                        echo "build failed"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                } else {
                    echo 'Creating build'
                    try {
                        sh "oc new-build --strategy=source --name=${name} --binary -l app=${name} -i jboss-eap70-openshift"
                        sh "oc start-build ${build} --from-file=deployments/ROOT.war?raw=true --follow"
                    } catch (e) {
                        echo "build exists"
                    }
                }
            }

            stage('Deploy') {
                echo 'Deploying image'
                def deploy = getDeployName(name)
                if (deploy) {
                    openshiftDeploy(deploymentConfig: deploy)
                } else {
                    echo 'Creating deployment'
                    sh "oc new-app ${name}"
                }
            }

            stage('Create Route') {
                echo 'Creating a route to application'
                createRoute(name)
            }
        }
    }

} catch (err) {
    echo "in catch block"
    echo "Caught: ${err}"
    currentBuild.result = 'FAILURE'
    throw err
}

// Expose service to create a route
def createRoute(String name) {
    try {
        def service = getServiceName(name)
        sh "oc expose svc ${service}"
    } catch (Exception e) {
        echo "route exists"
    }
}

// Get Build Name
def getBuildName(String name) {
    def cmd1 = $/buildconfig=$(oc get bc -l app=${name} -o name);echo $${buildconfig##buildconfig/}/$
    bld = sh(returnStdout: true, script: cmd1).trim()
    return bld
}

// Get Deploy Config Name
def getDeployName(String name) {
    def cmd2 = $/deploymentconfig=$(oc get dc -l app=${name} -o name);echo $${deploymentconfig##deploymentconfig/}/$
    dply = sh(returnStdout: true, script: cmd2).trim()
    return dply
}

// Get Service Name
def getServiceName(String name) {
    def cmd3 = $/service=$(oc get svc -l app=${name} -o name);echo $${service##service/}/$
    svc = sh(returnStdout: true, script: cmd3).trim()
    return svc
}

// Set Build Ref
def setBuildRef(String build, String source, String commit_id) {
    def cmd4 = $/oc patch bc/"${build}" -p $'{\"spec\":{\"source\":{\"git\":{\"uri\":\"${source}\",\"ref\": \"${
        commit_id
    }\"}}}}$'/$
    sh cmd4
}

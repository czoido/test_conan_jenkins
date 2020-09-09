config_url = "https://github.com/conan-ci-cd-training/settings.git"
artifactory_url = "host.docker.internal"
conan_develop_repo = "conan-develop"
conan_tmp_repo = "conan-tmp"

def profiles = [
  "debug-gcc6": "conanio/gcc6",	
  "release-gcc6": "conanio/gcc6"	
]

def get_stages(profile, docker_image) {
    return {
        stage(profile) {
            node {
                docker.image(docker_image).inside("--net=host") {
                    def scmVars = checkout scm
                    withEnv(["CONAN_USER_HOME=${env.WORKSPACE}/${profile}/conan_cache/"]) {
                        try {
                            stage("Configure Conan") {
                                //sh "conan --version"
                                sh "conan profile new ${profile} --detect"
                                withCredentials([usernamePassword(credentialsId: 'artifactory-credentials', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                                    sh "conan remote add ${conan_develop_repo} http://${artifactory_url}:8081/artifactory/api/conan/${conan_develop_repo}" // the namme of the repo is the same that the arttifactory key
                                    sh "conan user -p ${ARTIFACTORY_PASSWORD} -r ${conan_develop_repo} ${ARTIFACTORY_USER}"
                                    sh "conan remote add ${conan_tmp_repo} http://${artifactory_url}:8081/artifactory/api/conan/${conan_tmp_repo}" // the namme of the repo is the same that the arttifactory key
                                    sh "conan user -p ${ARTIFACTORY_PASSWORD} -r ${conan_tmp_repo} ${ARTIFACTORY_USER}"
                                }
                            }
                            stage("Create package") {                                
                                sh "conan graph lock . --profile ${profile} --lockfile=lockfile.lock -r ${conan_develop_repo}"
                                sh "cat lockfile.lock"
                                sh "conan create . --profile ${profile} --lockfile=lockfile.lock -r ${conan_develop_repo} --ignore-dirty"
                                sh "cat lockfile.lock"
                            }
                        }
                        finally {
                            deleteDir()
                        }
                    }
                }
            }
        }
    }
}

pipeline {
    agent none
    stages {

        stage('Build') {
            steps {
                script {
                    withEnv(["CONAN_HOOK_ERROR_LEVEL=40"]) {
                        parallel profiles.collectEntries { profile, docker_image ->
                            ["${profile}": get_stages(profile, docker_image)]
                        }
                    }              
                }
            }
        }
    }
}
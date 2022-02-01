pipeline {
    agent any

    options {
        ansiColor('xterm')
        disableConcurrentBuilds()
    }

    stages {
        // Stage conf to setup environment variable for build
        stage("conf") {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    env['PROJ'] = pom.artifactId
                    env['PROJ_PATH'] = "greet-demo"
                    def inventory = readFile file: 'deployment/inventory'
                    def get_ip = "${inventory}" =~ /(?sm).*minideploy ansible_host=((?:(?:\d+)\.?){4}).*/
                    echo get_ip
                    env['SERVER_IP'] = get_ip[0][1]
                    // Remove matcher object so it wont be serialized at the end of scope
                    // And a cps exception will be thrown
                    get_ip = null
                    sh 'env'
                }
            }
        }
        // Building the artifacts
        stage("build") {
            agent {
                docker {
                    image "maven:3-jdk-8-slim"
                    reuseNode true
                    args "-v \$HOME/.m2:/root/.m2"
                }
            }
            steps {
                sh 'mvn --version'
                sh 'mvn clean install -ntp -B -e'
            }
        }

        // building the image locally
        stage("build img") {
            steps {
                script {
                    env["IMAGE"] = "${env.PROJ}:${env.BRANCH_NAME}.${env.BUILD_ID}"
                    docker.build(env["IMAGE"])
                    sh "docker save -o deployment/${env["IMAGE"]}.tar ${env["IMAGE"]}"
                    env["ABS_IMAGE_PATH"] = "${env.WORKSPACE}/deployment/${env.IMAGE}.tar"
                }
            }
        }

        // building and packaging the helm package locally with jn permisiones
        stage("helm package") {
            agent {
                dockerfile {
                    filename 'Dockerfile_helm'
                    dir 'helm-package'
                    reuseNode true
                }
            }
            steps {
                script {
                    sh 'helm lint helm-package/greet'
                    sh 'helm package helm-package/greet -u'
                    def chert_yaml = readYaml file: 'helm-package/greet/Chart.yaml'
                    env["HELM_PACKAGE"] = "greet-${chert_yaml.version}.tgz"
                }
            }
        }

        stage("build deployment image") {
            steps {
                script {
                    env['DEP_IMAGE'] = "ansible_dep:${env.BUILD_ID}"
                    def dockerfile = 'deployment/Dockerfile_ansible'
                    docker.build("${env.DEP_IMAGE}", "-f ${dockerfile} .")
                }
            }
        }

        // deploying app when on release branches
        stage("deploy") {
            agent {
                docker {
                    image "${env.DEP_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                dir("deployment") {
                    sh 'ls -la'
                }
                ansiblePlaybook(
                        playbook: 'deployment/deploy_app_to_minikube.yml',
                        inventory: 'deployment/inventory',
                        colorized: true,
                        disableHostKeyChecking: true,
                        credentialsId: 'test-key',
                        extras: "-e image=${env.IMAGE} " +
                                "-e project_name=${env.PROJ} " +
                                "-e project_path=${env.PROJ_PATH} " +
                                "-e helm_package=${env.HELM_PACKAGE} " +
                                "-e replica_count=2 " +
                                "-e service_port=80 " +
                                "-e greeted=Jhon " +
                                "-vv"
                )
            }
        }

//        stage("remote test") {
//            agent {
//                docker {
//                    image "${env.DEP_IMAGE}"
//                    reuseNode true
//                }
//            }
//            steps {
//                sh 'ls -la'
//                ansiblePlaybook(
//                        playbook: 'deployment/start_test_minikube_app.yml',
//                        inventory: 'deployment/inventory',
//                        colorized: true,
//                        disableHostKeyChecking: true,
//                        extras: "-e project_name=${env.PROJ} " +
//                                "-e project_path=${env.PROJ_PATH} " +
//                                "-vv",
//                        credentialsId: 'test-key'
//                )
//                script {
//                    try {
//                        sleep 15
//                        sh "curl -L -D - http://${env.SERVER_IP}:8080/greeting?name=katsok"
//
//                    } catch (err) {
//                        echo "Remote Test Failed: ${err}"
//                        currentBuild.result = "UNSTABLE"
//                    } finally {
//                        echo "Always tear down env"
//                        ansiblePlaybook(
//                                playbook: 'deployment/stop_test_minikube_app.yml',
//                                inventory: 'deployment/inventory',
//                                colorized: true,
//                                disableHostKeyChecking: true,
//                                credentialsId: 'test-key'
//                        )
//                    }
//                }
//            }
//        }

    }
    post {
        always {
//            script {
//                def status = "${env.BUILD_TAG} - ${currentBuild.currentResult}"
//                def body = """
//Build: ${currentBuild.displayName}
//Result: ${currentBuild.currentResult}
//"""
//                mail body: body, subject: status, to: 'katsok@personetics.com'
//            }
            sh "rm deployment/${env.IMAGE}.tar"

        }
    }
}

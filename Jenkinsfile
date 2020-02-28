pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            if (!openshift.selector("dc", "myapp").exists()) {
                                openshift.newApp('--image-stream=nodejs:10', '--code=https://github.com/fabianpy/openshift-nodejs.git', '--name=myapp', '--strategy=source')
                                openshift.set("triggers", "dc/myapp", "--remove-all")
                                def bc=openshift.selector("bc", "myapp").object()
                                bc.spec.strategy.sourceStrategy.incremental=true
                                openshift.apply(bc)
                                def dc=openshift.selector("dc", "myapp").object()
                                openshift.selector("dc/myapp").delete()
                                openshift.newApp('--image-stream=myproject/myapp', '--name=myapp', '--allow-missing-imagestream-tags')
                                openshift.set("triggers", "dc/myapp", "--remove-all")
                                openshift.expose("svc", "myapp")
                                openshift.selector("bc", "myapp").cancelBuild()
                                openshift.startBuild("myapp", "--wait")
                            } else {
                                openshift.startBuild("myapp", "--wait")
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            if (openshift.selector("dc", "myapp").exists()) {
                                openshift.selector("dc", "myapp").rollout().latest()
                            }
                        }
                    }
                }
            }
        }
    }
}
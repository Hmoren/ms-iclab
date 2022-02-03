import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
     environment {
        NEXUS_USER_VAR      = credentials('NEXUS-USER')
        NEXUS_USER_PASS_VAR = credentials('NEXUS-PASS')
        
    }
    stages {
        stage("Paso 1: Compliar"){
            steps {
                script {
                sh "echo 'Compile Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 2: Testear"){
            steps {
                script {
                sh "echo 'Test Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 3: Build .Jar"){
            steps {
                script {
                sh "echo 'Build .Jar!'"
                // Run Maven on a Unix agent.
                sh "mvn clean package -e"
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage("Paso 4: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=ms-iclab'
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    //archiveArtifacts artifacts:'build/*.jar'
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'devops-usach-nexus',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar']
                                ],
                        mavenCoordinate: [
                            artifactId: 'DevOpsUsach2020',
                            groupId: 'com.devopsusach2020',
                            packaging: 'jar',
                            version: '0.0.1-MS-ICLAB']
                        ]
                    ]
                }
            }
        }
        stage("Paso 5 Download: Nexus"){
            steps {
                sh ' curl -X GET -u $NEXUS_USER_VAR:$NEXUS_USER_PASS_VAR "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1-MS-ICLAB/DevOpsUsach2020-0.0.1-MS-ICLAB.jar" -O'
            }
        }
        stage("Paso 6 Run: Levantar Springboot APP"){
            steps {
                sh 'mvn spring-boot:run &'
                sh 'nohup bash java -jar DevOpsUsach2020-0.0.1-MS-ICLAB.jar & >/dev/null'
            }
        }
        stage("Paso 7 Curl: Dormir(Esperar 40sg) "){
            steps {
               sh "sleep 40 && curl -X GET 'http://localhost:8082/rest/mscovid/test?msg=testing'"
            }
        }
        // stage(" paso 8 Subir nueva Version"){
        //     steps {
        //         //archiveArtifacts artifacts:'build/*.jar'
        //         nexusPublisher nexusInstanceId: 'nexus',
        //             nexusRepositoryId: 'devops-usach-nexus',
        //             packages: [
        //                 [$class: 'MavenPackage',
        //                     mavenAssetList: [
        //                         [classifier: '',
        //                         extension: 'jar',
        //                         filePath: 'build/DevOpsUsach2020-0.0.1.jar']
        //                     ],
        //                     //cometario Daniel Tapia 3
        //             mavenCoordinate: [
        //                 artifactId: 'DevOpsUsach2020',
        //                 groupId: 'com.devopsusach2020',
        //                 packaging: 'jar',
        //                 version: '0.0.4']
        //             ]
        //         ]
        //     }
        // }
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}
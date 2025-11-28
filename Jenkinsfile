pipeline {
    agent any
    tools { 
        nodejs 'Node 20' 
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning Repository...'
                checkout scm
            }
        }

        // stage('Install Backend Dependencies') {
        //     steps {
        //         echo 'Installing backend deps...'
        //         dir('CareFlow-BackEnd') {
        //             path 'npm install'
        //         }
        //     }
        // }

        stage('Install Frontend Dependencies') {
            steps {
                echo 'Installing frontend deps...'
                dir('CareFlow-FrontEnd') {
                    path 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running Tests...'
                // dir('CareFlow-BackEnd') { path 'npm test || true' }
                // dir('CareFlow-FrontEnd') { path 'npm test || true' }
            }
        }


    //     stage('Test_0') {
    // parallel {
    //     stage('Backend Tests') {
    //         steps {
    //             echo 'Running Backend Tests...'
    //             // dir('CareFlow-BackEnd') {
    //             //     path 'npm test || true'
    //             // }
    //         }
    //     }

//         stage('Frontend Tests') {
//             steps {
//                 echo 'Running Frontend Tests...'
//                 // dir('CareFlow-FrontEnd') {
//                 //     path 'npm test || true'
//                 // }
//             }
//         }
//     }
// }


//        stage('Docker Compose Up') {
//     steps {
//         echo 'Starting Containers...'
//         path 'docker-compose -d --build'
//     }
// }

    }

    post {
    success {
        echo 'Build succeeded, cleaning up...'
        // path 'docker compose down || true'
    }
    failure {
        echo 'Build failed, cleaning up...'
        // path 'docker compose down || true'
    }
}
}
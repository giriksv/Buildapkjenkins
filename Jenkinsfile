pipeline {
    agent any
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                flutter --version || curl -sL https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.13.3-stable.tar.xz | tar xJ
                export PATH=$PATH:$PWD/flutter/bin
                flutter doctor
                '''
            }
        }
        stage('Clone Repository') {
            steps {
                // Use credentials to clone the repository
                git branch: 'main', url: 'https://github.com/giriksv/Buildapkjenkins.git', credentialsId: 'ksv2023Sample'
            }
        }
        stage('Generate Keystore') {
            steps {
                sh '''
                keytool -genkeypair -v -keystore release-key.jks -keyalg RSA -keysize 2048 -validity 10000 \
                    -storepass Kanavu@2023 -keypass Kanavu@2023 -dname "CN=Giri, OU=KSV, O=KSV, L=Erode, S=Tamil Nadu, C=IN"
                '''
            }
        }
        stage('Generate SHA Values') {
            steps {
                sh '''
                keytool -list -v -keystore release-key.jks -storepass Kanavu@2023 | tee sha_values.txt
                '''
            }
            script {
                archiveArtifacts artifacts: 'sha_values.txt', fingerprint: true
            }
        }
        stage('Build APK & AAB') {
            steps {
                sh '''
                flutter build apk --release --no-sound-null-safety --flavor production --keystore=release-key.jks \
                    --store-password=Kanavu@2023 --key-alias=Giri --key-password=Kanavu@2023
                flutter build appbundle --release --no-sound-null-safety --flavor production --keystore=release-key.jks \
                    --store-password=Kanavu@2023 --key-alias=Giri --key-password=Kanavu@2023
                '''
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: '**/build/app/outputs/**/*.apk, **/build/app/outputs/**/*.aab', fingerprint: true
            echo "Build successful! APK and AAB files archived."
        }
        failure {
            echo "Build failed. Check the logs for details."
        }
    }
}

pipeline {
    agent none
    stages {
        stage('build') {
            parallel {

                stage('Android Release') {
                    environment {
                        CCACHE_BASEDIR = "${env.WORKSPACE}"
                        QGC_CONFIG = 'release'
                        QMAKE_VER = "5.11.0/android_armv7/bin/qmake"
                        GSTREAMER_ROOT_ANDROID = "/qgroundcontrol/gstreamerr"
                        QGC_REGISTRY_CREDS = credentials('qgc_uploader')
                    }
                    agent {
                        docker{
                            image 'mavlink/qgc-build-android'
                        }
                    }

                    steps {
                        sh 'echo $PATH'
                        sh 'echo $CCACHE_BASEDIR'
                        withCredentials(bindings: [file(credentialsId: 'AndroidReleaseKey', variable: 'ANDROID_KEYSTORE')]) {
                            sh 'cp $ANDROID_KEYSTORE ${WORKSPACE}/android/android_release.keystore.h'
                        }

                        sh './tools/update_android_version.sh;'
                        sh 'export'
                        sh 'ccache -z'
                        sh 'git submodule deinit -f .'
                        sh 'git clean -ff -x -d .'
                        sh 'git submodule update --init --recursive --force'
                        sh 'wget --user=${QGC_REGISTRY_CREDS_USR} --password=${QGC_REGISTRY_CREDS_PSW} --quiet http://192.168.2.144:8086/nexus/repository/gstreamer-android-qgroundcontrol/gstreamer/gstreamer-1.0-android-universal-1.14.4.tar.bz2'
                        sh 'tar jxf gstreamer-1.0-android-universal-1.14.4.tar.bz2 -C ${WORKSPACE}'
                        withCredentials([string(credentialsId: 'ANDROID_STOREPASS', variable: 'ANDROID_STOREPASS')]) {
                            sh 'mkdir build; cd build; ${QT_PATH}/${QMAKE_VER} -r ${WORKSPACE}/qgroundcontrol.pro INCLUDEPATH+=/curl-android-ios-cURL_7.60.0/prebuilt-with-ssl/android/include/ LIBS+=-L/curl-android-ios-cURL_7.60.0/prebuilt-with-ssl/android/armeabi-v7a/ CONFIG+=WarningsAsErrorsOn CONFIG+=installer CONFIG+=${QGC_CONFIG} CONFIG+=QGC_DISABLE_APM_PLUGIN_FACTORY -spec android-clang'
                            sh 'cd build; make -j`nproc --all`'
                        }
                        sh 'ccache -s'
                    }
                    post {
                        success {
                            stash(includes: 'build/**/*.apk', name: 'android_binary')
                            archiveArtifacts artifacts: 'build/**/*.apk'

                        }
                        cleanup {
                            sh 'rm -r ${WORKSPACE}/build || true'
                            sh 'rm -r ${WORKSPACE}/gstreamer* || true'
                            sh 'git clean -ff -x -d .'
                        }
                    }
                }
            }
        }
    }
    environment {
        CCACHE_CPP2 = '1'
        CCACHE_DIR = '/tmp/ccache'
        QT_FATAL_WARNINGS = '1'
    }
}

def getTag()
{
    tags = sh(returnStdout: true, script: "git tag").trim()
    print tags
    return tags
}

def getVersion()
{
    tags = sh(returnStdout: true, script: "git describe --tags --abbrev=7").trim()
    print tags
    return tags
}

def getHash()
{
    hash = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
    print hash
    return hash
}

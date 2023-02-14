#!groovy
// Check properties
properties([disableConcurrentBuilds()])

pipeline {

    agent { 
        label 'ADD'
        }

    parameters {
        string(name: "branch", defaultValue: "master", trim: true, description: "ветка с фичей")   
    }

    environment {
        BOTID=credentials('BotID')
        CHATID=credentials('ChatID')
        SERVERNAME=credentials('ServerName')
        BASENAME=credentials('BaseName')
        TESTBASEUSER=credentials('TestBaseUser')
        TESTBASEPASS=credentials('TestBasePass')
        SYNTAXCHECKFILESALLURE="${WORKSPACE}\\allure-results"
        SYNTAXCHECKFILEJUNIT="${WORKSPACE}\\allure-results\\junit.xml"
        CFPATH="${WORKSPACE}\\1Cv8.cf"
        TEXTSUCCESS="SUCCESS"
        TEXTUNSUCCESSFUL="UNSUCCESSFUL"
        SRCPATH="${WORKSPACE}\\featuresrc"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()
    }

    stages {

        // получаем изменения из ветки
        //stage('git pull') {
        //    steps {
        //        TODO переключаемся на ветку и получаем все изменения (pull) из ветки branch    
        //    }
        //}

        // компилируем конфигурацию
        stage('Compile CF') {
            steps {
                bat "@chcp 1251 && @call vrunner compile --v8version 8.3.19.1331 --src ${SRCPATH} --out ${CFPATH}"    
            }
            post {
                always {
                    archiveArtifacts artifacts: '1Cv8.cf', onlyIfSuccessful: true
                }
            }
        }

        // загружаем конфигурацию в тестовую базу
        stage('Load CF in test base') {
            steps {
                bat "@chcp 1251 && @call vrunner load --src 1Cv8.cf --ibconnection /S${SERVERNAME}/${BASENAME} --db-user ${TESTBASEUSER} --db-pwd ${TESTBASEPASS} --v8version 8.3.19.1331"
            }
            post {
                always {
                    bat "DEL ${CFPATH} /q /f"
                    bat "RD ${SRCPATH} /q /s"     
                }
            }
        }

        // обновляем тестовую базу
        stage('Update test base') {
            steps {
                bat "@chcp 1251 && @call vrunner updatedb --ibconnection /S${SERVERNAME}/${BASENAME} --db-user ${TESTBASEUSER} --db-pwd ${TESTBASEPASS} --v8version 8.3.19.1331 --v1"    
            }
        }

        // проверям тестовую базу
        stage('Syntax check') {
            steps {
                bat "@chcp 1251 && @call vrunner syntax-check --ibconnection /S${SERVERNAME}/${BASENAME} --db-user ${TESTBASEUSER} --db-pwd ${TESTBASEPASS} --junitpath ${SYNTAXCHECKFILEJUNIT} --allure-results2 ${SYNTAXCHECKFILESALLURE} --v8version 8.3.19.1331 --exception-file syntax_check_exception.txt --groupbymetadata false --debuglogfile false --mode -ThinClient -Server -ThickClientServerManagedApplication"
            }
            post {
                always {
                    allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
                }
            }
        }

        // запускаем тесты в тестовой базе
        stage('BDD test') {
            steps {
                bat "@chcp 1251 && @call vrunner vanessa --ibconnection /S${SERVERNAME}/${BASENAME} --db-user ${TESTBASEUSER} --db-pwd ${TESTBASEPASS} --v8version 8.3.19.1331 --workspace . --vanessasettings ./VBParams.json --pathvanessa ./vanessa-automation.epf"               
            }
        }

    }

    post {
        success { 
            bat "oscript ${WORKSPACE}/NotifyTelegram.os ${BOTID} ${CHATID} ${TEXTSUCCESS}"    
        }
        unsuccessful { 
            bat "oscript ${WORKSPACE}/NotifyTelegram.os ${BOTID} ${CHATID} ${TEXTUNSUCCESSFUL}"   
        }
    }

}
pipeline {
    agent any

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')

        withKubeConfig(caCertificate: '''-----BEGIN CERTIFICATE-----
        MIIC3TCCAcWgAwIBAgIRALJud0lsqUg7rMEei25f9MIwDQYJKoZIhvcNAQELBQAw
        FjEUMBIGA1UEAwwLQVBJLUNsdXN0ZXIwHhcNMjQwMTA4MTE0OTUxWhcNMjkwMTA3
        MTE0OTUxWjAWMRQwEgYDVQQDDAtBUEktQ2x1c3RlcjCCASIwDQYJKoZIhvcNAQEB
        BQADggEPADCCAQoCggEBAL5or11a+ieTf9tFCC3/uIomU34fTH5gE1PENu4g2fM/
        dH7nH+gtt6qnhi8AnvRpTfeNzBBHQ71t+rJjYv9VKO36hxL647v0zueh52kg6ogo
        onkmXuJdsaZKeRA3w4LYukE+cRTG4mwFM9/VnVB6FQT6tLOfHfcigJWzBw/u/XTQ
        EpNjd5PLf8m56zahlzfsSd7HSpCdyzBQVzvUkkVmbq/qPc6r6knbLjrKwjIsRh8j
        uRXPRHRNRgzB2lLjkN3tTfaXozE7/z7/6pIRcOAbR1VzlujUWQRbuzx+Aru/JQfO
        0RUd2zeFsRwOFiWhRJt2hbH5ECQcsuVEl1uoFqNJvfkCAwEAAaMmMCQwEgYDVR0T
        AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAgQwDQYJKoZIhvcNAQELBQADggEB
        AFwnrjQlYX8gfyxJN8JsIkU8BQNlUWy2xZSpx5nNzSxxwJXm1uRtley9kfpf4P4g
        QmwVzV9iV465z6Mmh5TRxiYVM/3d1Q0wfvwf9t60z1PW0F9jzLa3zjTF3Evdz6nL
        ys0BWDlEcIMiIVKUmdNiYle23p+bHDRsr4fm5IphxtHy/rbfvije5rNDs5gXVScd
        3gJrqa5IszbxZbvLNYvu97iFitxPUIZSplgTz0aoeATCBh/pwBNPbxGsnvF1Yzs5
        7rNj8Rg6g6Ph+UvRTm2sffoDE7oJmo4Zr08QeWta8chwS95qS6XcpCEyXadiDSde
        g8TnRLpn/QAjK/O0CRMUQMA=
        -----END CERTIFICATE-----''', clusterName: 'API-Cluster', contextName: 'default', credentialsId: 'f4826e5c-a7bb-4ace-ab83-1d91a7b14313', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://180.93.180.3:6443')
    }

    environment {
        DNS_APIKEY = 'gHzYh74VJWFj_GeraDcnduffMvAVVGiDnZP'
        DNS_APISECRET = 'JJA6UeV6NoQAc7SSb8hr3W'
        DNS_RECORD_DATA_API = '180.93.180.102'
        DNS_RECORD_DATA_WEB = '116.118.95.121'
        DOMAIN = 'minhnhut.online'
        SA_PASSWORD = 'As@19006123'
        SQLSERVER = '61.28.229.125'
        WEB_SERVER_IP = '116.118.95.121'
        WEBSERVER_PASSWORD = 'As@19006123'
        WEBSERVER_USERNAME = 'web-server\\stewie12061'
    }

    parameters {
        string(name: 'deploymentName', defaultValue: 'test', description: 'Deployment Name')
        string(name: 'expireMinute', defaultValue: '30', description: 'Expiration Minute (0-59)')
        string(name: 'expireHour', defaultValue: '17', description: 'Expiration Hour (0-23)')
        string(name: 'expireDay', defaultValue: '1', description: 'Expiration Day (1-31)')
        string(name: 'expireMonth', defaultValue: '1', description: 'Expiration Month (1-12)')
        string(name: 'expireYear', defaultValue: '2024', description: 'Expiration Year')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script{
                    def powershellScript = '''
                            $deploymentName = "$env:deploymentName"
                            $expireMinute = "$env:expireMinute"
                            $expireHour = "$env:expireHour"
                            $expireDay = "$env:expireDay"
                            $expireMonth = "$env:expireMonth"

                            function Validate-DeploymentName {
                                param (
                                    [string]$name
                                )
                                if ($name -notmatch "^[a-z][a-z0-9-]{0,61}[a-z0-9]$") {
                                    Write-Host $name + " ?"
                                    Write-Host "##vso[task.logissue type=error]Invalid deploymentName. It must contain at most 63 characters, only lowercase alphanumeric characters or \'-\', start with an alphabetic character, and end with an alphanumeric character."
                                    exit 1
                                }
                            }
                            function Validate-ExpireMinute {
                                param (
                                    [int]$minute
                                )
                                if ($minute -lt 0 -or $minute -gt 59) {
                                    Write-Host "##vso[task.logissue type=error]Invalid expireMinute. Allowed values are 0-59."
                                    exit 1
                                }
                            }
                            function Validate-ExpireHour {
                                param (
                                    [int]$hour
                                )
                                if ($hour -lt 0 -or $hour -gt 23) {
                                    Write-Host "##vso[task.logissue type=error]Invalid expireHour. Allowed values are 0-23."
                                    exit 1
                                }
                            }
                            function Validate-ExpireDay {
                                param (
                                    [int]$day
                                )
                                if ($day -lt 1 -or $day -gt 31) {
                                    Write-Host "##vso[task.logissue type=error]Invalid expireDay. Allowed values are 1-31."
                                    exit 1
                                }
                            }
                            function Validate-ExpireMonth {
                                param (
                                    [int]$month
                                )
                                if ($month -lt 1 -or $month -gt 12) {
                                    Write-Host "##vso[task.logissue type=error]Invalid expireMonth. Allowed values are 1-12."
                                    exit 1
                                }
                            }

                            # Validate deploymentName
                            Validate-DeploymentName -name $deploymentName

                            # Validate expireMinute
                            Validate-ExpireMinute -minute $expireMinute

                            # Validate expireHour
                            Validate-ExpireHour -hour $expireHour

                            # Validate expireDay
                            Validate-ExpireDay -day  $expireDay

                            # Validate expireMonth
                            Validate-ExpireMonth -month $expireMonth

                            Write-Host "All parameters passed validation successfully."
                    '''
                    powershell(script: powershellScript)
                }
            }
        }
        stage('Archive Artifact'){
            steps {
                archiveArtifacts artifacts: '**/*', followSymlinks: false
            }
        }
        stage('Lists Pods'){
            steps{
                sh 'kubectl get pods'
            }
        }
    }

    post {
        always {
            echo 'Finished, initiate the clean up workspace'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'Succeeeded!'
        }
        unstable {
            echo 'Unstable :/'
        }
        failure {
            echo 'Failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
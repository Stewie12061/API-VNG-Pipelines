pipeline {
    agent any

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')

        withKubeConfig(caCertificate: '', clusterName: 'API-Cluster', contextName: 'default', credentialsId: 'f4826e5c-a7bb-4ace-ab83-1d91a7b14313', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://180.93.180.3:6443')
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
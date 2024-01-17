pipeline {
    agent any

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
        number(name: 'expireMinute', defaultValue: 30, description: 'Expiration Minute (0-59)')
        number(name: 'expireHour', defaultValue: 17, description: 'Expiration Hour (0-23)')
        number(name: 'expireDay', defaultValue: 1, description: 'Expiration Day (1-31)')
        number(name: 'expireMonth', defaultValue: 1, description: 'Expiration Month (1-12)')
        number(name: 'expireYear', defaultValue: 2024, description: 'Expiration Year')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                powershell '''function Validate-DeploymentName {
                    param (
                        [string]$name
                    )
                    if ($name -notmatch "^[a-z][a-z0-9-]{0,61}[a-z0-9]$") {
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
                Validate-DeploymentName -name ${{ parameters.deploymentName }}

                # Validate expireMinute
                Validate-ExpireMinute -minute ${{ parameters.expireMinute }}

                # Validate expireHour
                Validate-ExpireHour -hour ${{ parameters.expireHour }}

                # Validate expireDay
                Validate-ExpireDay -day  ${{ parameters.expireDay }}

                # Validate expireMonth
                Validate-ExpireMonth -month ${{ parameters.expireMonth }}

                Write-Host "All parameters passed validation successfully."'''
            }
        }
        stage('Archive Artifact'){
            steps {
                archiveArtifacts artifacts: '**/*.yaml', followSymlinks: false
            }
        }
        stage('UnArchive Artifacrt'){
            steps{
                unarchive(mapping: ['dir/' : '.'])
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
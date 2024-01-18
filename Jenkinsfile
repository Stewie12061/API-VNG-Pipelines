pipeline {
    agent any

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')

        withKubeConfig(caCertificate: '', clusterName: 'API-Cluster', contextName: 'default', credentialsId: 'f4826e5c-a7bb-4ace-ab83-1d91a7b14313', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://180.93.180.3:6443')
    }

    environment {
        DNS_APIKEY = credentials('dns-api-key-creds')
        DNS_APISECRET = credentials('dns-api-secret-creds')
        DNS_RECORD_DATA_API = '180.93.180.102'
        DNS_RECORD_DATA_WEB = '116.118.95.121'
        DOMAIN = 'minhnhut.online'
        SA_PASSWORD = credentials('sa-password-creds')
        SQLSERVER = '61.28.229.125'
        WEB_SERVER_IP = '116.118.95.121'
        WEBSERVER_PASSWORD = credentials('web-server-password-creds')
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
                archiveArtifacts artifacts: '**/*.yaml', followSymlinks: false
            }
        }
        stage('Unarchive to Specific Folder') {
            steps {
                // Unarchive the artifacts to a specific folder
                unarchive mapping: ['*': './manifests']
            }
        }
        stage('Convert UTC and Replace Variables in YAML Templates'){
            steps{
                script{
                    def convertTimeReplaceScript = '''
                    function ConvertToUtc {
                        param(
                            [int]$day,
                            [int]$month,
                            [int]$year,
                            [int]$hour,
                            [int]$minute
                        )

                        $localTime = Get-Date -Year $year -Month $month -Day $day -Hour $hour -Minute $minute -Second 0
                        $utcTime = $localTime.ToUniversalTime()

                        return $utcTime
                    }
                    $SA_PASSWORD = "$env:SA_PASSWORD"
                    $SQLSERVER = "$env:SQLSERVER"
                    $DNS_APIKEY = "$env:DNS_APIKEY"
                    $DNS_APISECRET = "$env:DNS_APISECRET"
                    $DOMAIN = "$env:DOMAIN"

                    $deploymentName = "$env:deploymentName"
                    $expireMinute = "$env:expireMinute"
                    $expireHour = "$env:expireHour"
                    $expireDay = "$env:expireDay"
                    $expireMonth = "$env:expireMonth"
                    $expireYear = "$env:expireYear"
                    $utcTime = ConvertToUtc -day $expireDay -month $expireMonth -year $expireYear -hour $expireHour -minute $expireMinute
                    # Extract and display components separately
                    $utcHour = $utcTime.Hour
                    $utcDay = $utcTime.Day
                    $utcMonth = $utcTime.Month

                    $WORKSPACE = "$env:WORKSPACE\\manifests"

                    $templateFiles = Get-ChildItem -Path $WORKSPACE -Filter '*.yaml' -Recurse
                    foreach ($file in $templateFiles) {
                        (Get-Content $file.FullName) | ForEach-Object {
                            $_ -replace '\\$\\(deploymentName\\)', "$deploymentName" `
                            -replace '\\$\\(expireMinute\\)', "$expireMinute" `
                            -replace '\\$\\(expireHour\\)', "$utcHour" `
                            -replace '\\$\\(expireDay\\)', "$utcDay" `
                            -replace '\\$\\(expireMonth\\)', "$utcMonth" `
                            -replace '\\$\\(SQLSERVER\\)', "$SQLSERVER" `
                            -replace '\\$\\(SA_PASSWORD\\)', "$SA_PASSWORD" `
                            -replace '\\$\\(DNS_APIKEY\\)', "$DNS_APIKEY" `
                            -replace '\\$\\(DNS_APISECRET\\)', "$DNS_APISECRET" `
                            -replace '\\$\\(DOMAIN\\)', "$DOMAIN"
                        } | Set-Content $file.FullName
                    }
                    '''
                    powershell(script: convertTimeReplaceScript)
                }
            }
        }
        // stage('Deploy restore database job'){
        //     steps{
        //         powershell('kubectl apply -f ./manifests/job-restore-db.yaml')
        //     }
        // }
        // stage('Check database status'){
        //     steps{
        //         script{
        //             def checkDBScript = '''
        //                 # Additional step to wait for the database to be ONLINE
        //                 $DatabaseName = "1BOSS_$env:deploymentName"
        //                 $ServerName = "$env:SQLSERVER"
        //                 $DBPassword = "$env:SA_PASSWORD"

        //                 Write-Host "Server Name: $ServerName"
        //                 Write-Host "Database Name: $DatabaseName"

        //                 # Loop for a maximum of $maxAttempts times
        //                 $maxAttempts = 10
        //                 $attempts = 0
        //                 while ($attempts -lt $maxAttempts) {
        //                     $attempts++

        //                     $connectionString = "Server=$ServerName;Database=master;User Id=sa;Password=$DBPassword;TrustServerCertificate=True;"
        //                     $databaseStatus = Invoke-Sqlcmd -ConnectionString $connectionString -Query "SELECT state_desc FROM sys.databases WHERE name = '$DatabaseName'"
        //                     $databaseState = $databaseStatus.state_desc

        //                     Write-Host "Database state: $databaseState"

        //                     if ($databaseState -eq "ONLINE") {
        //                         Write-Host "Database is now ONLINE. Proceeding with deployment."
        //                         break
        //                     }
        //                     Write-Host "Waiting for the database to be ONLINE. Attempt $attempts of $maxAttempts."
        //                     Start-Sleep -Seconds 10
        //                 }
        //                 # Check if the loop exited due to reaching the maximum number of attempts
        //                 if ($attempts -eq $maxAttempts) {
        //                     Write-Host "Database did not come ONLINE after $maxAttempts attempts. Canceling deployment."
        //                     exit 1  # Exit the script with a non-zero status code to mark it as failed
        //                 }
        //             '''
        //             powershell(script: checkDBScript)
        //         }
        //     }
        // }
        // stage('Add DNS Record For API'){
        //     steps{
        //         script{
        //             def createDNSRecord = '''
        //                 $apiKey = "$env:DNS_APIKEY"
        //                 $apiSecret = "$env:DNS_APISECRET"

        //                 # Set the domain and record information
        //                 $domain = "$env:DOMAIN"
        //                 $recordType = "A"
        //                 $recordName = "$env:deploymentName-api"
        //                 $recordData = "$env:DNS_RECORD_DATA_API"
        //                 $ttl = 600

        //                 # Construct the API endpoint URL
        //                 $apiEndpoint = "https://api.godaddy.com/v1/domains/$domain/records"

        //                 # Construct the headers
        //                 $headers = @{
        //                     'Authorization' = "sso-key $($apiKey):$($apiSecret)"
        //                     'Content-Type'  = 'application/json'
        //                 }

        //                 # Construct the payload for the new DNS record
        //                 $payload = ConvertTo-Json @(@{type=$recordType;name=$recordName;data=$recordData;ttl=$ttl})

        //                 # Make the API request to create the DNS record
        //                 $response = Invoke-WebRequest -Uri $apiEndpoint -Method Patch -Headers $headers -Body $payload

        //                 # Display the response
        //                 $response
        //             '''
        //             powershell(script: createDNSRecord)
        //         }
        //     }
        // }
        // stage('Deploy API'){
        //     steps{
        //         powershell '''kubectl apply -f ./manifests/api-deploy.yaml
        //             kubectl apply -f ./manifests/api-job-ingress-add-host.yaml
        //             kubectl apply -f ./manifests/api-cron-job-run-delete-deploy.yaml'''
        //     }
        // }
        // stage('Add DNS Record For Web'){
        //     steps{
        //         script{
        //             def createDNSRecord = '''
        //                 # Set your GoDaddy API key and secret
        //                 $apiKey = "$env:DNS_APIKEY"
        //                 $apiSecret = "$env:DNS_APISECRET"

        //                 # Set the domain and record information
        //                 $domain = "$env:DOMAIN"
        //                 $recordType = "A"
        //                 $recordName = "$env:deploymentName-web"
        //                 $recordData = "$env:DNS_RECORD_DATA_WEB"
        //                 $ttl = 600

        //                 # Construct the API endpoint URL
        //                 $apiEndpoint = "https://api.godaddy.com/v1/domains/$domain/records"

        //                 # Construct the headers
        //                 $headers = @{
        //                     'Authorization' = "sso-key $($apiKey):$($apiSecret)"
        //                     'Content-Type'  = 'application/json'
        //                 }

        //                 # Construct the payload for the new DNS record
        //                 $payload = ConvertTo-Json @(@{type=$recordType;name=$recordName;data=$recordData;ttl=$ttl})

        //                 # Make the API request to create the DNS record
        //                 $response = Invoke-WebRequest -Uri $apiEndpoint -Method Patch -Headers $headers -Body $payload

        //                 # Display the response
        //                 $response
        //             '''
        //             powershell(script: createDNSRecord)
        //         }
        //     }
        // }
        // stage('Create IIS WEB Site'){
        //     steps{
        //         script{
        //             def remotePSSession = '''
        //                 $server = "$env:WEB_SERVER_IP"
        //                 $uri = "https://$($server):5986"
        //                 $user = "$env:WEBSERVER_USERNAME"
        //                 $password = "$env:WEBSERVER_PASSWORD"
        //                 $securepassword = ConvertTo-SecureString -String $password -AsPlainText -Force
        //                 $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $user, $securepassword

        //                 $sessionOption = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
        //                 $session = New-PSSession -ConnectionUri $uri -Credential $cred -SessionOption $sessionOption
        //                 Invoke-Command -Session $session -ScriptBlock {
        //                     $folderName= $using:env:deploymentName
        //                     $SA_PASSWORD= $using:env:SA_PASSWORD
        //                     $SQLSERVER= $using:env:SQLSERVER

        //                     # Create publish folder
        //                     robocopy.exe "C:\\Publish0" "C:\\WebDemo\\$folderName" /E /MIR /MT:4 /np /ndl /nfl /nc /ns

        //                     $siteName = "$folderName"
        //                     $publishFolder = "C:\\WebDemo\\$folderName"
        //                     $applicationPoolName = "$folderName"
        //                     $bindingIPAddress = "*"
        //                     $bindingPort = "80"
        //                     $hostname = "${folderName}-web.minhnhut.online"

        //                     # Check if IIS module is installed
        //                     if (-not (Get-Module -ListAvailable -Name WebAdministration)) {
        //                         Install-Module -Name WebAdministration -Force -AllowClobber
        //                     }

        //                     # Import the WebAdministration module
        //                     Import-Module WebAdministration

        //                     # Create Application Pool
        //                     New-WebAppPool -Name $applicationPoolName

        //                     # Create Website with Custom Binding
        //                     New-Website -Name $siteName -PhysicalPath $publishFolder -ApplicationPool $applicationPoolName -Port $bindingPort -HostHeader $hostname -Force

        //                     Write-Host "Website '$siteName' created successfully."

        //                     $configFilePath = "C:\\WebDemo\\$folderName\\web.config"

        //                     # Check if the file exists
        //                     if (Test-Path $configFilePath) {
        //                         $configContent = Get-Content -Path $configFilePath -Raw

        //                         # Perform the necessary modifications (replace database names)
        //                         $configContent = $configContent -replace "Server=SQLServer,1433;Database=DBDataName;User ID=username; Password=password;", "Server=$SQLSERVER,1433;Database=1BOSS_$folderName;User ID=sa; Password=$SA_PASSWORD;"
        //                         $configContent = $configContent -replace "Server=SQLServer,1433;Database=DBAdminName;User ID=username; Password=password;", "Server=$SQLSERVER,1433;Database=AS_ADMIN_1BOSS_$folderName;User ID=sa; Password=$SA_PASSWORD;"

        //                         # Save the modified content back to the web.config file
        //                         $configContent | Set-Content -Path $configFilePath

        //                         Write-Host "web.config file updated successfully."
        //                     } else {
        //                         Write-Host "The web.config file does not exist in the specified path."
        //                     }
        //                 }
        //                 Remove-PSSession $session
        //             '''
        //             powershell(script: remotePSSession)
        //         }
        //     }
        // }
        stage('Create Task Scheduler'){
            steps{
                script{
                    
                    def remotePSSession = '''
                        $utcPlus7 = Get-Date -Year $env:expireYear -Month $env:expireMonth -Day $env:expireDay -Hour $env:expireHour -Minute $env:expireMinute -Second 0
                        $utc = $utcPlus7.ToUniversalTime()

                        $server = "$env:WEB_SERVER_IP"
                        $uri = "https://$($server):5986"
                        $user = "$env:WEBSERVER_USERNAME"
                        $password = "$env:WEBSERVER_PASSWORD"
                        $securepassword = ConvertTo-SecureString -String $password -AsPlainText -Force
                        $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $user, $securepassword

                        $sessionOption = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
                        $session = New-PSSession -ConnectionUri $uri -Credential $cred -SessionOption $sessionOption
                        Invoke-Command -Session $session -ArgumentList $utc -ScriptBlock {
                            # Define task parameters
                            param($utc)
                            $username = "$using:env:WEBSERVER_USERNAME"
                            $password = "$using:env:WEBSERVER_PASSWORD"
                            $expireMinute = "$using:env:expireMinute"

                            $utcMinus8 = [System.TimeZoneInfo]::FindSystemTimeZoneById("Pacific Standard Time")
                            $convertTime = [System.TimeZoneInfo]::ConvertTime($utc, $utcMinus8) 
                            $convertHour = $convertTime.Hour
                            $convertDay = $convertTime.Day
                            $convertMonth = $convertTime.Month
                            $converYear = $convertTime.Year

                            $taskName = "Delete $deploymentName Site"
                            $triggerStartBoundary = Get-Date -Year $converYear -Month $convertMonth -Day $convertDay -Hour $convertHour -Minute $using:env:expireMinute -Second 0 -Format 'yyyy-MM-ddTHH:mm:ss'
                            $userId = "S-1-5-21-58857817-991352899-1529334289-1002"
                            $logonType = "Password"
                            $runLevel = "Highest"
                            $command = "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe"
                            $arguments = "-File C:\\CleanpUpSite.ps1 $deploymentName"

                            Register-ScheduledTask -TaskName "$taskName" -User "$username" -Password "$password" -Force -InputObject (
                                New-ScheduledTask -Action (
                                    (New-ScheduledTaskAction -Execute $command -Argument $arguments),
                                    (New-ScheduledTaskAction -Execute $command -Argument "-ExecutionPolicy ByPass -Command `"Unregister-ScheduledTask -TaskName '$taskName' -Confirm:`$false`"")
                                ) -Trigger (
                                    New-ScheduledTaskTrigger -Once -At $triggerStartBoundary
                                ) -Principal (
                                    New-ScheduledTaskPrincipal -UserId $userId -LogonType $logonType -RunLevel $runLevel
                                ) -Settings (
                                    New-ScheduledTaskSettingsSet -Priority '7' -Compatibility 'Vista'
                                )
                            )
                        }
                        Remove-PSSession $session
                    '''
                    powershell(script: remotePSSession)
                }
            }
        }

    }

    post {
        always {
            echo 'Finished'
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
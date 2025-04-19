$outputPath = '<filepath for output file>'
$datetime   = Get-Date -f 'yyyyMMddHHmmss'
$filename   = "SyncScriptTranscript-${datetime}.txt"
$Transcript = Join-Path -Path $outputPath -ChildPath $filename
Start-Transcript $Transcript

# store list of all server names from CMS within AG folder
$serverNames = Get-DbaRegServer -sqlinstance <sql instance with CMS> -Group "<CMS Folder with AG listeners>" | Select-Object -unique -expandProperty ServerName

# iterate through array of instances and run below code
	$serverNames | ForEach-Object {		
		# internal variables
			$ClientName = 'DBATools AG Sync'
			$primaryInstance = $null
			$secondaryInstances = @{}
		try {
			# connect to instance, get the name of the primary and all secondaries
				$replicas = Get-DbaAgReplica -SqlInstance $_
				$primaryInstance = $replicas | Where Role -eq Primary | select -ExpandProperty name
				$secondaryInstances = $replicas | Where Role -ne Primary | select -ExpandProperty name
			# create a connection object to the primary
				$primaryInstanceConnection = Connect-DbaInstance $primaryInstance -ClientName $ClientName
		    # get AG
                Get-DbaAvailabilityGroup -SqlInstance $primaryInstanceConnection
			# loop through each secondary replica and sync AG objects
				$secondaryInstances | ForEach-Object {
					$secondaryInstanceConnection = Connect-DbaInstance $_ -ClientName $ClientName
                    Sync-DbaAvailabilityGroup -Primary $primaryInstanceConnection -Secondary $secondaryInstanceConnection -Force -ExcludeLogin "sa","##MS_PolicyEventProcessingLogin##","##MS_PolicyTsqlExecutionLogin##" -Exclude DatabaseMail -EnableException
				}
		}
		catch {
			$msg = $_.Exception.Message
			Write-Error "Error while syncing DBAtools Availability Groups'$($AvailabilityGroupName): $msg'"
		}
	
	}
Stop-Transcript
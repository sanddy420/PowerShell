PowerShell
==========

PowerShell
<#	
	.NOTES
	===========================================================================
	 Created with: 	SAPIEN Technologies, Inc., PowerShell Studio 2014 v4.1.63
	 Created on:   	©2014-08-09 15:08
	 Created by:   	sanddy.hx
	 Filename:     	windows 2008 R2 服务器初始化脚本
	===========================================================================
#>
Set-ExecutionPolicy RemoteSigned -Force
Import-Module Servermanager
#0
function ScriptExit{
	if ($InNumber -eq 0)
	{ exit }
}

#1
function Edit_remoto_port {
	$PortNumber = Read-Host "请输入修改远程桌面端口号"
	Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name 'PortNumber' -Value $PortNumber
	Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp' -Name 'PortNumber' -Value $PortNumber
	$PortResult1 = Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name 'PortNumber'
	$PortResult2 = Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp' -Name 'PortNumber'
	if ($PortResult1.PortNumber -eq $PortNumber -and $PortResult2.PortNumber -eq $PortNumber)
	{
		Write-Host "已经成功修改端口为$PortNumber" -ForegroundColor Green
	}
	else
	{
		Write-Error "端口修改失败，请重试...."
	}
	Write-Host "输入出入站类型（in->入站;out->出站）" -ForegroundColor Red
	$AddTypes = Read-Host "输入出入站类型（in->入站;out->出站）"
	if ($AddTypes -eq 'in')
	{
		$AddType = 'in'
		netsh advfirewall firewall add rule name = "Allow_$AddType_$PortNumber" protocol = TCP dir = $AddType localport = $PortNumber action = allow
		Write-Host  "远程防火墙端口已开启!" -ForegroundColor Green
	}
	elseif ($AddTypes -eq 'out')
	{
		$AddType = 'out'
		netsh advfirewall firewall add rule name = "Allow_$AddType_$PortNumber" protocol = TCP dir = $AddType localport = $PortNumber action = allow
		Write-Host "防火墙出站端口$PortNumber已开启!" -ForegroundColor Green
	}
	else
	{
		Write-Host "远程防火墙端口未开启,请设置!" -ForegroundColor Red
	}
}
#2
function TermServiceStart  {
	Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name 'fDenyTSConnections' -Value 0
	Write-Host 正在重启 Remote Desktop Services .. . -ForegroundColor DarkYellow
	Set-Service TermService -StartupType Automatic -Status Running -PassThru
}
#3
function MyNew-NetFirewallRule{
	#              $Check = New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort $PortNumber -Action Allow
	#              if($Check.PrimaryStatus -eq 'OK')
	#                {
	#                    Write-Host "成功设置防火墙策略" -ForegroundColor Green
	#                }
	#                else
	#                {
	#                    Write-Error "防火墙策略设置失败,请重试..."
	#                }
	Write-Host "输入出入站端口及类型（in->入站;out->出站）" -ForegroundColor Red
	$PortNumber = Read-Host "输入防火墙端口："
	$AddTypes = Read-Host "输入出入站类型（in->入站;out->出站）"
	if ($AddTypes -eq 'in')
	{
		$AddType = 'in'
		Write-Host  "防火墙端口$PortNumber已开启!" -ForegroundColor Green
		netsh advfirewall firewall add rule name = "Allow_$AddType_$PortNumber" protocol = TCP dir = $AddType localport = $PortNumber action = allow
	}
	elseif ($AddTypes -eq 'out')
	{
		$AddType = 'out'
		Write-Host "防火墙出站端口$PortNumber已开启!" -ForegroundColor Green
		netsh advfirewall firewall add rule name = "Allow_$AddType_$PortNumber" protocol = TCP dir = $AddType localport = $PortNumber action = allow
	}
	else
	{
		Write-Host "远程防火墙端口未开启,请设置!" -ForegroundColor Red
	}
	
}
#4
function RestoreRemoDeskPort {
	Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name 'PortNumber' -Value 3389
	Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp' -Name 'PortNumber' -Value 3389
	$PortResult1 = Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name 'PortNumber'
	$PortResult2 = Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp' -Name 'PortNumber'
	if (($PortResult1.PortNumber -eq 3389) -and ($PortResult2.PortNumber -eq 3389))
	{
		Write-Host "已经成功恢复系统默认设置,远程端口为：3389" -ForegroundColor Green
	}
	else
	{
		Write-Error "恢复失败，请重试...."
	}
}
#5
function DenyRemotDeskPort  {
	$RemoDeskTop = Read-Host "您真滴要禁用远程桌面吗?真心的提醒您要小心哦!(输入:Yes或者No)"
	if ($RemoDeskTop -eq 'yes')
	{
		Write-Host 正在停止 Remote Desktop Services .. . -ForegroundColor DarkYellow
		Set-Service TermService -StartupType Disabled -Status Stopped -PassThru
		Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name 'fDenyTSConnections' -Value 1
	}
	else
	{
		write-host "小伙儿,您真小心!Oh...真好,我可以继续远程了^_^" -ForegroundColor 'green'
		Start-Sleep 1
	}
}

#6
function InstallRolesService {
	#Web-Server, Web-WebServer, Web-Common-Http, Web-Static-Content, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Http-Redirect, Web-DAV-Publishing, Web-App-Dev, Web-Asp-Net, Web-Net-Ext, Web-ASP, Web-CGI, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Includes, Web-Health, Web-Http-Logging, Web-Log-Libraries, Web-Request-Monitor, Web-Http-Tracing, Web-Custom-Logging, Web-ODBC-Logging, Web-Security, Web-Basic-Auth, Web-Windows-Auth, Web-Digest-Auth, Web-Client-Auth, Web-Cert-Auth, Web-Url-Auth, Web-Filtering, Web-IP-Security, Web-Performance, Web-Stat-Compression, Web-Dyn-Compression, Web-Mgmt-Tools, Web-Mgmt-Console, Web-Scripting-Tools, Web-Mgmt-Service, Web-Mgmt-Compat, Web-Metabase, Web-WMI, Web-Lgcy-Scripting, Web-Lgcy-Mgmt-Console, Web-Ftp-Server, Web-Ftp-Service, Web-Ftp-Ext, Web-WHC,File-Services, FS-FileServer, FS-DFS-Replication,AS-TCP-Activation, AS-Named-Pipes,Remote-Desktop-Services, RDS-RD-Server, RDS-Licensing,NET-Framework, NET-Framework-Core, NET-Win-CFAC, NET-HTTP-Activation, NET-Non-HTTP-Activ,SNMP-Services, SNMP-Service, SNMP-WMI-Provider,WAS, WAS-Process-Model, WAS-NET-Environment, WAS-Config-APIs,RSAT, RSAT-Role-Tools, RSAT-File-Services, RSAT-DFS-Mgmt-Con, RSAT-RDS, RSAT-RDS-RemoteApp, RSAT-RDS-Licensing, RSAT-Web-Server, RSAT-Feature-Tools, RSAT-NLB
	add-windowsfeature  Web-Server, Web-WebServer, Web-Common-Http, Web-Static-Content, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Http-Redirect, Web-DAV-Publishing, Web-App-Dev, Web-Asp-Net, Web-Net-Ext, Web-ASP, Web-CGI, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Includes, Web-Health, Web-Http-Logging, Web-Log-Libraries, Web-Request-Monitor, Web-Http-Tracing, Web-Custom-Logging, Web-ODBC-Logging, Web-Security, Web-Basic-Auth, Web-Windows-Auth, Web-Digest-Auth, Web-Client-Auth, Web-Cert-Auth, Web-Url-Auth, Web-Filtering, Web-IP-Security, Web-Performance, Web-Stat-Compression, Web-Dyn-Compression, Web-Mgmt-Tools, Web-Mgmt-Console, Web-Scripting-Tools, Web-Mgmt-Service, Web-Mgmt-Compat, Web-Metabase, Web-WMI, Web-Lgcy-Scripting, Web-Lgcy-Mgmt-Console, Web-Ftp-Server, Web-Ftp-Service, Web-Ftp-Ext, Web-WHC, File-Services, FS-FileServer, FS-DFS-Replication, AS-TCP-Activation, AS-Named-Pipes, Remote-Desktop-Services, RDS-RD-Server, RDS-Licensing, NET-Framework, NET-Framework-Core, NET-Win-CFAC, NET-HTTP-Activation, NET-Non-HTTP-Activ, SNMP-Services, SNMP-Service, SNMP-WMI-Provider, WAS, WAS-Process-Model, WAS-NET-Environment, WAS-Config-APIs -whatif
	Start-Sleep 60
	add-windowsfeature  Web-Server, Web-WebServer, Web-Common-Http, Web-Static-Content, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Http-Redirect, Web-DAV-Publishing, Web-App-Dev, Web-Asp-Net, Web-Net-Ext, Web-ASP, Web-CGI, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Includes, Web-Health, Web-Http-Logging, Web-Log-Libraries, Web-Request-Monitor, Web-Http-Tracing, Web-Custom-Logging, Web-ODBC-Logging, Web-Security, Web-Basic-Auth, Web-Windows-Auth, Web-Digest-Auth, Web-Client-Auth, Web-Cert-Auth, Web-Url-Auth, Web-Filtering, Web-IP-Security, Web-Performance, Web-Stat-Compression, Web-Dyn-Compression, Web-Mgmt-Tools, Web-Mgmt-Console, Web-Scripting-Tools, Web-Mgmt-Service, Web-Mgmt-Compat, Web-Metabase, Web-WMI, Web-Lgcy-Scripting, Web-Lgcy-Mgmt-Console, Web-Ftp-Server, Web-Ftp-Service, Web-Ftp-Ext, Web-WHC, File-Services, FS-FileServer, FS-DFS-Replication, AS-TCP-Activation, AS-Named-Pipes, Remote-Desktop-Services, RDS-RD-Server, RDS-Licensing, NET-Framework, NET-Framework-Core, NET-Win-CFAC, NET-HTTP-Activation, NET-Non-HTTP-Activ, SNMP-Services, SNMP-Service, SNMP-WMI-Provider, WAS, WAS-Process-Model, WAS-NET-Environment, WAS-Config-APIs
	Start-Sleep 60
}

#7
function JoinDomain{
	$DomainName = [System.Net.NetworkInformation.IPGlobalProperties]::GetIPGlobalProperties().DomainName
	if ($DomainName.Length -gt 0)
	{
		Write-Host "此服务器已加入域,请先退域后再加入新域!" -ForegroundColor 'Red'
		return
	}
	elseif ($DomainName.Length -eq 0)
	{
		$domain = Read-Host "请输入你所要加入的AD域名："
		Add-Computer -DomainName $domain
		Restart-Computer -ComputerName .
	}
}

#8
function LevelDomain{
	$DomainName = [System.Net.NetworkInformation.IPGlobalProperties]::GetIPGlobalProperties().DomainName
	if ($DomainName -is [string])
	{
		Write-Host "此服务器即将退出域,请确认!" -ForegroundColor 'Red'
		$inputY2N = Read-Host "请输入Yes or No!"
		if ($inputY2N -eq 'yes')
		{
			$DomainUserAdmin = Read-Host "请输入域管理员帐号"
			remove-computer -credential $DomainUserAdmin -passthru -verbose
			#						Restart-Computer -ComputerName .
		}
		elseif ($inputY2N -eq 'no')
		{
			return
		}
	}
}

#9
function CheckInstalledProgramsAndServices
{
	Get-WmiObject -Class Win32_Product | Select-Object -Property Name | Sort-Object -Property Name
	Start-Sleep 60
}

#10
function UnInstalledProgramsAndServices{
	$appName = Read-Host "输入要卸载的程序名"
	
	$apps = Get-WmiObject -Class Win32_product | Where-Object { $_.Name -match $appName }
	if (!$apps)
	{
		write-host "你输入的的程序&服务不存在: $appName" -ForegroundColor red
	}
	$i = 1
	$n = $apps.length;
	foreach ($a in $apps)
	{
		echo $a.Name "($i / $n ) "
		$a.Uninstall()
		echo "Finish uninstall "
		$i++
	}
	Start-Sleep 2
}

#11
function IIS_Site_And_Configuration{
	Import-Module WebAdministration  #导入IIS模块
	# 建立IIS站点所用参数
	[int]$sitePort = read-host 'Web端口'
	[string]$SiteName = read-host '站点名'
	[string]$SiteAppPools = $SiteName  #进程池名
	[string]$SiteAppPoolsModel = read-host "输入<Classic │ Integrated>"  #进程池使用通道模式
	[int]$AppPoolType = read-host "指定程序池的帐户标识（0>Local Service 1>Local System  2>Network Service  3>User 4>ApplicationPoolIdentity）"
	[string]$managedRuntimeVersion = read-host ".net版本 (v2.0|v4.0)"  #.net版本
	[int]$MaxprivateMemory = Read-Host "指定专用内存大小（2G=<2097152>;4G=<4194304>）："
	[string]$WebSitePath = read-host "站点程序路径" #站点程序路径
	[string]$HostHeader1 = read-host "绑定站点第一个域名" 	 #绑定站点域名
	[string]$HostHeader2 = read-host "绑定站点第二个域名" 	 #绑定站点域名
	[string]$HostHeader3 = read-host "绑定站点第三个域名"
	[string]$IISLogFile = "d:\LogFiles\IIS\$SiteName" #IIS日志路径
	$net32Or64 = $true #是否启用.net32模式
	
	
	#创建IIS应用程序池
	function BuildAppPool()
	{
		$appPoolName = "$SiteAppPools"
		$app = "iis:\AppPools\" + $appPoolName
		$existAppPool = Test-Path $app
		
		if ($existAppPool -eq $false){
			$appPool = New-WebAppPool  $appPoolName
			#Identity --> Network service
			$appPool.ProcessModel.IdentityType = $AppPoolType
			#.net 4
			$appPool.managedRuntimeVersion = "$managedRuntimeVersion"
			#PopelineMode
			$appPool.ManagedPipelineMode = $AppPoolType
			#enable32BitAppOnWin64
			$appPool.enable32BitAppOnWin64 = $true
			#start mode
			$appPool.startMode = "AlwaysRunning"
			#managedPipelineMode
			$appPool.managedPipelineMode = "$SiteAppPoolsModel"
			#autoStart
			$appPool.autoStart = $false
			#loadUserProfile
			$appPool.processModel.loadUserProfile = $true
			#pingingEnabled
			$appPool.processModel.pingingEnabled = $true
			#privateMemory
			$appPool.recycling.periodicRestart.privateMemory = $MaxprivateMemory
			#periodicRestart.time
			$appPool.recycling.periodicRestart.time = "1.00:00:00"
			#save property
			$appPool | Set-Item
			}
		elseif ($existAppPool -eq $true){
			echo "$SiteAppPools AppPool is Already exists"		
			#Identity --> Network service
			$appPool.ProcessModel.IdentityType = $AppPoolType
			#.net 4
			$appPool.managedRuntimeVersion = "$managedRuntimeVersion"
			#PopelineMode
			$appPool.ManagedPipelineMode = $AppPoolType
			#enable32BitAppOnWin64
			$appPool.enable32BitAppOnWin64 = $true
			#start mode
			$appPool.startMode = "AlwaysRunning"
			#managedPipelineMode
			$appPool.managedPipelineMode = "$SiteAppPoolsModel"
			#autoStart
			$appPool.autoStart = $false
			#loadUserProfile
			$appPool.processModel.loadUserProfile = $true
			#pingingEnabled
			$appPool.processModel.pingingEnabled = $true
			#privateMemory
			$appPool.recycling.periodicRestart.privateMemory = [int]99999
			#periodicRestart.time
			$appPool.recycling.periodicRestart.time = "1.00:00:00"
			#save property
			$appPool | Set-Item
		}
	}
	
	#创建IIS应用站点
	function BuildSite()
	{
		$appSitePath = "iis:\sites\" + $SiteName
		$existweb = Test-Path $appSitePath
		if (!$existweb)
		{
			New-Website -name $SiteName -Port ${sitePort} -ApplicationPool $SiteAppPools -PhysicalPath $WebSitePath
			New-WebBinding -Name $SiteName -Port ${sitePort} -HostHeader ${HostHeader1}
			New-WebBinding -Name $SiteName -Port ${sitePort} -HostHeader ${HostHeader2}
			New-WebBinding -Name $SiteName -Port ${sitePort} -HostHeader ${HostHeader3}
#			.$Env:windir\system32\inetsrv\appcmd.exe set site $SiteName /bindings:"http/*:${sitePort}:${HostHeader1},http/*:${sitePort}:${HostHeader2},http/*:${sitePort}:${HostHeader3}"
			.$Env:windir\system32\inetsrv\appcmd.exe set config /section:directoryBrowse /enabled:false
		}
		else
		{
			echo "'$SiteName' is Already exists"
		}
	}
	
	#设置IIS日志记录路径
	function CreatIISLogFile()
	{
		.$Env:windir\system32\inetsrv\appcmd.exe set site $SiteName "-logfile.directory:$IISLogFile"
	}
	
	#为F5设备创建ISPAI筛选器
	function CreatISP()
	{
		$x = [string](.$Env:windir\system32\inetsrv\appcmd.exe list config $SiteName /section:isapiFilters)
		if ($x -like "*F5XForwardedFor*")
		{
			echo "isapiFilters is Already exists"
		}
		else
		{
			.$Env:windir\system32\inetsrv\appcmd.exe unlock config $SiteName "-section:system.webServer/isapiFilters";
			.$Env:windir\system32\inetsrv\appcmd.exe set config $SiteName /section:isapiFilters /"+[name='F5XForwardedFor',path='$Env:windir\System32\F5XForwardedFor.dll',enabled='true']";
		}
	}
	
	function RunBuild()
	{
		BuildAppPool
		BuildSite
		CreatIISLogFile
#		CreatISP
#		.$Env:windir\system32\inetsrv\appcmd.exe start site $SiteName
		Start-Website -Name $SiteName
	}
	RunBuild
}

#12
Function Set-Password{
	[string]$computer = $env:computername
	[string]$account = "Administrator"
	[string]$password = Read-Host "请输入密码，并牢记!"
	
	
	$errorActionPreference = "silentlyContinue"
	
	[adsi]$user = "WinNT://$computer/$account,user"
	#get current password age
	[int]$oldage = $user.passwordage[0]
	if ($user.name)
	{ if ($password.Length -gt 0)
		{
			$user.SetPassword($password)
			Write-Host "你的密码修改成功,密码为:$password,请牢记!" -ForegroundColor 'Red'
			sleep -milliseconds 500
		}
		elseif ($password.Length -eq 0) {
			Write-Host "你输入的密码为空,请牢记!" -ForegroundColor 'Red'
			sleep -milliseconds 251
		}
	}
	else
	{
		$msg = "Failed to get $account on " + $computer.toUpper()
		Write-Warning $msg
		return
	}
	
	#刷新对象和检查以确保通过
	#password was changed
	$user.psbase.refreshcache()
	if ($user.passwordage -gt $oldage)
	{
		$msg = "Failed to change password for $account on " + $computer.toUpper()
		Write-Warning $msg
	}
	
}

##########################################################################################################################
function ServerManageAuto
{
	while ($InNumber -ne 0)
	{
		Write-Host " ##########################################################" -ForegroundColor Green
		Write-Host " # 1、自定义远程桌面端口并开启防火墙端口；                #"
		Write-Host " # 2、开启远程桌面；                                      #"
		Write-Host " # 3、开启防火墙远程桌面出入端口；                        #"
		Write-Host " # 4、恢复系统默认的远程桌面端口；                        #"
		Write-Host " # 5、禁用远程桌面；                                      #"
		Write-Host " # 6、安装角色服务；                                      #"
		Write-Host " # 7、服务器加域；                                        #"
		Write-Host " # 8、服务器退域；                                        #"
		Write-Host " # 9、查看服务器已安装程序及服务；                        #"
		Write-Host " # 10、卸载已安装程序及服务；                             #"
		Write-Host " # 11、IIS建站及配置；                                    #"
		Write-Host " # 12、修改Administrator用户密码；                        #"
		Write-Host " # 0、退出菜单 &  Exit；                                  #"
		Write-Host " # ########################################################" -ForegroundColor Green
		$InNumber = Read-Host "输入编号(0-12)进行相应操作"
		
		switch ($InNumber)
		{
			0 { ScriptExit }
			1 { Edit_remoto_port }
			2 { TermServiceStart }
			3 { MyNew-NetFirewallRule }
			4 { RestoreRemoDeskPort }
			5 { DenyRemotDeskPort }
			6 { InstallRolesService }
			7 { JoinDomain }
			8 { LevelDomain}
			9 { CheckInstalledProgramsAndServices}
			10 { UnInstalledProgramsAndServices}
			11 { IIS_Site_And_Configuration }
			12{ Set-Password}
			Default { Write-Error "请输入相对应操作编号:" }
		}
		Start-Sleep 2
#		Invoke-Command {cls}
	}
}

ServerManageAuto

Disk Low Level Discovery for Physical Disk within Windows Performance Monitoring in Zabbix 2.0
==========================

This Powershell script returns JSON for Windows Perfomance Monitoring within Zabbix.

This powershell script discovers the logical disks letters and partition index number for a windows machine. The returned values can be used to enable Zabbix to autodiscover all the individual disks in the following format: 

[partition number] [drive letter]:

0 C:

Note: If there are multiple partitions on a physical disk, this script will only return the first drive letter that it finds on that physical disk. We want to create items and monitor drives like something like 0 C: and 1 E:  We do not want to create items for 0 C: and 0 E: on the same host as this will throw an error. 

In order to monitor the physical disks for Windows performance monitoring within Zabbix as user must set up items for each individual disk. This is cumbersome as you must know the drive letter and index number for each Windows machine. Normal item syntax for physical disks within the Windows performance monitor in Zabbix looks like the following:

perf_counter[\234(0 C:)\208]

\234 Represents the Physical Disk.

\208 Represents Ave. Disk seconds/Read.

The conversion values are located in the registry.

Run regedit from the command line on your windows machine. Then navigate to the HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Perflib\009. 

I would personally copy the contents of the file and put it into a text document if you want to reference them later. The numeric representation of the Windows Performance Monitoring metrics is better if you work in an environment where the machines may have different locales (i.e. Languages) because the Windows Perfomance Monitoring strings change based on locale.

The most relevant numeric representation is \234.  

Our script returns JSON (JavaScript Object Notation) it will look like this.

{
  "data":[
  
  { "{#DISKNUMLET}":"0 C:" },
  
  ]
}

You must create a UserParameter somewhere in the zabbix_agent.win.conf or zabbix_agent.conf file. It depends on what you named the file initially. 

UserParameter=windowsPhysicalDisk.Discovery, powershell -File c:\diskdisc.ps1

Then restart the agent.

When you make a prototype item you must use the defined macro in the JSON object, in this case {#DISKNUMLET}.

It will look something like this:

perf_counter[\234(#DISKNUMLET)\208]

If you add a physical disk to the computer and the populated discovered items become unsupported, restart the agent. They should become supported and begin transmitting metrics once again. 

For addional Zabbix Windows Performance Monitoring, please visit the following website:

https://www.zabbix.com/documentation/2.0/manual/config/items/perfcounters

For more information about Zabbix Low Level Discovery visit this site:

https://www.zabbix.com/documentation/2.0/manual/discovery/low_level_discovery


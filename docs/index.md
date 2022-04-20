## Welcome to my GitHub Pages!

I'm currently in Microsoft's Softwarse and Systems Academy where I will have spent 5 months learning multiple skills such as PowerShell scripting and automation, Active Directory management, and fundamentals of Microsoft's Azure platform. I hope that through these GitHub Pages, I can showcase the steps I've taken to learn new skills and demonstrate my ability to not only learn, but also teach and explain how I accomplished the task. [qrcode_linkedin](https://user-images.githubusercontent.com/81198298/164302570-bc0b56ad-746e-4f82-8ce8-cc70ea6d9065.png)


Below I will be showcasing what I've done for the MSSA Challenge labs. I will include the prompt, interperate my tasks, and then show the code I used and my thought process behind it all.

**Challenge Lab - Windows Server - PowerShell Administration**

Overview - In this lab, you will be challenged with your understanding of PowerShell to manage and control your Windows computer.

Contoso Finance has recently begun a project to better understand what services and applications are running on their systems. 
Their management system is based around scripting and procedural steps while ensuring the staff can standard and repeatable steps of their systems. 
Therefore, they are electing to utilize default Windows PowerShell Modules and PowerShell scripting to administer their systems. 
They have reached out for assistance in developing and setting up commands and scripts for them to utilize.

## Part 1
### List the commands used for services in the Management module - Count of services on the windows server - Command(s) to restart the print spooler service on the local computer

_From this, I need to list the commands that deal with services, count all the services, and be able to restart the print spooler_
```
help *-service
# This uses the wild card to look at all the commands that end with '-service'.
# Since PowerShell uses the Verb-Noun format in it's commands, this will get all the actions that can be taken against the service module

get-service.count
# Runs Get-Service and then counts all the results

restart-service Spooler
# Restarts the Print Spooler, pretty self-explanitory
```

## Part 2
### Use the Get-Service command to list services but filter out the results where the status is running directly from PowerShell commands. For the report of details for all the running services, in consideration of concise output, ensure that only these key fields are included in the output: - Name - Status - Startup Type - File Location!


_From this, I need to list all services that are running and get a list of specified details_

```
get-service | where Status -eq Running | format-table
# Gets all running services

Get-wmiobject win32_service | Where Status -eq Running | Select Name,Status,StartMode,PathName | Format-Table
# Gets all running services, clears that list down to the ones that are running, and then shows the specified paramaters of that list
```

## Part 3
### You need to put the list of running services out to one file and the details need to be kept in a separate folder. The running services should be stored in HTML format so that it can be easily pulled into web status page. The details file should be stored in a JSON file so it is easy to work with later on. These folders need to keep different ones for each day for at least 30 days back in history. The output for this can be stored in a temporary folder or create a new folder for it, but there should be a consistant path for retrieving them.

_This is where it starts to get fun - The tasks I see are to store the running services in an HTML file, and store the details in a JSON file. Also need to be abe to retrieve files that are created within the past 30 days_

```
param 
(
$HTMLPath = 'C:\Users\demouser\Desktop\HTMLLogs',
$JSONPath = 'C:\Users\demouser\Desktop\JSONLogs'
$Time = 'Get-Date -Format yyyymmdd_hhmmtt'
)
# These params are the paths that I will be saving the files to and a time-date variable for the date stamp
# I always find that using paramaters makes my life easier. I'm lazy so I like to make params early and often so i have to type less. #SorryNotSorry

if (-not (test-path $HTMLPath -pathtype any)){New-Item -path $HTMLPath -ItemType Directory}
if (-not (test-path $JSONPath -pathtype any)){New-Item -path $JSONPath -ItemType Directory}
# These blocks test to see if a folder has been created in the proposed directory and creates one if there isn't one already
# This allows for a little more modularity and whoever is using this can change the params and still make things work

get-service | where status -eq running | convertto-html | Out-File "$HTMLPath\HTMLLog-$time.html"
get-wmiobject win32_service | select name,status,startmode,pathname | convertto-json | out-file "$JSONPath\JSONLog-$time.json"
# This block takes the commands from Part 2 and saves them in the paths I declared earlier with a time stamp at the end

Get-ChildItem -path $HTMLPath | ? { $_.CreationTime -ge (get-date).adddays(-30)}
Get-ChildItem -path $JSONPath | ? { $_.CreationTime -ge (get-date).adddays(-30)}
# This block checks to see all files that were created up to 30 days ago
```


## Part 4
### After exporting information for files, they’d also like to look for files on the file system. They would like to confirm that the output files were created for today. If there are no files created today for the jobs, the would like to write out an error that the files were not found. Furthermore, they would like to revisit printer spooler issue from Exercise 1. They would like to look for any files in the printer spool folder that are too old. Normally successful print jobs complete in an hour so if there are files in the queue over 6 hours old, they would like to restart the spooler service.

_Little verbose, but the three tasks are to check if files were created today, have a success/failure message, and check the spooler for if files are over 6 hours old_

```
param 
(
$HTMLPath = 'C:\Users\demouser\Desktop\HTMLLogs',
$JSONPath = 'C:\Users\demouser\Desktop\JSONLogs',
$time = 'get-date -format yyymmdd_hhmmtt'
)
# Same thing as before, creating params to set paths that are going to be checked and setting a time-date variable

if($CheckHTML = Get-ChildItem -path $htmlpath | ? { $_.CreationTime -eq (Get-Date)})
{write-host "Great Success, files are here!"}
else {write-host "Error: No files"}
if($CheckJSON = Get-ChildItem -path $jsonPATH | ? { $_.CreationTime -eq (Get-Date)})
{write-host "Great Success!"}
Else {write-host "Error: No files here"}
# Uses if statement to check the JSON and HTML paths to see is files are created today and gives a message depending if there's files or not

$CheckSpooler = Get-ChildItem -path C:\Windows\System32\spool\PRINTERS\ | ? { $_.CreationTime -le (get-date).addhours(-6)}
if ($CheckSpooler){restart-service spooler | write-output "Spooler hung & restarted"}
# Making the $CheckSpooler veriable is a little unnessicary, but I think it makes the execution command easier to read and this is my code so I'm going to do what I want.
````

## Part 5
### These commands have been very successful and Contoso is happy with them. It is now important to encorporate these commands into usuable script files (PS1). In addition, it is best to use parameters to increase the flexibility of the files. 
###Create files around these tasks from previous exercises:
###Export running services to a file and export details of all services to another file with paramater path
###Verify that the files were created today


```
param 
(
$HTMLBasePath = 'C:\Users\demouser\Desktop\HTMLLogs',
$JSONBasePath = 'C:\Users\demouser\Desktop\JSONLogs',
)
# Sets the base path for where directories are going to be created

$day = (Get-Date -Format dd)
$month = (Get-Date -format MM)
$year = (Get-Date -format yy)
$hour = (get-date -format hhmmtt)

$HTMLPath = "$HTMLBasePath\Year 20$year\Month $Month\Day $Day"
$JSONPath = "$JSONBasePath\Year 20$year\Month $Month\Day $Day"
# Sets full path where files will be stored in a directory with sub-folders according to the date

if (-not (test-path $HTMLPath -pathtype any)){New-Item -path $HTMLPath -ItemType Directory}
if (-not (test-path $JSONPath -pathtype any)){New-Item -path $JSONPath -ItemType Directory}
# Takes the specified path and check to see if a directory is there and creates one if not

get-service | where status -eq running | convertto-html | Out-File "$HTMLPath\HTMLLog_$hour.html"
get-wmiobject win32_service | select name,status,startmode,pathname | convertto-json | out-file "$JSONPath\JSONLog_$hour.json"
Combines all of hte above by spitting out running services and details into their HTML and JSON directories respectively
```

## Part 5.1
###Check for print spool files and if anything too old, restart the print spooler. Use a Paramater for the timeout length
_The prompt wants another script for the print spooler check so here's another block of code_

```
param 
(
$SpoolTimeout = '(Get-Date).addhours(-6)',
$RestartSpooler = 'Restart-Service Spooler',
)
# Sets the params for the code below, time is able to be changed if need be. 

$CheckSpooler = 'Get-ChildItem -path C:\Windows\System32\spool\PRINTERS\ -recurse | ? { $_.CreationTime -le $SpoolTimeout}'
if ($CheckSpooler){$RestartSpooler | write-output "Spooler hung & restarted"}
#Essentially the same code as the first time I checked for files, but I added the 

```

### Lab - Setup the Development Environment


@@@secondary
**Scenario**

As one of the developers at Contoso, setting up your development environment is an important step before starting to build in IoT solution. Azure IoT offers multiple developer tools and support across top IDEs.

In this lab you will:

* Download the lab files.
@@@

#### Exercise 1: Install Developer Tools and Products

@@@danger
**Note**: The software in the tasks has already been installed for you.  However, you do need to download the lab files in Task 0.
@@@

##### Task 0: Download labfiles.

1. [ ] From the lab virtual machine, click **Start** and search for **PowerShell** then open **PowerShell as Administrator**.

1. [ ] Run the following commands to download the latest version of the lab files to the virtual machine.
@@@warning
**Note**: If any of the commands fail run them again until they are succesfull.
@@@


  ```powershell
Import-Module -Name BitsTransfer
 ```
   ```powershell
MD C:\Labfiles
 ```
  ```powershell
Start-BitsTransfer -Source 'https://github.com/MicrosoftLearning/AZ-220-Microsoft-Azure-IoT-Developer/archive/master.zip' -Destination C:\LabFiles
 ```
 ```powershell
Expand-Archive -Path 'C:\LabFiles\master.zip' -DestinationPath 'C:\LabFiles'
 ```
  ```powershell
Move-item -Path "C:\LabFiles\AZ-220-Microsoft-Azure-IoT-Developer-master\AllFiles\*" -Destination "C:\LabFiles" -confirm:$false

 ```
Your development environment should be now setup!
 
@@@success
**Congratulations**! you have now completed this lab!
@@@

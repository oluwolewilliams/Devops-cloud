# DOCUMENTATION FOR LINUX PROJECT

## This project shows the implementation of some command line operations in linux regarding aspects such as file manangement, directory navigation, user and permission management, package installation and network configuration.

## Pre Installations and Dependencies
Please note that the project is being executed with a windows system and to obtain a linux machine for the project, we need to download and install the following:
1. Oracle VM VirtualBox Manager:
Below is the outcome after downloading and installing VirtualBox from www.virtualbox.org

![Oracle VBox](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9451271b-d3f9-4261-b7d4-634d2d990bd4)

2. Ubuntu Linux: Below is the outcome after downloading the Ubuntu Linux Disc Image File and installing on the Virtual Machine i.e Oracle VirtualBox.
   
   ![ubuntu linux environment](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a4c9767c-26af-4393-a5a4-200cc228b43e)

3. Below shows the Linux Command Line Interface fully operational and ready for use:
   
   ![Linux terminal](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c305d9f6-237b-4ead-a80e-5a4f9d4b5976)
   

## LINUX COMMANDS

## 1. sudo Command
sudo is shortform for "superuser do". It will enable you perform tasks that require administrative rights.
To perform an administrative task such as installing the available upgrades of all packages currently installed on the system , you run the following command:

`sudo apt upgrade`

After running this command, the system will prompt us to authenticate ourselves with a password. Here, if we enter the password incorrectly, or the current user is not part of the sudo group (as shown in the image below), the system will log the activity as a security event.

![sudo error](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5cdc7ee4-a811-40ae-8ca6-aea998ab2dbb)

To fix the error in the image above, we can enter the command below and enter the root password to switch to the root user:

`su`

And then, we can subsequently rerun the `sudo apt upgrade` command. Below is the correct output:

![sudo correct](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1d06bfb8-efee-4051-a870-39a0e9980b87)


## 2. pwd Command
This command shows the path of our current working directory. Enter the command as:

`pwd`

The output for this command is as shown below:

![pwd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/df80777c-cc7a-43a6-a077-1349e6da7e8b)


## 3. cd Command
We can use this command to change from our current working directory to another or we can use it to navigate to a folder by entering the command with the folder name (DevOps_folder) as shown below:

`cd DevOps_folder`

The image below represents the expected outcome:

![cd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/83510595-cd46-482c-bcd9-b2054b240ef6)

cd can also be used with flags. `cd -` to go to the last directory we were in. `cd ..` to go one directory up or we can enter `cd` alone to navigate back to the default home directory. The outputs of these commands should look like this: 

![cd flags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2be20299-5ddb-4052-910e-56934be6a074)


## 4. ls Command
Here, we use this command to list the files and folders in our system. Executing the command without any parameters shows the contents of our current working directory. We can achieve this by simply entering:

`ls`

The output we get is shown below:

![ls](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cb5d5ddb-4c4c-4cff-84b6-00cf22b6a3c1)

Wwe can also use ls to see the files in another directory by specifying the path. From root user, we can see the files in the home directory of vboxuser by entering: 

`ls /home/vboxuser`

Although we are still in the root directory, we are able to see the content of the specified path:

![ls root](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c01a1c84-4e4f-463a-9f78-4c5b9042a746)

The ls command has some flags that can extend its functionality. `ls -R` lists all the files in the sub directories as shown below:

![ls -R](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce7fa8dd-8c4e-4945-bfb1-096a66736e65)

As shown in the next image, entering the command: `ls -a` lists all the files along with the hidden files while `ls -lh` lists all the files in a more easily readable format whilst showing more detail.

![ls -a](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3c0d49e-9115-4be4-aa1b-fbde2bf6ff2d)


## 5. cat Command
We can use this to list, combine or write a file's content on to the Command Line Interface. So to show what we wrote in the DevOps file we simply enter the following command:

`cat DevOps`

The output of this command is as shown in the image below:

![cat](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/81189f4f-a763-476e-94e9-8ee48e74e6bd)


## 6. cp Command
We use this to copy files or folders and their contents to another location. It is used by entering the command, followed by the file name and then the destination directory. For instance, here we want to copy the DevOps file to the Desktop directory, so we execute the following command:

`cp DevOps Desktop/`

The command does not return any value on the standard output or Command Line Interface (CLI) as the operation has already beeen carried out in the background but for confirmation, we can run the `ls` command to list the files in the desktop directory and in the image below, we can see that the DevOps file has been successfully copied.

![cp](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9ebe1975-5ce4-4c17-83a7-eec3a1fe855a)


## 7. mv Command
This is the command we will use to move or rename files.  We can use it by entering the command, followed by the filename and then the destination directory. Here we want to move the DevOps2 folder to the Desktop folder, so we execute the following command:

`mv DevOps2 Desktop/`

Just like the `cp` command, there is no output to the CLI when `mv` is executed. But we can use ls to see its effect as shown in the image below. We see that the DevOps2 file has been removed from it's original location in the home directory and its current location is now the Desktop folder:

![mv](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/73eb7d7a-0571-4ced-82ac-c2e353639994)

To show how the `mv` command is used to rename a file, we simply enter mv and specify the filename (DevOps2) and its path followed by the new filename (DevOps_New) and its path:

`mv Desktop/DevOps2 Desktop/DveOps_New`

As mentioned earlier, the command does not return any value but by using the `ls` command, we can see its output. DevOps2 has now been renamed to DevOps_New:

![mv2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5a552f09-e7b7-4154-a5bb-5e1b576beac3)


## 8. mkdir Command

This is a command we can use to create one or multiple directories at once. We can also use a flag to set permissions for the directories as they are being created. For usage, you need to type the command and specify the directory name to be created as shown in the image below:

`mkdir DevOps_folder`

The output for this can be viewed with the `ls` command:

![mkdir](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/095866aa-d026-4e50-bd7a-5a334bc33894)

To create a folder called inside_folder inside the DevOps_folder:

`mkdir DevOps_folder/inside_folder/`

To create a folder called second_inside_folder inside the inside_folder:

`mkdir DevOps_folder/inside_folder/second_inside_folder`

To view the output we can use `cd` to change to the created directory and then we use `ls` to view its content:

![mkdir3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67407461-594c-4d8c-902b-06cc08db42e8)

We can also use flags with this command. To create a directory named movies with full read, write and execute permissions for all users, we enter the following:

`mkdir -m777 movies`

The output for this is shown below:

![mkdir4](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0614b866-aa92-4930-a6a4-ca152f5f9619)


## 9. rmdir Command

This command can be used to permanently remove or delete an empty directory. To illustrate this, we will delete second_inside_folder using the following command:

`rmdir DevOps_folder/inside_folder/second_inside_folder`

The command when entered does not return any value. However as shown in the output below, using the `ls` command to check the inside_folder directory shows that scond_inside_folder has been removed.

![rmdir](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/45a9af1f-8fd8-455c-b0ab-a1529a7eadba)


## 10. rm Command

We can use this to remove or delete files within a directory. To remove DevOps_file, we proceed to enter the command as:

`rm DevOps_file`

In the image below, the `ls` command is used to show that the DevOps_file file has been deleted:

![rm](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/18ac5b63-7069-4fd9-a310-b460ca62856c)

The command can also be used to remove folders recursively when the `-r` flag is added. i.e it deletes the folder and its contents:

`rm -r DevOps_folder`

The output is shown below:

![rm2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce0f1472-fb63-42db-918a-b7c02f1114f0)


## 11. touch Command

This creates an empty text editor file. We will implement this command by creating a file named DevOps. This is done by executing the following command:

`touch DevOps`

The output is shown below:

![touch2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/029bbcf0-203e-43e3-9c26-4fac793d493b)


## 12. locate Command

We can use this to find any file in the database system. Here, we simply implement it by entering the command, followed by the filename:

'locate DevOps'

The location for the DevOps file is shown on the terminal interface:

![locate](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6919495e-363b-4408-9c9f-bf5b74c39ac8)

## 13. find Command

We can use this to search for and show the location of files within a specific directory. It is usually used with `-name` flag to specify the filename. To look for the DevOps file in the home directory, we will enter the following command:

`find /home -name DevOps`

The output is returned on the standard output or CLI and the path for the DevOps file is shown:

![find](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/81d81124-c8a9-4f06-ad0f-a380917f3e77)


## 14. grep Command

This will let us find a word by searching through all the texts in a specific file. To implement it, we will use the command to show all the instances of the word: 'blue' written in the DevOps file:

`grep blue DevOps`

The command outputs all instances of 'blue' to the CLI as shown in the image below:

![grep](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/68beb553-2247-4275-a3f0-f8a6de8597b5)


## 15. df Command

This reports the system's diskspace usage in percentage and kilobytes. It can be used in combination with different flags to extend functionality. For instance, to show the current directory's system disk space usage in a human readable format, we enter the following command:

`df -h`

The output for this is as shown below:

![df](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/59b860b2-94e4-4a01-a689-08350df0410b)


## 16. du Command

This command will allow us to see how much a file or directory takes up. We shall specify the directory path in the use of this command as shown below:

`du Desktop/learning_linux`

Entering the command outputs the following:

![du](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/aab37c3e-0f0e-4505-b245-2b4bf068ace8)


## 17. head Command

Entering this command will enable us view the upper part or the first 10 lines of a text file. For instance to show the top part of the learning_linux file we enter the following command:

`head learning_linux`

The output of this command is shown in the image below:

![head](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/aebc6bfd-7390-4cc9-bad7-a6dc8f4c11c0)


## 18. tail Command

Entering this command will enable us view the lower part or the last 10 lines of a text file. For instance to show the lower part of the learning_linux file we enter the following command:

`tail learning_linux`

The output of this command is shown in the image below:

![tail](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/89a613f5-a33b-4e16-ac58-517af1d9ddfc)


## 19. diff Command

This command compares the contents of two files line by line and it displays the parts that do not match. To implement this, we proceed to compare the contents of the DevOps file and the learning_linux file by executing the following command:

`diff DevOps learning_linux`

The command outputs the parts of the two files that do not match:

![diff](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b7113bee-ec8f-4bbe-ab79-6c105f0491a8)


## 20. tar Command

We can use this command to archive multiple files into a TAR file. For instance if we want to archive the DevOps and the learning_linux files into an archive file named combined_archive, we simply execute the command below:

`tar -cvf combined_archive DevOps learning_linux`

The corresponding output should look like the image shown below:

![tar](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cde6d822-dab3-4e2a-8e84-1ba36a70c8fa)

## 21. chmod Command

We can use this command to modify a file or directory's read, write, execute permissions. For instance, to enable all users the ability to read, write and execute (rwxrwxrwx) the learning_linux file, we will enter the following command: 

`chmod 777 learning_linux`

As shown in the images of the command entry and output below, 777 is the numeric value for assigning rwxrwxrwx permissions. Subsequently, we use the `ls` command in combination with `-ltr` flag to show that the permissions on the learning_linux file have been successfully modified:

![chmod](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/15f863ea-73e0-439d-96be-1e1421e6c50c)

![chmod2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/215461b5-eb6c-42c6-97fd-c7855b890434)

## 22. chown Command

This command lets us change the ownership of a file or directory. For instance to give the user: Abdul, the ownership of the DevOps file, we simply enter the following command:

`chown Abdul DevOps`

By entering the `ls -ltr` command, can view the output shown in the image below. In the shown output, it is clear that the user: Abdul has been granted ownership of the DevOps file:

![chown](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4f0c7d11-35a9-4319-bd08-bf87240fc584)


## 23. jobs Command

This command is what we use to display all running processes along with their statuses. To implement this command and see how it works, we initiate a process or job by running the `sudo apt update` command to update the package lists on the system. Next we pause this process and send it to the background with ctrl+z. At this point, anybody approaching the terminal freshly is oblivious to that fact that there is a process/job running in the background. They can however do a check to ascertain if there is any running process with the `jobs` command. As shown in the image below, the command reveals the process in the background and its current status. Some flags can also be used with the command to extend functionality. 
`-l` lists process IDs along with their status information, `-p` lists status IDs only and `-n` lists jobs whose status has changed since the last notification.

![jobs 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f637e1f8-594b-42cc-8693-b6c7e1329566)


## 24. kill Command

We use this command to terminate an unresponsive program. However, to kill a program, it is essential to know its Process Identification Number (PID). You can detect this by entering the either the `ps ux` command or the `jobs -l` command. The kill command can also be used with signals. We can enter `kill SIGKILL PID` to force a program to stop and lose unsaved progress. However, entering `kill SIGTERM PID` stops a program but gives it time to save its progress. The system uses the SIGTERM signal by default if you enter the kill command without specifying a signal. The image below shows how we obtained the PID of the `apt upgrade` process and we used the command `kill 19682` to terminate this process:

![kill](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4bc2a518-1a52-4843-8e65-7f037bb1cb5f)


## 25. ping Command

This is the command that we enter to troubleshoot network connectivity issues and check whether a network or server is reachable. To use it, we simply type it along with the destination server hostname or IP address. So to check the availability of google.com we type the following command:

`ping google.com`

As shown in the image below, the command connects us to google.com and also shows useful information such as response time. We exit the process with ctrl+c: 

![ping](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/20949109-38fe-44c9-9377-3aa0df0f5e3a)


## 26. wget Command

This is used to download files from the internet. It can download files recursively and also in the background without hindering other running processes. We can implement it by entering the command followed by the download URL. For instance to download the lates version of WordPress, we execute the following command:

`wget https://wordpress.org/latest.zip`

In the output below, we can see that the system connects to the WordPress website via the specified URL and downloads the file 'latest.zip.1':

![wget](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/822f0585-2896-490a-a438-3cbd74188219)


## 27. uname Command

This prints detailed information about our Linux system and hardware. It can be used in combination with flags to extend functionality. Entering `uname -a` prints all system information, entering `uname -s` prints the kernel name and entering `uname -n` prints the node's hostname:

![uname](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/901885cf-9430-4e82-a80c-1f379e671c00)


## 28. top Command

This displays all the running processes and a dynamic real-time view of the current system. It sums up resource utilization from CPU to Memory usage. we can implement this by simply entering the command: 

`top`

The output is shown in the image below. ctrl+c is used to exit the command:

![top](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f3c2a115-8163-471e-889b-e4779f85e9f8)


## 29. history Command

This lists up to 500 previously executed commands allowing us to reuse them without necessarily re-entering them. It is implemented by simply entering the command:

`history`

The output below shows the previously entered commands:

![history](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c16b7b86-914d-406c-9ab2-3c632670379d)


## 30. man Command

This command displays the user manual for any command or utility that can be run on the terminal The user manual includes information like name, description and options for each command. The user manual also consists of 9 sections for each command. To implement, we shall use the command to show the user manual for the ls commannd, and also to show section 2 of the uname command by entering the following: 

`man ls`   `man 2 uname`

The first image below shows the output of man for the ls command. While the second image shows how specifying 2 in the command entry allowed the output to show us Section 2 of the uname command:

![man1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/845897d7-0df1-4251-a369-843a92429d29)

![man2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/06c9f5f1-7f84-4610-820e-86c972d6c8d2)


## 31. echo Command

We can use this to display a text or string on the CLI. It basically asks the system to return what we tell it to. We can use it by simply typing the command folled by the text or string we want displayed:

`echo My name is Abdul-Quadri Bello`

The image of the output shows the entered sentence displayed via the terminal:

![echo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8aafdc44-d59a-40b9-adfb-6d5b4621dba3)


## 32. zip, unzip Commands

The zip command can be used to compress a file into a zip file. This is useful for archiving and reducing disk usage. To utilize it we simply enter the command, followed by the name of the zip file to be created and subsequently the name of the fie to be compressed. So for instance to compress the DevOps file into a zip file called DevOps_archive, we execute the following command:

`zip DevOps_archive DevOps`

The output shows that the DevOps file was compressed by 28% and by running the `ls` command, we can see that the DevOps_archive.zip file has been created: 

![zip](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75488b88-83ca-4b01-99c8-25128661998b)

To unzip a zip file we will need to enter the unzip command followed by the name of the zip file to be unzipped. So to unzip DevOps_archive we will enter the following command:

`unzip DevOps_archive`

The output is as shown in the image below confirming the DevOPs file has been restored to it's original form:

![unzip](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6aae21ee-edc2-4164-b384-478cc7f3c39b)


## 33. hostname Command

This is used to know the system's hostname. To use the command we can simply enter:

`hostname` 

As shown the image below the comand can be entered without any flags or with some flags to extend functionality. `hostname -i` shows the system's IP address, `hostname -a display's the hostanme alias while `hostname -A` displays the system's Fully Qualified Domain Name (FQDN):

![hostname](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b99f5748-6e1a-467d-8a75-cf2924642d26)


## 34. useradd, userdel Commands

useradd is used to create a new user Account and only those with Admin rights can execute it. To use it, we type in the `useradd` command along with the name of the new user account and then we type the `passwd` command along with the name of the user account. So in our instance to create a user account named Abdul1, we enter the following commands:

`useradd Abdul1`

`passwd Abdul1`

The system prompts us to input the password once and then asks us to reconfirm it as shown in the output image below.

![useradd and userdel](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/700726dc-ffda-4755-8e38-aaf6dc91d903)

As can be seen in the above image, userdel is used to delete a user account. We simply enter the command along with the name of the user account. In our instance, we entered the following command to delete the user account named Abdul1:

`userdel Abdul1`


## 35. apt-get Command

This is a command line interface tool used for handling Advanced Package Tool Libraries in Linux. Its main task is to retrieve the information and packages from the authenticated sources for installation, upgrade, and removal of packages along with their dependencies. Using this command requires Admin privileges. The command can be used with different options as shown in the image below. The `apt-get update` command updates the package lists for available software packages from the configured repositories. `apt-get upgrade` installs the latest versions of the packages currently installed on the user’s system. And the `apt-get check` command  is used to update the package cache and check for broken dependencies.

![apt-get](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/94567c9a-c2c5-4ff5-83bb-8a00b36e6cec)


## 36. nano, vi, jed Commands

These commands can allow us to edit and manage files via a text editor. 

The nano command denotes keywords and can work with most languages. To use it, we will need to enter the command followed by the name of the file to be edited. For instance to open the learning_linux file with the nano command,  we simply enter the following:

`nano learning_linux`

The output is shown in the image below: 

![nano](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a1137e0b-7d99-4e2a-947a-fe56cd48da83)

The vi command works in two modes. `insert` is used to edit and create a text file while `command` performs operations such as saving, opening, copying and pasting a file. We will implement the command by using it on the learning_linux file:

`vi learning_linux`

The output is shown in the image below: 

![vi](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a9d46b41-508e-43d6-90e5-7f49ba6040e3)

The jed command has a dropdown menu interface that allows users to perform actions without entering keyboard combinationsor commands. We can use it by simply entering the command below:

`jed`

The output is shown in the image below: 

![jed](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9bf93330-5283-4721-b3d2-a7a023f7c017)


# 37. alias, unalias Commands

The alias command allows us to create a shortcut with the same functionality as a command, filename or text. It instructs the shell to replace one string with another. It is used by entering the coomand followed by the alias name, followed by the equal to (=) sign and follwed lastly by the command or filename to be replaced. To implement it, we will create an alias called 'listing' for the ls command by entering the following command:

`alias listing = ls`

The unalias command simply deletes an existing alias. It is used by entering the command, followed by the alias name. To unalias 'listing', we will enter the following command:

`unalias listing`

The image below shows the outputs for alias and unalais when implemented:

![alias and unalias](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fa80ac05-39a7-4dbb-b245-61725e01035d)

As seen in the image after the suceesful creation of the alias 'listing', it had the same use and functionality as the ls command. In the same vein after running the unalias command on 'listing', it lost the functionality accordingly and the system could no longer recognise it as an alias or as a command.


## 38. su Command

The switch user command allows us to run a program as a different user. It changes the administrative account during the current logged in session. It is a very useful command as it lets us switch to the Admin or Root user temporarily so that we can carry out administrative functions. Executing it will require us to enter the password of the user account we are switching to. It can be implemented by simply entering `su` which will inform the system to switch to the root user and send a prompt for password entry. It can aslo be implemented by entering su along with the username of the account we are switching to. For instance to switch from the root user to vboxuser, we enter the following command:

`su vboxuser`

And as shown in the image below, we switch back to the root user by entering `su` and inputting the appropriate password:

![su](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b3f087c-c7dc-424a-b551-a3e9be5dfac6)


## 39. htop Command

This is an interactive program that monitors system resources and server processes in real time. The command can be used by simply entering `htop`. We also note that this command can be used with flags or options that provide added functionality and improvements in comparison to the `top` command. 

`-d` shows the delay between updates in tenths of seconds. `-C` activates the monocrome mode and `-h` displays the help message and exit,

The output when we enter the `htop` command is shown below:

![htop](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/72a30431-a087-4ace-b1a3-6f6c8716aacb)


## 40. ps Command

The ps (process status) command produces a snapshot of all running processes in the system. To implement it we simply enter `ps` as shown in the image below. Executing this command without a flag or an option will list all the running processes in the shell along with their Process ID (PID), the Type of Terminal (TTY), the running time (TIME), and the command that launched the process (CMD). 

![ps](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/23c918f1-da01-4ee3-b033-28691a540cc1)

The ps command has some acceptable flags or options that can be used to extend its functionality:

When we enter `ps -T` it displays all processes associated with the current shell session:

![ps-t](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5bca676f-815a-41f4-b99a-b53882b1dad2)

When we enter `ps -u` vboxuser it lists all processes associated with the user 'vboxuser' specifically:

![ps-u](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e282b48b-302c-4718-8796-423e704f5163)

When we execute `ps -e` it shows all the running processes:

![ps-e](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9f931807-9734-49e7-a83a-4a03cb1bfdef)



## This brings us to the conclusion of this project. Thank you!





## DOCUMENTATION FOR SHELL SCRIPTING (HANDS ON) PROJECT

This project shows how to write shell scripts. It covers essential aspects of Shell Scripting from basic commands to advanced automation and it enables us learn how to streamline tasks and boost productivity.

### <br>Introduction to Shell Scripting and User Inputs<br/>

A [Shell Script](https://en.wikipedia.org/wiki/Shell_script) is a collection of commands and instructions that are executed sequentially in a shell. Essentially, shell scripting helps us automate repetitive tasks. For instance, rather than type the **`git clone`** command 1000 times to clone 1000 repositories, we can simply write a script that does this job and then we can call the script whenever we want to perform the task. Shell scripts are created by saving a series of commands in a text file with **.sh** extension. These scripts can then be executed directly from the command line or called from another script.

### <br>Shell Scripting Syntax Elements<br/>

1. **Variables:** Bash allows a user to define and work with variables. Variables can store various data types such as numbers strings and arrays. Values can be assigned to variables using the **=** operator. And a variable's value can be accessed by preceeding the variable name with a **$** sign.

For example, to assign value to a variable we enter the command below:

**`name="Moyosore"`**

To retrieve the value from a variable:

**`echo $name`**

![echo name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bf1b48ed-113a-4fc9-89a0-ffc77d51dfc0)

2. **Control Flow:** Bash enables us to contol the flow of execution in our scripts by implementing the use of control flow statements such as **if-else**, **else-if (elsif)**, **for loops**, **while loops** and **case** statements. All these statements allow us make decisions and execute different commands based on different conditions.

For Example, to use _**if-else**_ to execute a script based on conditions, we enter the following:

```
#!/bin/bash

# Example script to check if a number is positive, negative, or zero

read -p "Enter a number: " num

if [ $num -gt 0 ]; then
    echo "The number is positive."
elif [ $num -lt 0 ]; then
    echo "The number is negative."
else
    echo "The number is zero."
fi
```

As shown in the output image below, the above piece of code prompts us to type a number and prints a statement stating if the number is positive or negative:

![if-else statement](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/845b2327-6a62-4e08-8d4e-9802fe6742fa)

To illustrate another example, we can use a _**for loop**_ to iterate through a list with the following commnad:

```
#!/bin/bash

# Example script to print numbers from 1 to 5 using a for loop

for (( i=1; i<=5; i++ ))
do
    echo $i
done
```

The output of the command is shown below:

![For Loop](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/602a7fe1-8258-4c66-a4db-919b21b4c05b)

3. **Command Substitution:** This enables us to capture the output of a command and use it as a value within our script. we can use the bactick **`** or the **$()** syntax for command substitution.

For example, to use bactick **`** for command substitution, we enter the following:

```
current_date=`date +%Y-%m-%d`
```

To use **$()** syntax for command substitution, we enter the following:

```
current_date=$(date +%Y-%m-%d)
```

4. **Input and Output:** Bash allows users a number of ways to handle input and output. We can use the **`read`** command to accept user input and we can use the **`echo`** command to output text to the CLI. In addition, we can also redirect input and output using < (input from a file), > (output to a file), and | (use the output of one command as input to another.

An example of code accepting user input:

```
echo "Enter your name:"
read name
```

An example of code to output text to the terminal:

**`echo "Hello World"`**

An example showing how to output the result of a command into a file:

**`echo "hello world" > index.txt`**

Example showing how to pass the content of a file as input into a command:

**`grep "pattern" < input.txt`**


Example for passing the content of a command as input into another command:

**`echo "hello world" | grep "pattern"`**

5. **Functions:** Bash enables us to define and use functions so that we can group and execute related commands together. functions provide a way to modularize code and make it more reusable. Functions can be defined by the use of the function keyword or by declaring the function name followed by parentheses.

For example we define a function we call **greet** below:

```
#!/bin/bash

# Define a function to greet the user
greet() {
    echo "Hello, $1! Nice to meet you."
}

# Call the greet function and pass the name as an argument
greet "John"
```

As seen in the last line of the code above, we call the greet function and set the name "John" as the value and this results in the ouput shown below when we run this script:

![greet function](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/481f1320-2d3e-47c7-8376-cf10f416f6dc)

### We can write our first Shell Script for this project by following the steps below:

**Step 1:** To store all the scripts we will be creating in this project, we create a folder called _**shell-scripting**_ by entering the following command in our terminal:

**`mkdir shell-scripting`**

**Step 2:** We move inside the created folder with **`cd shell-scripting`**, and we create a file called _**user-input.sh**_ by entering the following command:

**`touch user-input.sh`**

**Step 3:** We copy the block of code below, then we paste inside the file.

```
#!/bin/bash

# Prompt the user for their name
echo "Enter your name:"
read name

# Display a greeting with the entered name
echo "Hello, $name! Nice to meet you."
```

**Step 4:** We save the file.

**Step 5:** We make the file executable by running the following command:

**`sudo chmod +x user-input.sh`**

**Step 6:** We run the script by using the following command:

**`./user.input.sh`**

As can be seen in the output image below, executing the script prompts for our name and after we enter it, it displays a greeting message with our name included as specified in the script.

![first script output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8a86437-d69f-4688-9728-6d131720cc23)

### <br>Directory Manipulation and Navigation<br/>

We will  begin by writing a shell script to focus on how to manipulate and traverse through directories. Here, we will be writing a shell script that will display the current directory, create a new directory called _**"my_directory"**_, switch to that directory, create two files inside it, list the files, move back one level up, remove the _**"my_directory"**_ and its contents and then finally list the files in the current directory again. We will implement thsi with the following steps:

**Step 1:** We create file that will be named _**navigating-linux-filesystem.sh**_ using the command below:

**`touch navigating-linux-filesystem.sh`**

**Step 2:** We open the file in our editor and then we copy and paste the following block of code into the file we created:

```
#!/bin/bash

# Display current directory
echo "Current directory: $PWD"

# Create a new directory
echo "Creating a new directory..."
mkdir my_directory
echo "New directory created."

# Change to the new directory
echo "Changing to the new directory..."
cd my_directory
echo "Current directory: $PWD"

# Create some files
echo "Creating files..."
touch file1.txt
touch file2.txt
echo "Files created."

# List the files in the current directory
echo "Files in the current directory:"
ls

# Move one level up
echo "Moving one level up..."
cd ..
echo "Current directory: $PWD"

# Remove the new directory and its contents
echo "Removing the new directory..."
rm -rf my_directory
echo "Directory removed."

# List the files in the current directory again
echo "Files in the current directory:"
ls
```

**Step 3:** We run the following command to make our file executable:

**`sudo chmod +x navigating-linux-filesystem.sh`**

**Step 4:** We run our script using the command below:

**`./navigating-linux-filesystem.sh`**

The output of executing our script is shown in the image below:

![navigating linux filesystem](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f2f59cb8-658b-444e-b041-604680a2b680)

### <br>File Operations and Sorting<br/>

In this part of the project, we will write a shell script that focuses on file operations and sorting. The script will create three files (*__file1.txt, file2.txt and file3.txt__*), it will display the files in their current order, sort them out alphabetically, save the sorted files in _**sorted_files.txt**_, display the sorted files, remove the original files, rename the sorted file to _**sorted_files_sorted_alphabetically.txt**_, and then display the content of the final sorted file. We shall proceed to implement this by using the following steps:

**Step 1:** On our terminal, we create a file that will be named _**sorting.sh**_ using the follwing command:

**`touch sorting.sh`**

**Step 2:** We open our file in an editor, then we copy and paste in the following block of code:

```
#!/bin/bash

# Create three files
echo "Creating files..."
echo "This is file3." > file3.txt
echo "This is file1." > file1.txt
echo "This is file2." > file2.txt
echo "Files created."

# Display the files in their current order
echo "Files in their current order:"
ls

# Sort the files alphabetically
echo "Sorting files alphabetically..."
ls | sort > sorted_files.txt
echo "Files sorted."

# Display the sorted files
echo "Sorted files:"
cat sorted_files.txt

# Remove the original files
echo "Removing original files..."
rm file1.txt file2.txt file3.txt
echo "Original files removed."

# Rename the sorted file to a more descriptive name
echo "Renaming sorted file..."
mv sorted_files.txt sorted_files_sorted_alphabetically.txt
echo "File renamed."

# Display the final sorted file
echo "Final sorted file:"
cat sorted_files_sorted_alphabetically.txt
```

**Step 3:** We run the following command to make our file executable:

**`sudo chmod +x sorting.sh`**

**Step 4:** We run our script using the command below:

**`./sorting.sh`**

The output of executing our script is shown in the image below:

![file operations and sorting script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6af5828d-e87a-4ca2-9f69-808f87fdc999)

### <br>Working with Numbers and Calculations<br/>

In this aspect of the project, we will write a shell script that defines two variables (_**num1 and num2**_) with numeric values, performs basic arithmetic operations (addition, subtraction, multiplication, division and modulus) and displays the results. Our shell script will also perform more complex calculations such as raising num1 to the power of 2 and calculating the square root of num2, and then it will display these results as well. We will implement this with the following steps: 

**Step 1:** On our terminal, we create a file that will be named _**calculations.sh**_ using the follwing command:

**`touch calculations.sh`**

**Step 2:** We open our file in an editor, then we copy and paste in the following block of code:

```
#!/bin/bash

# Define two variables with numeric values
num1=10
num2=5

# Perform basic arithmetic operations
sum=$((num1 + num2))
difference=$((num1 - num2))
product=$((num1 * num2))
quotient=$((num1 / num2))
remainder=$((num1 % num2))

# Display the results
echo "Number 1: $num1"
echo "Number 2: $num2"
echo "Sum: $sum"
echo "Difference: $difference"
echo "Product: $product"
echo "Quotient: $quotient"
echo "Remainder: $remainder"

# Perform some more complex calculations
power_of_2=$((num1 ** 2))
# square_root=$(awk "BEGIN{ sqrt=$num2; print sqrt }")
square_root=$(echo "$num2" | awk '{print sqrt($1)}')

# Display the results
echo "Number 1 raised to the power of 2: $power_of_2"
echo "Square root of number 2: $square_root"
```

**Step 3:** We run the following command to make our file executable:

**`sudo chmod +x calculations.sh`**

**Step 4:** We run our script using the command below:

**`./calculations.sh`**

The output of executing our script is shown in the image below:

![calculations script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8c0f11e4-5ee5-4650-9a97-2a2bed8cce6a)

### <br>File Backup and Timestamping<br/>

In this project section, we will be focusing on backing up files and using timestamps. We will write a shell script that defines the source directory and backup directory paths. Then the script will create a timestamp using the current date and time and create a backup directory with the timestamp appended to its name. Next, the script will copy all files from the source directory to the backup directory using the **`cp`** command with the **`-r`** option to copy recursively. Finally, the script will display a message to indicate the completion of the backup process and show the path of the backup directory with the timestamp. We shall proceed to implement this by using the following steps:

**Step 1:** On our terminal, we create a file that will be named _**backup.sh**_ using the follwing command:

**`touch backup.sh`**

**Step 2:** We open our file in an editor, then we copy and paste in the following block of code:

```
#!/bin/bash

# Define the source directory and backup directory
source_dir="/c/Users/USER/source_directory"
backup_dir="c/Users/USER/backup_directory"

# Create a timestamp with the current date and time
timestamp=$(date +"%Y%m%d%H%M%S")

# Create a backup directory with the timestamp
backup_dir_with_timestamp="$backup_dir/backup_$timestamp"

# Create the backup directory
mkdir -p "$backup_dir_with_timestamp"

# Copy all files from the source directory to the backup directory
cp -r "$source_dir"/* "$backup_dir_with_timestamp"

# Display a message indicating the backup process is complete
echo "Backup completed. Files copied to: $backup_dir_with_timestamp"
```

**Step 3:** We run the following command to make our file executable:

**`sudo chmod +x backup.sh`**

**Step 4:** We run our script using the command below:

**`./backup.sh`**

The output of executing our script is shown in the image below:

![file backup and timestamping](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/392d2082-2892-4656-a30f-59f6436041e6)

### <br>This brings us to the conclusion of this project. Thank you!<br/>


# DOCUMENTATION FOR GIT PROJECT

## This project shows the implementation of some core Git concepts and functionalities such as Initializing a Repository, Making Commits, Working with Branches, Collaboration and Remote Repositories and Tagging Track Changes

## Pre Installations and Dependencies

Please note that before executing this project we will need to download and install git on our computer sytem. Below is the image of the opened Git bash terminal which comes with the installation of Git:

![Git Bash terminal](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fbe0d0a9-d743-4c2d-858c-506a44cc9ae9)

## Initializing a Git Repository

To initialize our Git repository, we firstly need to create our working folder or directory which will be named DevOps. To do this, we will execute the following command in our opened Git Bash terminal:

`mkdir DevOps`

Next, we move into the created folder which will be our working directory with the following command:

`cd DevOps`

Then while inside the folder, we run the command below to initialize our repository:

`git init`

As shown in the output in the image below, we can see that our initialization process was successful.

![git init](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3b0787b0-5496-42c7-9eb0-b2ea2000abe3)


## Making your First Commit

It is important to note that we have 3 different environments in Git. We have the Working Files which is where we make edits to our files, the Staging area which is where our files are held before we commit. and lastly we have the Commit Environment.

Making a commit constitutes saving changes made to the files in our directory. These changes can be in form of adding, deleting or modifying files. It means a snapshot our repository is staken at the current state when we commit and saved in the .git folder in our working directory.

To make our first commit, we need to first of all create a text file that will be called index.txt inside our working directory. We do this by executing the following command: 

`touch index.txt`

The above command creates the text file, then as shown below, we enter a sentence in to our index file via a text editor and then we save our changes:

![index text](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fd33fff7-c606-404b-8508-584d40e5cb43)

The next step is for us to add our changes to the git staging area via the following command:

`git add .`

After executing the command above, we now have our files being held in the staging environment. From here we proceed to commit by entering the following command:

`git commit -m "initial commit"`

As shown in the output seen in the image below, the commit process was successful. Also we used the -m flag so that we can include a message ("initial commit") to provide some clarity and context about the commit.

![commit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2d4d8d59-07d7-4965-8d5b-b688acb1beb7)


## Working with Branches

Creating a git branch helps us to make a different copy of our source code i.e. a different copy of the main branch. By doing this, we can make edits and changes in our new copy as we please and these changes will remain independent of the original copy of our source code. This is an especially useful collaboration tool for programmers as different team members can create different branches to make changes or work on additional features and then merge all the changes to the main branch when all the work is done.

To make our first git branch which will be named my-new-branch, we execute the following command: 

`git checkout -b my-new-branch`

As shown in the output image below, the -b flag helps us to create and change into our new branch:

![mynewbranch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/973db837-9384-46c6-bb33-4ef7bbba4042)

To see all the branches in our local git repository, we enter the command below:

`git branch`

As we can see in the image, the command lists all the branches and identifies the branch we are currently in with an asterisk symbol.

![git branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ab00b3cf-0275-417a-a47f-80f0504a25d5)

To change back into an existing or old branch, for instance our 'main' branch, we simply execute the following command:

`git checkout main`

We can also merge a branch into another branch. To demonstrate this, we shall merge our branch titled 'my-new-branch' in to our 'main' branch. Please note that we made some changes to 'my-new-branch' by adding a line of text to the index.txt file as shown in the image below:

![index merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/063f54dd-0cf0-4a45-912f-da2720289280)

After adding the new line of text, we add our changed file to the staging environment with `git add .` and after this, we commit the change with `git commit -m "Added another line of text"`. 

To merge the edited index.txt file in 'my-new-branch' into the file in our unedited 'main' branch, we firstly switch to our 'main' branch with `git checkout main`  and then we execute the following command:

`git merge my-new-branch`

The image below shows the output of the merge process described above:

![git merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/21a81f5d-20ae-488e-bedb-20ffb0847358)

Now that we have merged the changes inside 'my-new-branch' into 'main' branch,  we can delete 'my-new-branch'. To execute this, we enter the following command:

`git branch -d my-new-branch`

Subsequently, when we enter `git branch` as shown in the image below, we can see that 'my-new-branch' has been deleted and no longer exists.

![git delete branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/de16ab50-a4b7-4d1f-8b9b-e7309511d25f)


## Collaboration and Remote Repositories

Github is a web based platform where git repositories are hosted. It enables collaboration between remote web or application development teams as they can work and make changes on different branches of the same code base in the same web based repository.

### To create a Github account, we carry out the following steps:

Step 1: We navigate to the [github.com](https://www.github.com) home page.

![github homepage](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/89ac088f-3f80-4e16-86a4-e43524290765)

Step 2: We enter our email, password and username.

![username](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ecf6271a-1e1c-4928-9ea4-5193e01cb3f4)

Step 3: Next, we take the verification quiz to confirm our identity.

![quiz](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/17b2e2f8-1b09-478f-a48a-3263486424c8)

Step 4: After completing the verification quiz, we click on the create account button at the bottom of the page.

![create account](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eee36f5d-44ff-48e4-a78d-ebf457915e37)

Step 5: After clicking on the create account button, a launch code was sent to our registered email address, so we proceed to enter the code in the provided textbox and then we click on the Continue button.

![launch code](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6d148a55-c164-4067-845b-626778823dbc)

Step 6: The next page asks us to choose how many team members we will be working with and to also choose if the user is a student or a teacher. So we choose our preferred options and click on the Continue button.

![team members](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/399aa079-c16f-4c84-84d6-b4759dfebd1a)

Step 7: The next page asks us to choose what specific github feature we are interested in using. So we select our preferred options and click on the Continue button.

![specific feature](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/71220a5d-91d1-4946-ad46-a63b88292cb0)

Step 8: Next, we are taken to the recommended plan page and here we will click on the "Continue for free" button and this creates our Github account.

![recommended plan](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/27f4a2f5-2a5a-43eb-a649-742154eaf379)

### Creating a New git Repository

To create a new repository, we carry out the following steps:

Step 1: We click on the plus sign (+) at the top right corner of our github account. A dropdown menu box appears and we select New repository.

![plus sign](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b8fdf5ae-6f7a-4396-9bbc-aadaea9bfd2d)

Step 2: We fill out the form by entering a unique and previously unused name for our repository. We enter a description and then we check the box to add a README.md file. And then we leave every other box or button in their default state.

![new repository](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/084013f0-a0ce-4320-adb7-0d569b76e835)

Step 3: We click the green 'Create repository' button at the bottom of the page to create our repository.

![create repository button](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2672317f-6cac-4411-ac17-319f512e2e10)

### Pushing your Local git Repository to your Remote github Repository

After we have created our github account and our github repository, we can now send a copy of our local repository on the computer to our repository in github. This can be executed by carrying out the following steps:

Step 1: We obtain the remote link for our repository by clicking the green 'Code' button and copying the https link.

![remote link](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6ed03f07-fc2e-464c-b252-2632fe333dce)

Step 2: We add our remote repository to our local repository using the following command:

`git remote add origin https://github.com/Quadri-Bello/DevOps_Repo.git`

![add repo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/436e5abd-3f2b-4d6c-8d88-20648f9dc1d5)

Step 3: We push the content in our local repository to our remote repository using the following command:

`git push origin main`

As shown in the output image below, using the command `git push origin` along with our branch name 'main' pushes all the files in our local repository to the main repository. 'origin' in the command makes reference to our remote repository url that was obtained in Step 1 and connected to our local repository in Step 2.

![git push](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3cd25d59-4236-4b30-b3b4-dcedf86590c6)

### Cloning your Remote git Repository

We can also make a local copy of our remote repository inside our local machine. We can do this by executing the following command: 

`git clone https://github.com/Quadri-Bello/DevOps_Repo.git`

As seen below, entering the `git clone` command along with the link to our repository downloads our local repository onto our local computer.

![git clone](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/66494618-55ee-4b8a-b575-3567d04329ac)


## Branch Management and Tagging

### Introduction to Markdown Syntax

The markdown syntax is a lightweight markup language that allows us to add formatting elements to our text without needing to use complex HTML or other formatting languages. It allows to format text using using elements like bold and italic styles, creating headers, lists, links etc. Markdown syntax can be used in creating files, documents, posts and web pages. To demonstrate how it works, we shall utilize some of the most commonly used markdown syntax elements below: 

1. Headings: To create headings we simply use the hash (#) symbol and a space before the text. The number of hash signs used reflects the level of the heading as shown below:

![headings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e5e7250f-6970-4a8f-bdd4-37bfbe2f9cad)

The output of the three heading styles can be seen below:

# Heading 1
## Heading 2
### Heading 3

2. Emphasis: We can use asterisk (*) or underscore (_) to place emphasis on text. As shown below, To put a text in Italics we can either put one asterisk before and after the text or we can put one underscore before and after the text. And to put a text in Bold, we can either put two asterisks before and after the text or we can put two underscores before and after the text.

![emphasis](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/18d7a633-ada1-404d-8844-f7fb63c63644)

The output of emphasis in Italics and in bold can be seen below:

*italic* or _italic_

**bold** or __bold__

3. Lists: The markdown syntax can support both ordered and unordered lists.

For ordered lists, we simply enter as in the example below:

![ordered list](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4490ab52-02ec-425c-b8fd-12d53a347aec)

As shown below: the output is exactly as entered:

1. First Item
2. Second Item
3. Third Item

For unordered lists we simply enter as in the example below: 

![unordered list](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/037f0c71-61e1-4abf-923d-b0796a903bee)

As shown below, the output is exactly as entered:

- Item 1
- Item 2
- Item 3

4. Links: To create a hyperlink, we enclose the link text with square [] brackets followed by the URL enclosed in parentheses (). For instance if we want to create the link that says 'visit google.com', we enter as shown below:

   ![link](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7130dfa9-424b-4407-b094-75a5aa6a16da)

   As shown in the output below, the link has been created:

   [visit google.com](https://www.google.com)

5. Images: The markdown syntax can also be used to display images. To display an image, we firstly enter an exclamation mark (!), followed by the image alt text enclosed in square brackets [], and then the image url enclosed in parentheses (). For instance to display the image of a shoe, we enter the following:

![shoe image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/26643c8b-a7af-4757-89dc-c06e792066f1)

As shown in the output below, the image of the shoe is displayed:

![shoe](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d81c35bb-aa1e-4e52-ba9e-bd1a0b192841)

6. Code: To display code or snippets of code in markdown syntax, we need to enter the code and enclose it in back ticks (`). For instance, to enter the code: console.log ('Welcome to Darey.io'), we simply input the following:

![code](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9446152f-ead8-4e1c-8a3a-16c40a0de860)

The output below shows how the code is formatted in markdown syntax:

`console.log ('Welcome to Darey.io')`

## This brings us to the conclusion of this project. Thank you!

How to use GIT on this couse

DONE **ONLY** AT FIRST TIME

1. Download and install git bash tool: <https://git-scm.com/>
2. It is intended that you already have signed up as a github user on it’s web page at <https://github.com>
3. Create access with SSH key by clicking on right upper corner on your user icon and go to “Settings”
4. Select SSH and GPG keys
5. Open git bash black command line window
6. Type command ssh-keygen and enter
7. Accept all defaults, type yes if needed, do not enter passwords, just pass them with enter
8. Go with cd command to .ssh configuration directory

cd

cd .ssh

Print public key to screen and copy paste it using mouse RIGHT button and always find copy paste from there:

cat id\_rsa.pub

1. Click “New SSH key”

Invent a name for the key like laptop and copy output from cat id\_rsa.pub

1. Click “Add SSH key”

Setup who are you for github using Git Bash

git config --global user.name "FIRST\_NAME LAST\_NAME"

git config --global user.email "MY\_NAME@example.com"

Go to your project folder with cd command to the folder which has pom.xml file.

Initialize repo

git init

Click the plus (+) sign. Insert a name of the repository in all lower case and no spaces. Preferably keep your repository public. Next github opens instruction page on what to do, that has same commands as here.

DONE ALL THE TIME WHEN WORKING WITH GIT (INCLUDING FIRST TIME). STUFF DONE DURING DAILY WORK IS IN BOLD – OTHER STUFF DONE ONLY FIRST TIME NOT BOLD

1. Add the files to new repository by using git command: **git add .**
2. Commit the files with comments: **git commit -m “First commit”**
3. Change your GitHub repository to your remote repository SELECT SSH NOT HTTPS, IF YOU ARE USING A KEY (Use URL given by github):

git branch -M main

1. Add origin where repository is located in the internet:

git remote add origin Your\_Github\_Repository\_URL

1. Push the changes to GitHub: **git push -u origin main**
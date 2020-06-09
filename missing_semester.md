#Basic Command#

- ls --help	and you'll see some basic commands that everyone might already know
- pwd	will print current directory
- ls	list the file/folders inside current directory
- ls -l	list files/folder with other information which includes, owner, read/write/execute permission allowed, size etc.
- ls -la	hidden file shown too
- rm	remove file
- rm -r	remove folder recursively
- rmdir remove a directory only if it is empty
- mv	rename file or move file
- cp	copy file
- cd -	moves back one directory and cd- again redo previous step. toggle between directories

# Combining commands for powerful usage. #

1. input and output in a terminal
    < means input and < means output

echo hello > hello.txt
writes output of the echo command inside hello.txt
Similarly, cat shows the content of a file. hence cat < hello.txt > heloo2.txt

we can use >> to append instead of write by replacing


**pipe character **

Take the output of program to left and make input to the program to the right

ls -l (we want only last line)
tail -n1 print last line
ls -l | tail -n1 (these two thing do not know each other)

curl --head --silent google.com | grep -i content-length | cut delimiter=' ' -f2

pipes are not only for text , could be for image, video.

# Finding Files #

- find . -name src -type d
	THis command searches for directory with name src recursively in the current directory

- find . -path '**/test/*.py -type -f
	This command searches for python file that is inside test directory which is inside any number of directory from current path.

- find . -mtime -1
	files modified today

Find not only can find stuff but also can do something to them.

- find . -name "*.tmp* -exec rm {} \
	THis command will find all file with temp extension and remove them

- find . -name "*.tmp" instead of this shorter one could be
	```
		fd "*.tmp"

	```

Note: DO man find for more info on the flags.


Alternative command: locate - it looks for file in the system. It is much faster than find command since it uses database index property.


# Finding content of file #

- GREP

	grep -R | <some text>	WIll go through every file and check for the text. *usecase:* you know you had some written some code but somewhere in filesystem you dont know. Then you can quickly search	
 
- ripgrep 

```
sudo snap install ripgrep --classic
```
Follow the link for distribution other than ubuntu.

[ripgrep](https://snapcraft.io/ripgrep)

rg (ripgrep) is same idea as grep but has extra things like color coding, its fast and more useful flag, has unicode support)
it also brings context- like  

-C 5 (5 line of context)
-u (for hidden file)
--files-without-match "~#\!" -t sh
--stats (extra info like no of matches)

# History #
method 1: licking up arrow , not that efficient

method 2: history 1 | grep <command>

method 3: ctrl + R >> type in portion of command >> ctrl + R to switch between commands >> enter when you get your required command

method 4: fzf lets us do fuzzy search, a type of interactive grep. Once you install fzf, it will automatically bind to your ctrl + r , history searching.


Fuzzy magic fzf

- installation
```
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```
Basics
- it binds to ctrl +r automatically
- it binds to ctrl +t , which is fuzzy search interface for a file

- cd ** and tab, it will trigger all the fzf and you can select the directory to go into.


## Directory Navigation ##
ls -R Recursively list files
tree - list the tree for direcory
broot - allows fuzzy search of files or folder

Installation:
1. Download the bin file from [here](https://dystroy.org/broot/documentation/installation/)
2. Copy it into /usr/bin
3. make it executable.
4. exit the terminal and re enter. Taaa Daa ready to go

# DATA WRANGLING #

COnverting data from one format to another. SOme examples:

whenever using pipe , taking output from one command to other as input is a kind of data wrangling.

Some of the fancier and useful data wrangling.

To do data wrangling you need source of data.

A simple example of data wrangling:

journalctl is a command for viewing logs collected by system.
We'll be viewing system log of my local system to extract users who have been entering through ssh or username of the server it doesssh to.

[Before that here's a small guide to doing ssh and scp (secure copy) without password.](#head1234)

1. Journalctl | grep ssh | grep "Disconnected from" | less > sshlog.log

	'less' created a pager.

2. using sed programming language to remove or replace the strings.
	```
	cat sshlog.log | sed 's/.*Disconnected from//'

	```
	what does .* means. Its a regex pattern. dot means single character and * means zero or any number of single characters.

3. now implementing regex to get user
	```
	cat ssh.log | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?/\2/'

	```
4. finally to get list of user sorted based on count of connection
	```
	cat ssh.log | sed -E 's/.*Disconnected from (invalid |authenticating )?(user )?(.* )?[0-9.]+ port [0-9]+( \[preauth\])?/\3/' | sort | uniq -c | sort -nk1,1 | tail -n20 | awk '{print $2}' | paste -sd,
	```

sed : a programming language mainly used in bash for pattern matching and replcing only.
sort : sorts the columnary data based on columns in ascending order
uniq -c : gives unique value with count of duplicated
tail: give last entry of a output
paste : writes all the output into single line using user input deliminitor
awk : very important in data wrangling.

Example of awk:
 - filtering user that has name starting with 'e' only
	```
	awk '$2 ~ /^e.*/ {print $0}
	```
 - number of user matching certain pattern
	```
	awk 'BEGIN{rows=0} $2 ~/^e.*/ {rows=rows+1}END {print rows}'
	```
 - counting the no of times user had been doing ssh
   we will be using 'bc' from berkely calculator
    ```
    awk '$1 != 1 {print $1}' | paste -sd+ | bc -l
    ```

## Data wrangling to make arguments ##

Sometimes you want to do data wrangling to find things to install or remove based on some longer list. The data wrangling we’ve talked about so far + xargs can be a powerful combo:
	```
	pip list | grep tensorflow-* | xargs pip uninstall
	```

<a name="head1234"></a>

1. copying files to remote server from local

```
scp –P <port> <file to be copied> username@remote_ip:<destination>

``` 
For folder add -r after scp.

2. copying files from remote to local

```
scp -P <port> username@remote_ip:<file to be copied> <your local destination>

```

Moving ahead is the way to do ssh and scp without password, if you have to constantly access any remote server.

1. enter following command that will generate two keys for you and save them in ~/.ssh/

```
ssh-keygen –t rsa

```
What ever the command ask you to enter, just enter "enter" blatantly. Default will do the work.

YOu will have two keys: id_rsa and id_rsa.pub. First one is private and later one public.

2. Now, copy the public key in remote server inside ~/.ssh/ as authorized_key2 file

```
scp ~/.ssh/id_rsa.pub remote_server@ip:/~/ssh/authorized_keys2

```

This will do the work. Now, when you do ssh or scp , password will not be asked.

Is this less secure than password:
Not at all. Your public key is a encrypted key which can only be decrypted by private keys. So, sharing your public key is secure. It means private key matches with only one public key and remote allows you to access it if the public key it has matches private key. 
It’s actually pretty similar in theory to using your password. If someone has knows your password, your security goes out of the window. If someone has your private key file, then security is lost to any computer that has the matching pubic key, but they need access to your computer to get it.

How to make it more secure:

While in first step you were creating keys, enter a passphrase it asks for. You can then combine sharing public key with passphrase concept. You will be asked the passphrase once while doing ssh or scp for the first time and then not required untill you are logged on in your computer. 

Notes: Tutotial for regex [link](https://regexone.com/)

## Command Line Environment ##




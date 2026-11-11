2026-11-08 23:41

To find out which directory you are currently in you can use `pwd` which means 'print working directory'.

To list the files in the current directory you use the `ls` command. `ls -l` lists them out in a list or table format which makes it easier to read. `ls -a` lists all the files including hidden files in the current directory. The hidden files in Linux have a dot before the file name. `ls -lh` lists out the files in the current directory with the size field being in more human readable format.

`cd ..` goes back one directory. **`cd ~`** changes the current working directory to the current user's home directory.

`touch` allows us to create files or change file timestamps. So if we want to create a file called `test.txt`  we can say `touch test.txt`. Now if we want to direct text into the file we can use `echo`. `echo` displays a line of text and we can direct that data into our `test.txt` file by saying `echo "nav loves golang" > test.txt`. `>` is the redirection operator in Linux, and since `echo` displays a line of text we are able to redirect where it displays that text by having the redirection operator point it to the file.

`cat` displays the contents of a file so using `cat test.txt` should print what we wrote earlier: `nav loves golang`.  This can also use the redirection operator to write to a file.

`rm` can remove files and directories. So if we say `rm test.txt` we remove that file. To create a directory we use `mkdir`. So `mkdir new_dir` makes that directory and to remove that directory we say `rm -R Test/`. We use `-R` to mention that it is recursively because that directory itself can contain files and sub-directories so it has to remove each one recursively. `rmdir` can also remove a directory but only if you do not have any files within that directory.

`mv` can move a file to a different directory and `cp` can copy a file to a different directory. `mv` can also rename files by using `mv test.txt newname.txt`

You can use vim and nano as text editors to edit files: `nano test.txt` or `vi test.txt`
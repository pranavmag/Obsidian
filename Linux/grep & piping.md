2026-11-11 22:57

`grep` is used to print lines that match patterns. Let's say that we want to find the word `nav` from the `test.txt` that we made. I wrote the line `nav was here.` in the file so if we use `grep "nav" test.txt` we will get that entire line printed.

`grep` is case sensitive but if we don't want that we can use the `-i` flag like `grep -i "Nav" test.txt` for it to work and print the line. Without that flag, it won't recognize `Nav` being in the file because we wrote it as lowercase in the file.

We can also use a pipe to direct the output of one command into another. We can do that with `grep` for example if we `cat` the contents of `test.txt` and then use a pipe that feeds the output of that to `grep`. We can use `cat test.txt | grep "nav"`. `cat test.txt` will print the contents of the file which will then be fed to `grep "nav` to search for the specific word and print out that line.

The difference between the redirection operator (>) and pipe (|) is that the redirection operator directs the output of one command to or from files rather than other commands. So using `cat test.txt > grep "nav"` will not work but you can do `cat test.txt > new_test.txt` which will write the contents of `test.txt` into `new_test.txt`.
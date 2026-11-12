2026-11-11 22:48

The current owner of a file is listed as the name when using the command `ls -alh` as well as group permissions if there are any.

If I wanted to change the ownership of a file to the root user I can use `sudo bash` to get into the root user and then use `chown`. `chown` changes file ownership and group, so you can use `chown root test.txt` which changes the ownership to the root user but the group ownership will still show under the old group. To change the group we can use `groups root` to see what group the root user belongs to and then use `chgrp root test.txt` which will now also change the group the file belongs to.


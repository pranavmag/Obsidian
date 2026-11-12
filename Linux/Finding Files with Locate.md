2026-11-11 23:21

Tags: 

`locate` is used to find files by name. If we use `locate test.txt` it will actually list all of the files that include `test.txt`. So when I ran that we got back our `test.txt` file location which we wanted but we also got a lot of different ones like `mod_get_test.txt` because it has `test.txt` within the name.

We can actually use `grep` to make this more efficient if we know at least one of the directories it would be in. Let's say the structure is `/home/nav/hi/hello/goodbye/found.txt` and I remember that the `found.txt` file is in the `goodbye` folder but I forget the rest of the structure. I can use `locate found.txt | grep "/goodbye"` and it will return that full file path that matches also having `goodbye` within the path.

I actually made that last exercise on my file system immediately to test it out and I was confused because it wasn't working when I used `locate found.txt | grep "/goodbye"` after just having created those folders and files. I looked it up and realized that `locate` doesn't search the live file system, it queries a pre-built index database that's refreshed by `updatedb` usually once a day via a cron job or systemd timer. So i had to run `sudo updatedb` and only then it worked.

For a live file system check I can use `find ~ -name "found.txt"`. `locate` is meant to be fast and returns all possible results it finds because it uses a substring match whereas `find` does an exact basename match so using find for `test.txt` only returns results where the actual file name is `test.txt` so we won't get the file path for `mod_get_test.txt` if we use `find`.

If we want `find` to have the same scope as `locate` we can use `sudo find / -name "*test.txt*"` so that any letters before or after `test.txt` are also included.

`locate --all -c test.txt` returns the number of occurrences of files that have `test.txt` in the name.
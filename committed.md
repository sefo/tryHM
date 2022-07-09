# Committed

This is a writeup for TryHackMe room `Committed` over at https://tryhackme.com/room/committed

## Prep

When the machine starts we get a file `commited.zip`. We can extract it in the same directory and open a terminal in there.

## Git

The following commands will be needed to solve the room

- git status
- git reflog
- git reset
- git diff

### Master

Running `ls` we see there are 2 files in the repository

- Readme.md
- main.py

### History

To get the commits history we can run `git reflog` command

```
...
4e16af9 (dbint) HEAD@{18}: commit: Reminder Added.
c56c470 HEAD@{19}: commit: Oops
3a8cc16 HEAD@{20}: commit: DB check
...
```

There are a bunch of commits in there but one in particular has our attention "oops"

### Checking a commit

To go to this particular commit we run `git reset HEAD@{19}`

With `git status` we see that the file `main.py` was modified.

To find out what was modified (see the difference between old and modified versions) `git diff main.py`

This gives detailed info on what was added (+) and removed (-) from the file:

```
diff --git a/main.py b/main.py
index 54d0271..0e1d395 100644
--- a/main.py
+++ b/main.py
@@ -4,7 +4,7 @@ def create_db():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root", # Username Goes Here
-    password="<REDACTED ANSWER IS HERE>" # Password Goes Here
+    password="" # Password Goes Here
     )
```

Thanks for the room!

regards ;)

# Mac OS

## Keyboard

- `Delete` is actually `Backspace` and for forward delete, you have to do `fn + Delete`.

## Mouse

- You have to enable `Right-click` for Apple mouses in `System preferences`. Otherwise you have to do two-fingers tap.
- Two-fingers tap on the pad is like right-click on mouse.

## Gestures on the mouse pad

- 3 fingers up -> Show all open apps (`Mission Control`)
- 3 fingers down -> Show all open windows of a particular app
- 3 fingers right/left -> Switch between desktops (a desktop can be created in `Mission control`)

## Screenshots

- Take a screenshot of the whole screen

```dos
Command + Shift + 3
```

- Take a screenshot of a section

```dos
Command + Shift + 4
then
Drag the cursor to specify the region you want to take a screenshot of
```

- Take a screenshot of an app window

```dos
Command + Shift + 4 + Space
```

## File and Folders Operations

`Finder` is equivalent to the explorer.

- To see the current path and status, go to View in menu bar and select `Show Path Bar` and `Show Status Bar`.

- Open a file

```dos
Command + O
```

- Create a folder

```dos
Tap with two fingers on the pad or right-click on the mouse and click on "New Folder"
```

- Rename a file/folder

```dos
Select the file + Enter
or
Right-click and click on "Rename"
```

- Get properties of a file/folder

```dos
Right-click and click on "Get Info"
```

- Create a new Finder tab

```dos
Command + T
```

- Copy and paste

```dos
Command + C
Command + v
```

- Cut and paste

```dos
Command + C
Command + Option + V
```

- Select all files

```dos
Command + A
```

- Delete a file/folder

```dos
Command + Delete -> Move it to trash
Command + Option + Delete -> Delete permanently
```

## App Operations

Spotlight is for search.

- Search an app

```dos
Command + Space
or
Click on the lens icon on the menu bar
```

- Change the current open application

```dos
Command + Tab
```

- Close the current application (By clicking on the red button, it won't be closed!)

```dos
Command + Q
or
Click on the name of the application on the menu bar and then click on Quit
or
Right-click on the app icon on the dock (If it's open, it has a black dot under it) and click Quit

```

- Force-quit an app

```dos
Command + Option + Esc
```

- Installing an app

  1. Download the dmg file.
  2. Double-click on the dmg file.
  3. Drag the app icon to the Applications folder.

- Uninstalling an app
  1. Go to Applications folder in the Finder.
  2. Click on the application.
  3. Press `Command + Delete`.
     or, you just drag it to the trash.

# Terminal

- Command line interface (CLI) or terminal emulators (terminal for short) can be used to run a language such as python or a shell language.
- Shell is a program that wraps the OS kernel and accepts user commands via a terminal.

  - `Bash` is the shell language that runs in most terminals of Linux systems.
  - `PowerShell` is the shell language that runs in most terminals of Windows systems.
  - Another shell language is `zsh`.
  - Windows command prompt (cmd) uses `DOS batch language` which is far behind.

| Linux CLI Commands | Windows CLI Commands | Description                                       |
| ------------------ | -------------------- | ------------------------------------------------- |
| ls                 | dir                  | List the directory (folder) system                |
| cd pathname        | cd pathname          | Change directory (folder) in the file system      |
|                    | cd \\                | Move to the root folder of the file system        |
| cd ..              | cd ..                | Move one level up (one folder) in the file system |
| cp                 | copy                 | Copy a file to another folder                     |
| mv                 | move                 | Move a file to another folder                     |
| mkdir              | mkdir or md          | Creates a new directory (folder)                  |
| rmdir              | rmdir or rd          | Remove a directory (folder)                       |
| cat                | type                 | Prints the contents of a text file in the console |
| touch file_name    |                      | Creates a single file                             |
| clear              | cls                  | Clears the CLI window                             |
| ps                 | ps                   | Displays the currently-running processes          |
| exit               | exit                 | Closes the CLI window                             |

## Bash

- To change the command prompt showing on the left or the output results each command returns, you can configure bash by modifying `.bashrc` and/or `.bash_profile` files.
- `.` in bash means current directory.
- In linux, `/` is the root directory which has a `home` directory inside it and every user has a directory with their name inside `home`.
- `~` represents home directory.
- To confirm that we are in bash:

### Working with directories

```bash
echo $0
```

- To show the present working directory:

```bash
pwd
```

- To list all the directories and files:

```bash
ls
```

- To see more info and also hidden files/directories (identified by a dot before the filename):

```bash
ls -al
```

- To make a directory:

```bash
mkdir projects
```

- To change directory:

```bash
cd projects
```

- To remove a directory:

```bash
cd ..
rmdir projects
```

### Working with files

- To tell what is a file:

```bash
file file.mp3
```

It returns `Directory`, if it is not a file.

- To create an empty file:

```bash
touch file1.md
```

- To delete a file:

```bash
rm file1.md
```

- To copy a file:

```bash
cp file2.md file2-copy.md
```

- To rename a file:

```bash
mv file2-copy.md file3.md
```

- To open VI text editor:

```bash
vi file2.md
```

- To insert content in a file in VI text editor:
  - press `i` to enable insert mode
  - press `esc` when you are done
  - type `:wq` and hit `enter` (`:q` just quit without writing/saving)

### Other

- To clear everything:

```bash
clear
```

- To check the internet speed:

```bash
ping googl.com
ctrl + C
```

- To displays the currently-running processes:

```bash
ps
```

- To set an environment variable:

```bash
export PORT=5000
```

In Windows, it is `set PORT=5000`.

### Scripting in bash

- Create a file called `whatever`.
- Every file that starts with `#!/bin/bash` is a bash script:

```bash
#!/bin/bash
clear
echo "Hello, world."
```

- Run this file by:

```bash
./whatever
```

But it won't run! Because each file or directory has a set of permissions.
Usually we see `rwx`, for read, write, and execute.
If we run `ls al` it shows the permissions, like `-rwxrw-r--`:

1.  `-` for a file and `d` for a directory.
2.  permissions for the `owner`.
3.  permissions for the `group owning a file`.
4.  permissions for `others`.

- Enabling the `execute` permission for file:

```bash
chmod +x whatever
or
chmod 700 whatever  -> -rwx------
chmod 744 whatever  -> -rwxr--r--
chmod 644 whatever  -> -rw-r--r--
chmod 755 whatever  -> -rwxr-xr-x
```

# IIITD Google Drive (and Google Photos) Data Transfer to Personal Google Drive (and Google Photos)

## 1\. Introduction: Why Do This?

Your IIITD Google Workspace account storage limit will be reduced to 5GB after graduation. This guide provides a comprehensive, step-by-step method to transfer all your data (including "Shared with me" files) from your IIITD account to a personal Google account using a powerful tool called `rclone`.

**This guide is split into 5 parts:**

1.  **Install rclone:** Get the tool on your computer.
2.  **Configure rclone:** Connect your two Google Drive & Photos accounts.
3.  **Copy Your Data:** Run the commands to transfer everything.
4.  **Verify Your Transfer:** Manually check that your data is safe.
5.  **Delete IIITD Data:** Securely wipe your IIITD account data.

**Please note this guide is only tested/verified on Mac**
-----

## 2\. Part 1: Install rclone

You must first install `rclone` on your computer. Open your Terminal (on macOS/Linux) or Command Prompt (on Windows).

### On Windows

The easiest method is to download the pre-compiled binary.

1.  Go to the [rclone downloads page](https://rclone.org/downloads/).
2.  Download the `.zip` file for "Intel/AMD - 64 Bit".
3.  Unzip the file (e.g., to `C:\rclone`).
4.  You will need to run commands from this folder. For ease of use, you should add this folder to your system's `PATH`.
      * Search for "Environment Variables" in the Start Menu and open "Edit the system environment variables".
      * Click "Environment Variables...".
      * Under "System variables", find and select the `Path` variable, then click "Edit...".
      * Click "New" and paste the path to your rclone folder (e.g., `C:\rclone`).
      * Click OK on all windows.
5.  Restart your Command Prompt or PowerShell and type `rclone --version` to confirm it's installed.

### On macOS

The easiest method is to use [Homebrew](https://brew.sh/).

```bash
# Install Homebrew if you don't have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install rclone
brew install rclone
```

### On Linux

The official install script is recommended:

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

-----

## 3\. Part 2: Configure rclone (The Remotes)

Now you will connect `rclone` to your four accounts. This process will open your web browser to ask for permission.

Open your Terminal or Command Prompt and type:

```bash
rclone config
```

Follow this interactive setup four times, once for each remote.

### Remote 1: `gdrive_iiitd` (Your IIITD Google Drive)

1.  `No remotes found - make a new one`
2.  `n)` (New remote)
3.  `name>` **`gdrive_iiitd`**
4.  `Storage>` Find "Google Drive" and enter its number (or just type `drive`).
5.  `client_id>` Press **Enter** (leave blank).
6.  `client_secret>` Press **Enter** (leave blank).
7.  `scope>` **`1`** (Full access).
8.  `root_folder_id>` Press **Enter** (leave blank).
9.  `service_account_file>` Press **Enter** (leave blank).
10. `Edit advanced config?>` **`n`** (No).
11. `Use auto config?>` **`y`** (Yes).
12. Your browser will open. **Log in to your @iiitd.ac.in account.** Grant permission.
13. `Configure this as a team drive?>` **`n`** (No).
14. `y/e/d>` **`y`** (Yes, this is OK).

### Remote 2: `gdrive_personal` (Your Personal Google Drive)

1.  At the main menu, select `n)` (New remote).
2.  `name>` **`gdrive_personal`**
3.  Follow the **exact same steps** as above (select `drive`, scope `1`, etc.).
4.  **Crucial Difference:** When the browser opens, **log in to your personal @gmail.com account.**

### Remote 3: `gphotos_iiitd` (Your IIITD Google Photos)

1.  At the main menu, select `n)` (New remote).
2.  `name>` **`gphotos_iiitd`**
3.  `Storage>` Find "Google Photos" and enter its number (or just type `photos`).
4.  `client_id>` Press **Enter**.
5.  `client_secret>` Press **Enter**.
6.  `scope>` **`2`** (Read-only access). This is the source, so read-only is safest.
7.  `Edit advanced config?>` **`n`** (No).
8.  `Use auto config?>` **`y`** (Yes).
9.  Browser opens: **Log in to your @iiitd.ac.in account.** Grant permission.
10. `y/e/d>` **`y`** (Yes, this is OK).

### Remote 4: `gphotos_personal` (Your Personal Google Photos)

1.  At the main menu, select `n)` (New remote).
2.  `name>` **`gphotos_personal`**
3.  `Storage>` Find "Google Photos" and enter its number (or just type `photos`).
4.  `client_id>` Press **Enter**.
5.  `client_secret>` Press **Enter**.
6.  `scope>` **`3`** (Append-only access). This allows rclone to add photos but not delete anything from your personal account.
7.  `Edit advanced config?>` **`n`** (No).
8.  `Use auto config?>` **`y`** (Yes).
9.  Browser opens: **Log in to your personal @gmail.com account.** Grant permission.
10. `y/e/d>` **`y`** (Yes, this is OK).

-----

Once all four are set up, type **`q`** to quit the configuration.

-----

## 4\. Part 3: Copy Your Data

These commands are **server-side**, meaning data moves from Google to Google directly, not through your computer. It's very fast.

  * `-P` shows a live progress bar.
  * `--drive-server-side-across-configs` enables the fast server-side transfer.
  * `--log-file=copy.log` saves a detailed log, which is useful for checking errors.

### Option A: Copy **ONLY** "My Drive" (Your Owned Files)

This copies everything in your IIITD "My Drive" to a new folder named `IIITD_Drive_Backup` in your personal drive.

```bash
rclone copy gdrive_iiitd: gdrive_personal:IIITD_Drive_Backup -P \
--drive-server-side-across-configs --log-file=drive_copy.log
```

### Option B: Copy "My Drive" **AND** "Shared with me"

This is a two-step process that ensures you get everything, including files shared with you, and handles potential duplicate filenames (e.g., `IMG_1234.jpg`).

**Step 1: Copy "My Drive" (Same as Option A)**

```bash
rclone copy gdrive_iiitd: gdrive_personal:IIITD_Drive_Backup -P \
--drive-server-side-across-configs --log-file=drive_copy_1.log
```

**Step 2: Copy "Shared with me"**
This copies all shared files/folders into `IIITD_Drive_Backup/SharedWithMe`. If it finds files with duplicate names, it moves the conflicts into `IIITD_Drive_Backup/SharedDuplicates` so no data is lost.

```bash
rclone copy gdrive_iiitd: --drive-shared-with-me gdrive_personal:IIITD_Drive_Backup/SharedWithMe -P \
--drive-server-side-across-configs \
--backup-dir=gdrive_personal:IIITD_Drive_Backup/SharedDuplicates \
--log-file=drive_copy_2.log
```

### Google Photos Transfer

This command will copy all photos and videos from your IIITD Photos library to your personal Photos library.

**Note:** The Google Photos API has limitations. This will copy all media, but it **will not recreate your albums.** The photos will appear in your personal library sorted by date.

```bash
rclone copy gphotos_iiitd: gphotos_personal: -P --log-file=photos_copy.log
```

-----

## 5\. Part 4: VERIFY YOUR TRANSFER (CRITICAL STEP)

**DO NOT PROCEED TO PART 5 UNTIL YOU HAVE DONE THIS.**

1.  Open your personal Google Drive (`drive.google.com`).
2.  Find the `IIITD_Drive_Backup` folder.
3.  Browse through it. Spot-check important folders and files to ensure they are present and open correctly.
4.  If you ran the "Shared with me" copy, also check the `SharedWithMe` and `SharedDuplicates` folders.
5.  Open your personal Google Photos (`photos.google.com`).
6.  Scroll through your library to confirm your photos and videos from IIITD have appeared.

Only when you are **100% certain** all your data is safe should you even consider deleting the originals.

-----

## 6\. Part 5: Delete Data from IIITD Account

**🛑 WARNING: THIS IS PERMANENT. THERE IS NO UNDO. NO ONE CAN RECOVER THIS DATA FOR YOU. PROCEED WITH EXTREME CAUTION. 🛑**

Always run the commands with `--dry-run` first. This will simulate the deletion and show you a list of what *would* be deleted, without actually deleting anything.

### Google Photos Deletion

rclone **cannot delete** from Google Photos. You must do this manually.

1.  Go to [photos.google.com](https://photos.google.com/) and log in with your **IIITD account**.
2.  Select your photos (you can click the first, hold `Shift`, and scroll down to select many).
3.  Click the trash icon to move them to the bin.
4.  Go to the Bin and empty it.

### Google Drive Deletion

#### Option A: Delete **EVERYTHING** from your IIITD Drive

This command will permanently delete all files and folders in your `gdrive_iiitd` "My Drive".

**Step 1 (Safety Check):**

```bash
rclone delete gdrive_iiitd: --dry-run
```

*(Review the output carefully.)*

**Step 2 (Permanent Deletion):**

```bash
rclone purge gdrive_iiitd:
```

*(`purge` is more aggressive than `delete` and will remove the entire remote's contents.)*

-----

#### Option B: Delete **ONLY PRIVATE FILES** (Safer)

This command is for users who have shared important files with others and *do not* want to delete those shared files. It will only delete files that **you own** and that are **not shared with a link** (i.e., `visibility='private'`).

**Note for Windows Users:** This command uses `()` which may not work in the default Command Prompt. Please use **Git Bash** or **Windows Subsystem for Linux (WSL)** to run it.

**Step 1 (Safety Check):**

```bash
rclone delete --tpslimit 8 --dry-run --files-from <(rclone backend dump gdrive_iiitd: --drive-qs "'me' in owners and trashed=false and visibility='private'")
```

*(Review the log. This will show you all the "private" files it intends to delete. We add `--tpslimit 8` to avoid rate-limit errors from the scan.)*

**Step 2 (Permanent Deletion):**

```bash
rclone delete --tpslimit 8 --files-from <(rclone backend dump gdrive_iiitd: --drive-qs "'me' in owners and trashed=false and visibility='private'")
```

**Note for Winows Users:** Option 2, run without the () parameter in two seperate commands.
```bash
rclone backend dump gdrive_iiitd: --drive-qs "'me' in owners and trashed=false and visibility='private'" > files.txt
rclone delete --tpslimit 8 --files-from files.txt
```


-----

*This readme was prepared by Shivam Dwivedi.*

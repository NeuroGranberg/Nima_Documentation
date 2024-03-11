
## General information

to be written..


## Initial structure

`iAID` is structured into a main directory and a couple of sub-directories in FG. Inside the `iAID` folder, there's a `derivatives` folder that splits into two parts:

 - `bids-dcm`: Presumably BIDS-formatted DICOM files which was empty.
 - `original-data-zipped`: Compressed original data.
 
The zipped files follow a numeric organization, each encompassing a distinct range of data entries with sizes ranging between 22GB and 69GB, totaling approximately 610GB.

```bash
📦iAID
┃┗📂derivatives
┃  ┣ 📂bids-dcm
┃  ┣ 📂orignal-data-zipped
┃  ┃ ┣ 📜MS-IAID_00001-01000_(570).zip   69GB  
┃  ┃ ┣ 📜MS-IAID_01001-02000_(501).zip   54GB  
┃  ┃ ┣ 📜MS-IAID_02001-03000_(498).zip   56GB  
┃  ┃ ┣ 📜MS-IAID_03001-04000_(538).zip   59GB  
┃  ┃ ┣ 📜MS-IAID_04001-05000_(528).zip   35GB  
┃  ┃ ┣ 📜MS-IAID_05001-06000_(185).zip   28GB  
┃  ┃ ┣ 📜MS-IAID_06001-07000_(301).zip   37GB  
┃  ┃ ┣ 📜MS-IAID_07001-08000_(541).zip   58GB  
┃  ┃ ┣ 📜MS-IAID_08001-09000_(623).zip   59GB  
┃  ┃ ┣ 📜MS-IAID_09001-10000_(431).zip   22GB  
┃  ┃ ┣ 📜MS-IAID_10001-11000_(742).zip   42GB  
┃  ┃ ┣ 📜MS-IAID_11001-12000_(727).zip   33GB  
┃  ┃ ┣ 📜MS-IAID_12001-13000_(648).zip   34GB  
┃  ┃ ┗ 📜MS-IAID_13001-13782_(429).zip   36GB 
```

## Copying iAID to $HOME

I copied `iAID` folder to my home directory on the FG server over SSH. Here's how I did it:

```bash
cd /mnt/raid/MS/projects/
tar -cf - iAID | (cd ~; tar -xf - )
```

I chose `tar` for its efficiency with large datasets. This approach compresses `iAID` and then decompresses it directly into my home directory in a single operation, bypassing the slower process of transferring files one by one.[^1] 

[^1]: Approaches For [Copying Large Data](https://neurogranberg.github.io/Nima_Documentation/workspace/copying_large_data/)


!!! WARNING "Problem with `rsync` on FG server!"

    I've also tried to use `rsync` and for some reason which I could not find, was not stable on the server. The log was:

    ```Bash
    iAID/derivatives/orignal-data-zipped/MS-IAID_00001-01000_(570).zip
    rsync error: received SIGINT, SIGTERM, or SIGHUP (code 20) at io.c(503) [generator=3.2.3]
    rsync error: received SIGINT, SIGTERM, or SIGHUP (code 20) at rsync.c(703) [sender=3.2.3]
    rsync: [receiver] write error: Broken pipe (32)
    ```

    SIGINT, SIGTERM, or SIGHUP (code 20): This error suggests rsync received an interrupt signal. I was not manually cancelling the process, so it might be a system or network issue causing the interruption.
    Write error or Broken pipe (32): This usually happens when the connection between the source and destination is lost during the transfer. Maybe due to permission!

## Decompressing (Unzipping)

When attempting to decompress zip file, the standard `zip` function was unsuccessful[^2]. Upon investigating the issue, I discovered the following:

[^2]: Step-wise Approach to [Fixing Corrupted ZIP Archives](https://neurogranberg.github.io/Nima_Documentation/workspace/fixing_corrupted_archives/)

```Bash

nima@fractalgrande:$ zip -T -v MS-IAID_00001-01000_\(570\).zip 

Could not find:
  MS-IAID_00001-01000_(570).z01

Hit c      (change path to where this split file is)
    q      (abort archive - quit)
 or ENTER  (try reading this split again): 

Could not find or open
  MS-IAID_00001-01000_(570).z01

Hit c      (change path to where this split file is)
    q      (abort archive - quit)
 or ENTER  (try reading this split again): q 

zip error: Interrupted (MS-IAID_00001-01000_(570).zip)
nima@fractalgrande:~/iAID/derivatives/orignal-data-zipped$ zip -T -v MS-IAID_00001-01000_\(570\).zip 


Could not find:
  MS-IAID_00001-01000_(570).z01

Hit c      (change path to where this split file is)
    q      (abort archive - quit)
 or ENTER  (try reading this split again):
```

The error message I encountered indicated that the ZIP file `MS-IAID_00001-01000_(570).zip` appears to be a segment of a multipart (split) ZIP archive, with the tool failing to locate the subsequent segment, named `MS-IAID_00001-01000_(570).z01`. Typically, in multipart ZIP archives, data is distributed across several files, and it is generally required to have all parts available to successfully open or extract the contents of the archive. However, this ZIP file is not actually part of a split archive but is incorrectly being recognized as one.

Initially, I thought the problem might be related to the handling of very large ZIP files by the `zip` utility, possibly due to its limitations or specific behaviors with large files. To test the integrity of the ZIP file and explore alternative solutions, I experimented with using the `unzip` and `7z` tools.

 ```Bash

 $ unzip -t MS-IAID_00001-01000_\(570\).zip

   Archive:  MS-IAID_00001-01000_(570).zip
   warning [MS-IAID_00001-01000_(570).zip]:  68616287551 extra bytes at beginning or within zipfile 
   (attempting to process anyway)
   error [MS-IAID_00001-01000_(570).zip]:  start of central directory not found;
   zipfile corrupt.
   
 ```

This suggests that the ZIP archive could be corrupted, possibly due to problems arising during the file's transfer or creation process.

The warnings included:

- Extra bytes at the beginning or within the zipfile
- The central directory's start not found

These are typical signs of corruption or incomplete files. The option to ensure the ZIP files were downloaded or transferred in binary mode was not viablen tho. (See the best practices on binary transferration[^3])

[^3]: Best Practice In [Downloading Large Data](https://neurogranberg.github.io/Nima_Documentation/workspace/download_large_data/)

### Understanding the Problem

ZIP files contain a central directory that lists all the files in the archive, along with metadata about those files (such as file names, sizes, and compression methods). This central directory is essential for tools to locate and extract files from the ZIP archive.

When a ZIP file is corrupted, it can mean:

- The central directory is damaged or missing.
- The file entries are incomplete, incorrect, or out of order.
- There are extra bytes within the ZIP file that confuse ZIP tools because they don't follow the expected ZIP file structure.

`zip` has two option to deal with damaged files. `-F` will try and make the zip file usable without needing to resort to any drastic measures. It works under the assumption that the majority of the zip file is intact and tries to repair issues like missing or corrupted sections of the file. However, it might not be successful if the zip file is significantly corrupted.

When I used `-F` option the proccess failed:

```Bash
 $ zip -F MS-IAID_03001-04000_\(538\).zip --out MS-IAID_03001-04000_\(538\)_fixed.zip

 Found end record (EOCDR) - says expect single disk archive
 Scanning for entries...
 copying: MS-IAID_03001-04000_(538)/2B333630716C5272325244666971304A4D70586256673D3D/  (0 bytes)
 copying: MS-IAID_03001-04000_(538)/2B333630716C5272325244666971304A4D70586256673D3D/4E4653615133416133304244765057763937497173513D3D/  (0 bytes)
 copying: MS-IAID_03001-04000_(538)/2B333630716C5272325244666971304A4D70586256673D3D/4E4653615133416133304244765057763937497173513D3D/10/  (0 bytes)
 copying: MS-IAID_03001-04000_(538)/2B333630716C5272325244666971304A4D70586256673D3D/4E4653615133416133304244765057763937497173513D3D/10/1.3.12.2.1107.5.2.30.59058.2017042608235624357703003_927.dcm 
```
The initial error messages indicated that the ZIP tool was unable to find the central directory, suggesting it was either misplaced within the file due to corruption or extra bytes were presenting as if part of the file was missing or additional to the expected content.

- **Found end record (EOCDR)**: This indicates that `zip` found the End of Central Directory Record, which is a good sign. It means that there was enough intact structure for the repair tool to identify the end of the ZIP archive, suggesting that at least some of the central directory could be located or inferred.
- **Scanning for entries...**: This part of the output shows that `zip` is scanning the ZIP file for individual file entries, attempting to piece together the archive's structure from whatever intact data can be found.
- **Copying entries**: The detailed lines about copying entries show that `zip` is extracting what it identifies as valid file entries from the corrupted archive into a new, repaired archive file. The mention of `0 bytes` for some entries suggests that it found directory entries or possibly corrupted file entries that didn't contain actual data.

The second option which is `zip -FF` command, can be helpful here because `zip` actually found `End of Central Directory Record (EOCDR)`. It will attempt to reconstruct the central directory by scanning the entire ZIP file for markers that indicate the start of file entries. It then tries to rebuild the central directory based on this scan. This is a more aggressive repair strategy than `zip -F` though.

### Fixing the packages with `-FF` option

I checked one of the curropted files with `-FF` option and since it worked I wrote a Bash script to do it for all the zip files:

`fix_zip_files.sh`[[source]]():

```Bash
#!/bin/bash

# Navigate to the directory containing the ZIP files
cd /home/nima/iAID/derivatives/orignal-data-zipped

# Loop through all ZIP files in the current directory
for zip_file in *.zip; do
  # Define the name of the fixed ZIP file
  fixed_zip="${zip_file%.zip}_fixed.zip"
  
  # Define the error log file name
  error_log="${zip_file%.zip}_error.log"
  
  # Attempt to repair the ZIP file. Redirect only stderr to the log file, discard stdout.
  zip -FF "$zip_file" --out "$fixed_zip" 2> "$error_log"
  
  # Optional: Check if the repair was successful
  if [ ! -s "$fixed_zip" ]; then
    echo "Repair failed or file was not recoverable for $zip_file, check $error_log for details."
    # Remove the fixed file if it's empty indicating repair failure
    rm -f "$fixed_zip"
  else
    echo "Repair process completed for $zip_file. Output in $fixed_zip and errors (if any) in $error_log."
    # Since repair was successful and fixed file is not empty, delete the original ZIP file
    rm -f "$zip_file"
  fi
done

```

This script is a Bash shell script designed to automate the process of attempting to repair corrupted ZIP files within a specified directory. Here's a breakdown of its functionality:

1. **Setting the Working Directory**: The script starts by changing the current directory to `/home/nima/iAID/derivatives/orignal-data-zipped` using `cd`. This is where it expects to find the ZIP files to be repaired.

2. **Processing Each ZIP File**: It uses a `for` loop to iterate over all `.zip` files in the current directory. For each file, it performs several actions as detailed below.

3. **Defining Filenames for Outputs**: For each ZIP file encountered, it constructs two new filenames based on the original ZIP file's name:
    - A new filename for the repaired ZIP file, appending `_fixed.zip` to the original name before the `.zip` extension.
    - An error log filename, appending `_error.log` to the original name, to store any error messages that occur during the repair process.

4. **Attempting to Repair the ZIP File**: The script uses the `zip -FF` command to attempt a more aggressive repair of each ZIP file. It directs the output of this command (`stdout`) to be discarded and only saves the error messages (`stderr`) to the previously defined error log file. This command creates a new ZIP file (`$fixed_zip`) in an attempt to repair the corrupted one.

5. **Checking the Repair Outcome**: After attempting a repair, the script checks if the repaired ZIP file (`$fixed_zip`) is non-empty. If it's empty (`! -s "$fixed_zip"`), it indicates the repair failed or the file was not recoverable. It then prints a message to the console and removes the empty fixed file.

6. **Handling Successful Repairs**: If the repaired file is not empty, indicating a successful repair or partial recovery, it prints a success message. It also deletes the original ZIP file to clean up the directory, assuming the fixed file is now the preferred version to keep.

7. **Loop Continuation**: This process repeats for every ZIP file in the directory, attempting to repair each in turn and handling the outcome accordingly.

!!! DANGER "Failed attempts on two of the ZIP files"
    The `-FF` option managed to successfully repair 12 out of the 14 ZIP files, suggesting a more severe issue with the remaining two. Further investigation revealed that their End of Central Directory Record (EOCDR) could not be located by the software, showimg that the corruption in these two files was too extensive to be fixed by the `-FF` option.


### Discarding CDR

After successfully fixing most of the corrupted ZIP files in the `iAID` dataset using the `-FF` option, I encountered issues with two particular archives: `MS-IAID_11001-12000_(727).zip` and `MS-IAID_13001-13782_(429).zip`.

```bash
  📜MS-IAID_11001-12000_(727).zip   33GB  
  📜MS-IAID_13001-13782_(429).zip   36GB 
```

The CDR is a crucial component of a ZIP archive, acting as a directory for all the files it contains. It indexes the files, storing information about their names, sizes, and locations within the archive. However, when the CDR is corrupted or missing, conventional extraction tools struggle to locate and extract the files based on this index. This led me to explore the method of discarding the Central Directory Record (CDR) to recover the remaining files. Despite the sizeable nature of these files, each being over 30GB, I opted for the `7z x` command as a last resort. 

```Bash
7z x derivatives/orignal-data-zipped/MS-IAID_11001-12000_\(727\).zip -oRecoveredFiles
```

and

```Bash
7z x derivatives/orignal-data-zipped/MS-IAID_13001-13782_\(429\).zip -oRecoveredFiles
```

This strategy proved effective to a large extent, managing to extract almost all contents except for the very last file in each archive. The error messages pointed to an "Unexpected end of archive," indicating that the files might have been truncated or the archive structure was severely compromised towards the end:


For `MS-IAID_11001-12000_(727).zip`

```Bash
Scanning the drive for archives:
1 file, 34891640832 bytes (33 GiB)

Extracting archive: derivatives/orignal-data-zipped/MS-IAID_11001-12000_(727).zip
                
ERRORS:
Unexpected end of archive

--
Path = derivatives/orignal-data-zipped/MS-IAID_11001-12000_(727).zip
Type = zip
ERRORS:
Unexpected end of archive
Physical Size = 34891704733

ERROR: Data Error : MS-IAID_11001-12000_(727)/6F785157526D634A664B32682B51596E4377517532413D3D/37514D72366239712F5534796944736F5358734F74774455364A6367436A4C48/2/1.3.12.2.1107.5.2.18.41252.2018050819370829555987017_358.dcm
                                                                         
Sub items Errors: 1
Archives with Errors: 1
Open Errors: 1
Sub items Errors: 1

```

And for `MS-IAID_13001-13782_(429).zip`:

```Bash
Scanning the drive for archives:
1 file, 37760675840 bytes (36 GiB)

Extracting archive: derivatives/orignal-data-zipped/MS-IAID_13001-13782_(429).zip
                
ERRORS:
Unexpected end of archive

--
Path = derivatives/orignal-data-zipped/MS-IAID_13001-13782_(429).zip
Type = zip
ERRORS:
Unexpected end of archive
Physical Size = 37760913262

ERROR: Data Error : MS-IAID_13001-13782_(429)/656B4B41394F37555678727576336D637845514431513D3D/37514D72366239712F5537586C74444B656D56322B774455364A6367436A4C48/8/1.3.12.2.1107.5.2.43.67059.2018112413585850919854295_838.dcm
                                                                         
Sub items Errors: 1
Archives with Errors: 1
Open Errors: 1
Sub items Errors: 1
```

Given the situation, discarding the CDR proved to be a viable path forward. By ignoring the central directory, I focused on extracting files based on their local headers, which contain essential metadata for extraction and it worked almost perfectly. Then, the fixed zip files were extracted as normal.


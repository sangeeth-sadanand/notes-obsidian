---
up:
  - "[[003_skills/data-engineering/03 Storage layer|03 Storage layer]]"
down:
prev:
topic: false
question: What are the commands used for interaction with HDFS?
---
# What are the commands used for interaction with HDFS?


> [!Summary] Summary
>  We can used the 'hdfs dfs' or' hadoop fs' for interaction with file system common commands used are:
> - `-put` : to put files from local to hdfs
> - `-get` : to get files from hdfs to local.
> - `-ls` : to list files and directory
> - `-mkdir`: to make directory
> - `-rm` : to remove file
> - `-mv` : to move /rename file
> - `-up` : copy. a file
> - `-cat`: outputs entire file
> - `-chmod`: change permission
> - `-chown`: change ownership


- `hadoop fs` list all command used in hadoop
    
- `hdfs dfs` is same as `hadoop fs`
    
- In most of the command it is communicated with name node. for some command like cat, head it will also communicate with the data node.
    - `-appendToFile <localsrc> ... <dst>`: command is used to append data from one or more local files to a file in the Hadoop Distributed File System (HDFS)
	- `-cat [-ignoreCrc] <src> ...`: to view the contents
	- `-checksum [-v] <src> ...`: In the context of Hadoop, a **checksum** is a value calculated from a data set and used to verify the integrity of data.
	- `-chgrp [-R] GROUP PATH...`/: The `chgrp` command in Linux is used to change the group ownership of files and directories.
	- `-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...`| change the permission of a file or directory
	- `-chown [-R] [OWNER][:[GROUP]] PATH...`: Change the ownership of a file or directory
	- `-copyFromLocal [-f] [-p] [-l] [-d] [-t <thread count>] <localsrc> ... <dst>`: copy a file local to HDFS
	- `-copyToLocal [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>`: copy file HDFS to local directory
	- `-count [-q] [-h] [-v] [-t [<storage type>]] [-u] [-x] [-e] <path> ...`: To count the number of files and directories in a specified path
	- `-cp [-f] [-p | -p[topax]] [-d] <src> ... <dst>`: copy file from src to dest in HDFS
	- `-createSnapshot <snapshotDir> [<snapshotName>]`: In Hadoop, creating a snapshot allows you to capture the state of a file system at a specific point in time.
	- `-deleteSnapshot <snapshotDir> <snapshotName>`: Delete a snapshot.
	- `-df [-h] [<path> ...]`: the `df` (disk free) command is used to check the disk space usage of the HDFS
	- `-du [-s] [-h] [-v] [-x] <path> ...`: is used to estimate the space used by files and directories in HDFS
	- `-expunge [-immediate] [-fs <path>]`: the `expunge` command is used to permanently remove files and directories from the trash.
	- `-find <path> ... <expression> ...`: The `find` command in Hadoop allows you to search for files and directories in HDFS based on various criteria.
	- `-get [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>`: copy file HDFS to local directory
	- `-getfacl [-R] <path>`: The `getfacl` command in Linux is used to get the Access Control Lists (ACLs) of files and directories.
	- `-getfattr [-R] {-n name | -d} [-e en] <path>`: The `getfattr` command in Linux is used to retrieve the extended attributes of files and directories.
	- `-getmerge [-nl] [-skip-empty-file] <src> <localdst>`: The `getmerge` command in Hadoop is used to merge multiple files in the Hadoop Distributed File System (HDFS) into a single local file.
	- `-head <file>`: Print the head data.
	- `-help [cmd ...]`: prints help for the command
	- `-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [-e] [<path> ...]`: list files and directories
		![[001_Meta/media/hdfs_ls_cmd.svg]]
	- `-mkdir [-p] <path> ...`: make directory
	- `-moveFromLocal <localsrc> ... <dst>`: move the file from local to HDFS
	- `-moveToLocal <src> <localdst>`: move the file from HDFS to local
	- `-mv <src> ... <dst>` move file from src to dest within HDFS
	- `-put [-f] [-p] [-l] [-d] <localsrc> ... <dst>`: copy file from local directory to HDFS
	- `-renameSnapshot <snapshotDir> <oldName> <newName>`: Rename the snapshot
	- `-rm [-f] [-r|-R] [-skipTrash] [-safely] <src> ...`: Remove the file
	- `-rmdir [--ignore-fail-on-non-empty] <dir> ...`: remove directory
	- `-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]`: The `setfacl` command in Linux is used to set Access Control Lists (ACLs) for files and directories.
	- `-setfattr {-n name [-v value] | -x name} <path>`: The `setfattr` command in Linux is used to set extended attributes of files and directories.
	- `-setrep [-R] [-w] <rep> <path> ...`: The `setrep` is used to change the replication factor of files in the Hadoop Distributed File System (HDFS).
	- `-stat [format] <path> ...`: is used to display detailed information about files and directories.
	- `-tail [-f] [-s <sleep interval>] <file>`: print tail data.

## 6. Administrative (dfsadmin)

These commands are usually reserved for cluster admins and start with `hdfs dfsadmin`.

- **`-report`**: Shows a health summary of the cluster (Live/Dead nodes).
    
- **`-safemode <enter/leave/get>`**: Controls "Safe Mode" where the NameNode is read-only during startup or maintenance.
    
- **`-refreshNodes`**: Tells the NameNode to re-read the include/exclude host files (used when adding or removing servers).
    

> **Pro-Tip:** If you ever forget a flag, you can always run `hdfs dfs -usage <command>` or `hdfs dfs -help` for a full list of options available in your specific Hadoop version.
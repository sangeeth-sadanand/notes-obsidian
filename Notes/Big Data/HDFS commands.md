# HDFS commands

## Directory Operations
- `-ls <path>` → List files/directories
- `-lsr <path>` → Recursive list
- `-mkdir <path>` → Create directory
- `-mkdir -p <path>` → Create parent directories as needed
- `-rmdir <path>` → Remove empty directory

## File Upload & Download
- `-put <local> <hdfs>` → Upload local file
- `-copyFromLocal <local> <hdfs>` → Copy local file to HDFS
- `-moveFromLocal <local> <hdfs>` → Move local file to HDFS (deletes local copy)
- `-get <hdfs> <local>` → Download file from HDFS
- `-copyToLocal <hdfs> <local>` → Copy file from HDFS to local
- `-getmerge <hdfs_dir> <local_file>` → Merge multiple HDFS files into one local file

## Viewing & Reading
- `-cat <path>` → Show file contents
- `-tail <path>` → Show last KB of file
- `-text <path>` → Display file in text format (works for SequenceFiles, Avro, etc.)
- `-checksum <path>` → Show checksum of file

## File Removal & Modification
- `-rm <path>` → Delete file
- `-rm -r <path>` → Recursive delete
- `-mv <src> <dst>` → Move/rename file
- `-cp <src> <dst>` → Copy file within HDFS

## File Information
- `-du <path>` → Disk usage of files
- `-dus <path>` → Summary disk usage
- `-count <path>` → Count directories, files, and size
- `-stat <path>` → Show file/directory status (like `stat` in Linux)

## Permissions & Ownership
- `-chmod <mode> <path>` → Change permissions
- `-chown <owner> <path>` → Change owner
- `-chgrp <group> <path>` → Change group

## Advanced / Special
- `-expunge` → Empty the HDFS trash
- `-setrep <n> <path>` → Set replication factor
- `-touchz <path>` → Create empty file
- `-appendToFile <local> <hdfs>` → Append local file contents to HDFS file
- `-truncate <path>` → Truncate file to given length
- `-find <path> -name <pattern>` → Search files in HDFS

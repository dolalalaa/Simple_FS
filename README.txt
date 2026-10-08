CSE321 Lab Term Project - Summer 2026
SimpleFS: Implementation of a Simple File System in C

Group Number: 01

Group Members:
1. Ashfia Newaz-23301259
2. Nabila Tabassum Khan-23301257
3. Nishat Anjuman Dola-23301206


#COMPILATION COMMANDS:

gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder


#EXECUTION EXAMPLES:

1) Create an empty SimpleFS image:
   ./simplefs_builder --image disk.img

2) Add files to the image:
   ./simplefs_adder --input disk.img --file test1.txt
   ./simplefs_adder --input disk.img --file test2.txt
   ./simplefs_adder --input disk.img --file test3.txt

3) Verify the image using a hex viewer:
   xxd disk.img
   xxd -s 4096 -l 16 disk.img     (inode bitmap)
   xxd -s 8192 -l 16 disk.img     (data bitmap)
   xxd -s 16384 -l 128 disk.img   (root directory: . and .. entries)

BRIEF DESCRIPTION OF IMPLEMENTATION:
SimpleFS is a small educational file system created with two C programs. Both programs use one binary disk image named disk.img.

-simplefs_builder.c:

simplefs_builder.c creates and formats a new SimpleFS disk image. The image is 262,144 bytes in size which is equal to 256 KiB. It is divided into 64 blocks and each block is 4096 bytes.

The program prepares the main file system areas:

Block 0: Stores the superblock. It contains the magic number, block size, total number of blocks, inode count and the locations of the file system regions.
Block 1: Stores the inode bitmap. Inode 1 is marked as allocated because it belongs to the root directory.
Block 2: Stores the data bitmap. Data Block 4 is marked as allocated because it stores the root directory data.
Block 3: Stores the inode table. The root inode is created as a directory with a size of 128 bytes. Its first direct pointer refers to Block 4.
Block 4: Stores the root directory entries. The entries . and .. are created and both point to inode 1.


-simplefs_adder.c:

simplefs_adder.c adds one regular file from the current working directory to an existing SimpleFS disk image.

The program first opens the disk image and checks the superblock magic number. If the magic number is incorrect the image is rejected.

It checks the file before adding it:

Files larger than 12,288 bytes are rejected because SimpleFS supports only three direct blocks. Each block is 4096 bytes.
File names longer than 58 characters are rejected.
A file is rejected if another file with the same name already exists in the root directory.
The program also checks whether free inodes, data blocks and directory entries are available.

The program uses a first-fit allocation method. It scans the inode bitmap to find the first free inode. It then scans the data bitmap to find enough free data blocks for the file.

After finding free blocks the program copies the file contents into them. If the final block is not completely filled the unused space is filled with zeros.

The program creates a new inode for the file. The inode stores the file type, link count, file size and direct block pointers.

A new directory entry is added to the root directory. This entry connects the file name with its inode number. The root directory size is increased by 64 bytes for each new entry.

Finally the program updates the inode bitmap and the data bitmap. It writes both updated bitmaps back to the disk image.

Error Handling:

Both programs use safe error handling. They can handle missing files, missing disk images, invalid arguments, files that are too large, duplicate file names and full inode or data block areas. They also prevent crashes when the root directory has no space for another entry.


CONTRIBUTION OF EACH GROUP MEMBER

Ashfia Newaz(Part 1):
- Implemented all of simplefs_builder.c (Builder TODO 1-6): superblock initialization, inode bitmap and data bitmap         
  allocation for the root, root inode initialization and the "." / ".." root directory entries.

Nabila Tabassum Khan(Part 2):
- Implemented the helper/lookup functions in simplefs_adder.c (Adder TODO 1-4): find_free_inode, find_free_data_block,
  filename_exists and find_free_directory_entry.

Nishat Anjuman Dola(Part 3):
- Implemented the main file-adding logic in simplefs_adder.c (Adder TODO 5-11): required-block calculation, data block
  allocation, copying file contents into blocks, new inode initialization, inode bitmap marking, directory entry creation
  and root inode size update.


KNOWN LIMITATIONS / PROBLEMS

- As per the project specification, SimpleFS does not support subdirectories, file deletion, file renaming, indirect block
  pointers, permissions, journaling, checksums or caching.
- The root directory has a maximum capacity of 31 regular files,limited by the 32 available inodes (one reserved for root).
- No separate file-read command is implemented, as it was not required by the specification.
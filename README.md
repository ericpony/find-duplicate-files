# find-duplicate-files
A Perl script that locates duplicate files (ie. files with identical contents but at different inodes) with O(n) time and O(1) space overhead. It uses GNU sort that takes O(nlogn) time and O(n) space to sort the entries. This script manages to minimize the overheads needed to find duplicates. It should be among the most efficient scripts available on GitHub for the same purpose. 

File duplicatiion is checked using [String::CRC32](http://search.cpan.org/~soenke/String-CRC32-1.5/CRC32.pod) and [Digest::MD5](http://search.cpan.org/~gaas/Digest-MD5-2.54/MD5.pm). You can use one or both of them as checksum.

Usage
-----
    Usage: /data/slugfly/ericpony/BIN/find-duplicate [OPTIONS] {DIR,FILE}*
    Options:
        --checksum=TYPE  Specify the type of checksum. TYPE can be {crc, md5, both}. Default is crc.
        --digest         Don't find duplicates. Instead, print the checksums of the input files.
        --parallel=N     Change the number of sorts run concurrently to N.
        --verbose        Print debug messages.
        --help           Print this message.

Given at least one DIR/FILE, the script checks the files contains in DIR or listed in FILE. A FILE is expected to be the output of `ls -1 DIR`. For example, to find duplicate files in directory `images` using MD5 as checksum, you can write

    find-duplicate --checksum=md5 images

This script also receives input from STDIN. However, the file listing should be digested before being used for comparison. For example, the above command is equivalent to

    /bin/ls -1 images/* | find-duplicate --checksum=md5 --digest | find-duplicate

Note the use of globbing here. If you use `images` instead of `images/*`, the script may not obtain the correct paths of the listed files. Globbing however can cause an "Argument list too long" error when the number of files in `images` is huge, in which case one should use 

    find-duplicate --checksum=md5 --digest images | find-duplicate

In practice, computing checksum is usually the most time-consuming stage in finding duplicate files. Hence, it is sometimes preferrable to separate this stage from the process of finding duplicate files. For example, consider the situation that you want to compare files in folder A against those in folders B<sub>1</sub>, ..., B<sub>n</sub> and you don't want to compare files in B<sub>1</sub>, ..., B<sub>n</sub> with each other. In this case, saving the checksums in intermediate files and comparing the folders using these files will be far more efficient than comparing the folders in pairs directly.

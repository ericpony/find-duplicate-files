    # find-duplicate-files
A Perl script that locates duplicate files (ie. files with identical contents but at different inodes) with O(n) time and O(1) space overhead. It uses GNU sort that takes O(nlgn) time and O(n) space to sort the entries. This script manages to minimize the overheads needed to find duplicates. It should be among the most efficient scripts available on GitHub for the same purpose. 

File duplicatiion is checked using [String::CRC32](http://search.cpan.org/~soenke/String-CRC32-1.5/CRC32.pod) and [Digest::MD5](http://search.cpan.org/~gaas/Digest-MD5-2.54/MD5.pm). You can use one or both of them as checksum.

Usage
-----
    Usage: /data/slugfly/ericpony/BIN/find-duplicate [OPTIONS] {DIR,FILE}*
    Options:
      --checksum=TYPE  Specify the type(crc, md5, both) of checksum. Default is crc.
      --digest         Do not find duplicates. Instead, print checksums of the files.
      --pipe           Read checksums from STDIN.
      --parallel=N     Change the number of sorts run concurrently to N.
      --verbose        Print debug messages.
      --help           Print this message.

Given at least one DIR/FILE, the script checks the files contains in DIR or listed in FILE (which is expected to be the output of something like `ls -1 DIR` or `find DIR`). Note that the script ignores sub-directories and empty files. To check files recursively, you have to feed the filenames to the script using `find`. For example, to find duplicate files in directory `images` using MD5 as checksum, you can write

    find-duplicate --checksum=md5 images

You can also do this in two stages (see discussions below):

    find-duplicate --checksum=md5 --digest images | find-duplicate --pipe

Note that the above lines doesn't check files in the sub-directories of `image`. To check files recursively, use

    find images | find-duplicate --checksum=md5 

Discussions
-------
In practice, computing checksum is usually the most time-consuming stage in finding duplicate files. Hence, it is sometimes preferrable to separate this stage from the process of finding duplicate files. For example, consider the situation that you want to compare files in folder A against those in folders B<sub>1</sub>, ..., B<sub>n</sub> and you don't want to compare files in B<sub>1</sub>, ..., B<sub>n</sub> with each other. In this case, saving the checksums in intermediate files and comparing the folders using these files will be far more efficient than comparing the folders in pairs directly.

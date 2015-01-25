#Find duplicate files

A Perl script that locates duplicate files (ie. files with identical contents but at different inodes) with O(n) time and O(1) space overhead. It uses GNU sort that takes O(nlgn) time and O(n) space to sort the entries. This script manages to minimize the overheads needed to find duplicates. It should be among the most efficient scripts available on GitHub for the same purpose. 

File duplicatiion is checked using [String::CRC32](http://search.cpan.org/~soenke/String-CRC32-1.5/CRC32.pod) and [Digest::MD5](http://search.cpan.org/~gaas/Digest-MD5-2.54/MD5.pm). You can use one or both of them as checksum.

Usage
-----
    Usage: /data/slugfly/ericpony/BIN/find-duplicate [OPTIONS] {DIR,FILE}*
    Options:
      --checksum=TYPE  Specify the type(crc, md5, both) of checksum. Default is crc.
      --digest         Don't find duplicates. Print file checksums instead.
      --pipe           Don't compute checksums. Read checksums from STDIN or FILE.
      --parallel=N     Change the number of sorts run concurrently to N.
      --verbose        Print debug messages.
      --help           Print this message.

Given at least one DIR/FILE, the script checks the files contains in DIR or listed in FILE (which is expected to be the output of something like `ls -1 DIR` or `find DIR`). Note that the script ignores sub-directories and empty files. To check files recursively, you have to feed the filenames to the script using `find`. For example, to find duplicate files in directory `images` using MD5 as checksum, you can write

    find-duplicate --checksum=md5 images

which is equivalent to (see discussion below):

    find-duplicate --checksum=md5 --digest images | find-duplicate --pipe

Note that the above usages don't check files in the sub-directories. To check files recursively, use

    find images | find-duplicate --checksum=md5 

Discussion
-------
In practice, computing checksum is usually the most time-consuming stage in finding duplicate files. Hence, it is sometimes preferrable to separate this stage from the process of finding duplicate files. For example, consider the situation that you want to compare files in folder A against those in folders B<sub>1</sub>, ..., B<sub>n</sub> and you don't want to compare files in B<sub>1</sub>, ..., B<sub>n</sub> with each other. In this case, comparing the folders using pre-computed checksums is far more efficient than comparing the folders in pairs directly. Another advantage of two-stage processing is that you can compute the checksums in parallel:

    find-duplicate --digest A > A.checksum
    for i in $(seq 1 $n); do # spawn $n processes
        (   find-duplicate --digest B$i > B$i.checksum
            (   flock 200
                cat A.checksum B$i.checksum | find-duplicate --pipe && rm B$i.checksum
                rm B$i.checksum
            ) 200>.lock 
        ) &
    done
    rm .lock

For another example, suppose you want to compare files in folders B<sub>1</sub>, ..., B<sub>n</sub>. Instead of invoking `find-duplicate B1 ... Bn` directly, you can parallelize the computation of checksums as follows:

    for i in $(seq 1 $n); do # spawn $n processes
        find-duplicate --digest B$i > B$i.checksum &
    done
    wait 
    cat *.checksum | find-duplicate --pipe && *.checksum

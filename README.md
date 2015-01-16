# find-duplicate-files
A Perl script that locates duplicate files (ie. files with identical contents but at different inodes) in O(n) time and O(1) space. Should be more efficient than most of the scripts publicly available on GitHub for this purpose.

Usage
-----
    find-duplicate DIR1 DIR2

When there are multiple duplicates in the same directory, the script associates the files in `DIR2` to the first duplicate file found in `DIR1`. Duplication is checked using [String::CRC32](http://search.cpan.org/~soenke/String-CRC32-1.5/CRC32.pod) and [Digest::MD5](http://search.cpan.org/~gaas/Digest-MD5-2.54/MD5.pm).

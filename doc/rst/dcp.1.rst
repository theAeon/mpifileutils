dcp
===

SYNOPSIS
--------

**dcp [OPTION] SRC DEST**

DESCRIPTION
-----------

Parallel MPI application to recursively copy files and directories.

dcp is a file copy tool in the spirit of :manpage:`cp(1)` that evenly
distributes the work of scanning the directory tree, and copying file
data across a large cluster without any centralized state.  It is
designed for copying files that are located on a distributed parallel
file system, and it splits large file copies across multiple processes.

OPTIONS
-------
.. option:: --bufsize SIZE

   Set the I/O buffer to be SIZE bytes.  Units like "MB" and "GB" may
   immediately follow the number without spaces (e.g. 8MB). The default
   bufsize is 4MB.

.. option:: --chunksize SIZE

   Multiple processes copy a large file in parallel by dividing it into chunks.
   Set chunk to be at minimum SIZE bytes.  Units like "MB" and
   "GB" can immediately follow the number without spaces (e.g. 64MB).
   The default chunksize is 4MB.

.. option:: --xattrs WHICH

    Copy extended attributes ("xattrs") from source files to target files.
    WHICH determines which xattrs are copied.  Options are to copy no xattrs,
    all xattrs, xattrs not excluded by /etc/xattr.conf, or all xattrs except
    those which have special meaning to Lustre.  Certain xattrs control Lustre
    features on a file-by-file basis, such as how the file data is distributed
    across Lustre servers.  Values must be in {none, all, libattr, non-lustre}.
    The default is non-lustre.

.. option:: --daos-api API

   Specify the DAOS API to be used. By default, the API is automatically
   determined based on the container type, where POSIX containers use the
   DFS API, and all other containers use the DAOS object API.
   Values must be in {DFS, DAOS}.

.. option:: --daos-preserve FILENAME 

   Option to turn on metadata preservation in DAOS. This should
   be used in the case that data is being moved to/from DAOS. For instance,
   if data is being moved from DAOS to Lustre, then the preserve option can
   be used to write the DAOS container metadata to an HDF5 file. That way,
   when data is moved back from Lustre to DAOS the container properties can
   be preserved. A filename to write the metadata to must be specified.

.. option:: -i, --input FILE

   Read source list from FILE. FILE must be generated by another tool
   from the mpiFileUtils suite.

.. option:: -L, --dereference

   Dereference symbolic links and copy the target file or directory
   that each symbolic link refers to.

.. option:: -P, --no-dereference

   Do not follow symbolic links in source paths. Effectviely allows
   symbolic links to be copied when the link target is not valid
   or there is not permission to read the link's target.

.. option:: -p, --preserve

   Preserve permissions, group, and timestamps.

.. option:: -s, --direct

   Use O_DIRECT to avoid caching file data.

.. option:: -S, --sparse

   Create sparse files when possible.

.. option:: --progress N

   Print progress message to stdout approximately every N seconds.
   The number of seconds must be a non-negative integer.
   A value of 0 disables progress messages.

.. option:: -v, --verbose

   Run in verbose mode.

.. option:: -q, --quiet

   Run tool silently. No output is printed.

.. option:: -h, --help

   Print a brief message listing the :manpage:`dcp(1)` options and usage.

RESTRICTIONS
------------

If a long-running copy is interrupted, one should delete the partial
copy and run dcp again from the beginning. One may use drm to quickly
remove a partial copy of a large directory tree.

To ensure the copy is successful, one should run dcmp after dcp
completes to verify the copy, especially if dcp was not run with the -s
option.

EXAMPLES
--------

1. To copy dir1 as dir2:

``mpirun -np 128 dcp /source/dir1 /dest/dir2``

2. To copy contents of dir1 into dir2:

``mkdir /dest/dir2 mpirun -np 128 dcp /source/dir1/\* /dest/dir2``

3. To copy while preserving permissions, group, timestamps, and
   attributes:

``mpirun -np 128 dcp -p /source/dir1/ /dest/dir2``

KNOWN BUGS
----------

Using the -S option for sparse files does not work yet at LLNL. If you
try to use it then dcp will default to a normal copy.

The maximum supported file name length for any file transferred is
approximately 4068 characters. This may be less than the number of
characters that your operating system supports.

SEE ALSO
--------

The mpiFileUtils source code and all documentation may be downloaded
from <https://github.com/hpc/mpifileutils>

README
======

``pg-uncorrupt`` is an one-file utility to easy recovering ``Postgres``'s
bad-pages from a standby copy.

Status
------

v2.0 release.

Usage Example
-------------

Suppose you have next error in ``Postgres``::

    invalid page in block 21359616 of relation base/16393/22556601

The utility can help with recovering like this::

    $ ./pg-uncorrupt -v -- 21359616 base/16393/22556601 \
            root@my-standby-server:/var/lib/postgres/data \
            root@my-primary-server:/var/lib/postgres/data

The verbose output would be the next::

    source file: 'root@my-standby-server:/var/lib/postgres/data/base/16393/22556601.162'
    destination file: 'root@my-primary-server:/var/lib/postgres/data/base/16393/22556601.162'
    file offset: 125952
    source page header's content:
      PageHdr(pd_lsn=10004628924459422639, pd_checksum=0, pd_flags=0, pd_lower=40, pd_upper=64, pd_special=8192, pd_pagesize_version=8196, pd_prune_xid=0)
    destination page header's content:
      PageHdr(pd_lsn=3449757388383453185, pd_checksum=33, pd_flags=12256, pd_lower=24543, pd_upper=8192, pd_special=0, pd_pagesize_version=1, pd_prune_xid=0)
    source page is empty: False
    destination page is empty: False
    source page header is valid: True
    destination page header is valid: False
    page headers are equivalent: False
    replacing the page successfully!

This replaces (via ``ssh``) the bad page (in primary server) with the good
page (from the standby server), **after making checks that the page replacing is
really needed**.

*The utility won't replace a good page with another good one, only a bad page
could be replaced, it is important for keeping your data safe.*

I.e. relaunching of the utility would raise an error like this::

    destination page header is already valid, and page headers are already equivalent! stop replacing the page

You could use **"pretend mode"** to seeing what would be if we tried to
recovery the bad page::

    $ ./pg-uncorrupt -vp -- 21359616 base/16393/22556601 \
            root@my-standby-server:/var/lib/postgres/data \
            root@my-primary-server:/var/lib/postgres/data
    
    source file: 'root@my-standby-server:/var/lib/postgres/data/base/16393/22556601.162'
    destination file: 'root@my-primary-server:/var/lib/postgres/data/base/16393/22556601.162'
    file offset: 125952
    source page header's content:
      PageHdr(pd_lsn=10004628924459422639, pd_checksum=0, pd_flags=0, pd_lower=40, pd_upper=64, pd_special=8192, pd_pagesize_version=8196, pd_prune_xid=0)
    destination page header's content:
      PageHdr(pd_lsn=3449757388383453185, pd_checksum=33, pd_flags=12256, pd_lower=24543, pd_upper=8192, pd_special=0, pd_pagesize_version=1, pd_prune_xid=0)
    source page is empty: False
    destination page is empty: False
    source page header is valid: True
    destination page header is valid: False
    page headers are equivalent: False
    replacing the page successfully! (really no! we are just pretending)

Full options list is in the help page of the utility::

    $ ./pg-uncorrupt -h
    usage: pg-uncorrupt [-h] [-v] [-p] [-e COMMAND] [-d] page_num rel src_data dst_data
    
    An one-file utility to easy recovering Postgres's bad-pages from a standby copy
    
    positional arguments:
      page_num              page number
      rel                   relation
      src_data              source data path
      dst_data              destination data path
    
    optional arguments:
      -h, --help            show this help message and exit
      -v, --verbose         be verbose
      -p, --pretend         stop before writing anything
      -e COMMAND, --rsh COMMAND
                            specify the remote shell to use
      -d, --dump            dump page to file only. do not perform a recovering procedure

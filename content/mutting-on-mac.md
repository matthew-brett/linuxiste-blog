Title: Compiling mutt on OSX
Date: 2013-04-29 17:30
Tags: Mutt, OSX
Category: mail, OSX, compiling
Slug: mutt-on-mac
Author: Matthew Brett
Summary: Compiling Mutt on OSX

I first half-read some pages:

<http://linsec.ca/Using_mutt_on_OS_X>
<http://thomer.com/howtos/mutt_on_mac.html>
<http://web.archive.org/web/20100603225320/http://linsec.ca/Using_mutt_on_OS_X>

## My setup

I compile everything into `/Users/mb312/usr/local`

I have the `LIBRARY_PATH` and `LD_LIBRARY_PATH` environment variables set to
include `/Users/mb312/usr/local/lib/` and `CPATH` set to include
`/Users/mb312/usr/local/include/`

`/Users/mb312/usr/local/bin` is on my `PATH`.

I tend to install release tarballs from a directory `/Users/mb312/stable_trees`
and I put development code into `/Users/mb312/dev_trees`.

## Install

### gdbm library for header cache databasing

For IMAP use, it's good to be able to cache headers locally (see mutt
`header_cache` variable configuration).  You have a choice of various
databases, including Berkeley DB4 and [Toyko
Cabinet](http://fallabs.com/tokyocabinet/).  I chose
[gdbm](http://www.gnu.org.ua/software/gdbm/).

    cd ~/stable_trees/
    curl -O ftp://ftp.gnu.org/gnu/gdbm/gdbm-1.10.tar.gz
    tar zxvf gdbm-1.10.tar.gz
    cd gdbm-1.10
    ./configure --prefix=/Users/mb312/usr/local
    make
    make install

### SASL library for SMTP authentication

<http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer>
<http://cyrusimap.web.cmu.edu/mediawiki/index.php/Downloads>

    cd ~/stable_trees
    curl -O ftp://ftp.cyrusimap.org/cyrus-sasl/cyrus-sasl-2.1.26.tar.gz
    tar zxvf cyrus-sasl-2.1.26.tar.gz
    cd cyrus-sasl-2.1.26/
    ./configure --prefix=/Users/mb312/usr/local
    make install
    # This will need to go into my bash_profile
    export SASL_PATH=/Users/mb312/usr/local/lib/sasl2

### Mutt

I installed the development version of mutt - see: [mutt
development](http://dev.mutt.org/trac/)

    cd ~/dev_trees
    hg clone http://dev.mutt.org/hg/mutt#HEAD
    cd mutt
    ./prepare --prefix=/Users/mb312/usr/local --with-curses  --with-regex \
        --enable-locales-fix --enable-pop --enable-imap --enable-smtp \
        --with-sasl --enable-hcache --with-ssl
    make

When doing `make install` I got this error:

    chgrp: you are not a member of group mail
    Can't fix mutt_dotlock's permissions!  This is required to lock mailboxes in the mail spool directory.

So I added myself to the `mail` group using this [user to group command
line](http://superuser.com/questions/214004/how-to-add-user-to-a-group-from-mac-os-x-command-line):

    sudo dseditgroup -o edit -a mb312 -t user mail

and:

    make install

<!--- vim:ft=markdown -->

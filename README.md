# UFDS

Repository: <git@git.joyent.com:ufds.git>
Browsing: <https://mo.joyent.com/ufds>
Who: Mark Cavage, Pedro Palazón Candel
Docs: oioi, you rather check the code for now.
Tickets/bugs: <https://devhub.joyent.com/jira/browse/PUBAPI>

# Overview

UFDS is the "unified foundational directory service" built for SDC over
[ldapjs](http://ldapjs.org) and [moray](https://mo.joyent.com/docs/moray/master/),
and is used to track accounts, credentials, and more. It is a superset of
functionality offered by previous SDC versions employing CAPI (there is a
backwards compatible "shim" that offers the same API as CAPI did in SDC 6.5).

# Development

    git clone git@git.joyent.com:ufds.git
    cd ufds
    git submodule update --init
    make all
    node main.js -f ./etc/ufds.laptop.config.json -d 2 2>&1 | bunyan

This assumes several things:

- You've got a moray instance running exactly how it's specified on the
  config file `ufds.laptop.config.json`.
- Your node version is greater than 0.6 (package says 0.7, but it's working OK
  with 0.6.x so far).
- You want to see debug output. If you don't, remove the `-d 2`, but it's
  strongly recommended while hacking.
- You do have bunyan module installed globally. Given the `make all` command
  should also install it locally, you could also replace that with
  `./node_modules/.bin/bunyan`.

# Testing

File by file:

    ./node_modules/.bin/nodeunit test/add.test.js

or the make task for this file:

    make add_test

The following make tasks are available:

    add_test
    bind_test
    compare_test
    del_test
    mod_test
    search_test

Of course, if you run `make test` all these tasks will run.

# Schema

Schema for UFDS is built on a custom framework where you extend a
`Validator` class, and simply model the attributes you want, whether
they're required or optional, and the number of values to allow.  This
means to add new schema types into UFDS, you have to write (minimal)
code.  Take a look at `./schema` to get a feel for what this looks
like.  It's really not rocket science.

The schema framework automatically runs on add/modify/modifyDN, and
UFDS "discovers" all schema in that directory, so all you need to do
to get a new type in the system is drop a file in there.

In terms of the paradigm, you describe your _required_ attributes, and
the number of values each can have, and then decide whether or not you
want the type to be _strict_.  Strict set to true means that only the
attributes described in your schema will be allowed, and you can then
use the _optional_ block to describe optional attributes.  If _strict_
is false, then _optional_ is pretty much irrelevant, as anything
goes (_required_ attributes however, must be present). The default for
strictness is _false_.


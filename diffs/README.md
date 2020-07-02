# Handshake-Bridge Patches

These are patches for the [Handshake.org](http://www.handshake.org) [hsd](https://github.com/handshake-org/hsd) project.

## `dumpzone.diff`

This patch adds the `rpc` call `dumpzone <filename>` which will dump all the handshake zone data to the file specified.

It first dumps to the file name with a `~` suffix, then when the dump is done, it will rename the file.

This makes is easier for an external program to know when the dump has completed.

Typically a dump takes a few tens of seconds, depending on your CPU


## nosign.diff

`NS` referral record in a parent zone should **NOT** be signed. This is a bug in `hsd`.

This patch fixes this bug, which also means the `dumpzone` runs about 10x faster.

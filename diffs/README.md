# Handshake-Bridge Patches

These are patches for the [Handshake.org](http://www.handshake.org) project [hsd](https://github.com/handshake-org/hsd).

## `dumpzone.diff`

This patch adds the `rpc` call `dumpzone <filename>` which will dump all the handshake zone data to the file specified.

It first dumps to the file name with a `~` suffix then, when the dump is done, it will rename the file to the name you specified.

This makes is easier for an external program to know when the dump has completed.

Typically a dump takes a few tens of seconds, depending on your CPU.


It is "normal" for the `dumpzone` to cause a `timeout` error in the `rpc` client, becuase the entire dump takes longer than
the `rcp` client allows. However, the dump will still complete, so just ignore this error.

If you have `debug` messages enabled in your `hsd`, it will report dump progress every `5000` names.

*Eventually* I intend to create a patch to add `AXFR` support (standard TCP Zone Transfer request). Until then, this will do.

This patch is just a modified version of [Mark Tyneway's](https://github.com/tynes) original [dumpzone patch](https://github.com/handshake-org/hsd/pull/280)


## `nosign.diff`

`NS` referral record in a parent zone should **NOT** be signed. This is a bug in `hsd`.

This patch fixes this bug and disabled the `RRSIG` record for `NS` referrals. This means the `dumpzone` runs about 10x faster.

This patch is optional.

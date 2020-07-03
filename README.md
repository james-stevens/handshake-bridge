# handshake-bridge
## Bridging Handshake &amp; ICANN TLDs

The purpose of this project is to provide a bridge between the existing (ICANN maintained) public DNS infrastructure and the Handshake project
while providing at least the same level of security as the public infrastrusture.

To do this it will
- Take a copy of both ROOT zones
- Merge them
- Sign them with your own set of keys (KSK & ZSK)
- Provide an AXFR/IXFR Service & Resolver Service of the merged, signed ROOT zone

You can then tell any DNSSEC aware DNS software to trust your own KSK (by giving it the public key), use this ROOT service / resolver
and all the ICANN & Handshake data will correctly validate.

You can do this by either using the resolver service directly, or by running a DNSSEC-aware stubb resolver on your desktop
which has been told to trust your own KSK's Public Key.


To use this project, you **must** apply the `hsd` `dumpzone` patch included as `diffs/dumpzone.diff`

I would also strongly recommend you apply the `diffs/nosign.diff` patch.

There is a `README` in the `diffs` directory which gives more information.



## Getting it working

### Prerequisites

- Ensure you have `hsd` running correctly
- Ensure you have the full `bind` package installed, including `named`, `rndc`, `rndc-confgen`, `dnssec-keygen` & `dnssec-signzone`


**NOTE:** Until XFR is working in `hsd`, the transfer of handshake data requires shared file space. This can be done in containers, but is more painful
then just running on the same host. So for the time being running `hsd` and this on the same host is probably the easiest option.



### Actions

- Clone this project and go into the project directory
- Edit the file `config` to meet your needs
- Run `./bin/setup`

If the `set-up` runs OK you should now have `bind` running, as the process `named-handshake-bridge`, providing four separate DNS services. They are
- A local mirror of the ICANN ROOT zone
- A authoritative master with the signed merged ROOT zone, which only offers AXFR support
- A authoritative master with the signed merged ROOT zone, which can offer AXFR & IXFR support
- A validating resolver that fully supports DNSSEC and trusts your locla ROOT KSK for validating any DNSSEC data (ICANN or Handshake)

These run on the IP Addresses you gave in the `config` as well as `127.0.0.0/8` IPs as follows
- `{{config.bind_local_prefix}}.1` -> Resolver
- `{{config.bind_local_prefix}}.2` -> AXFR/IXFR Merged ROOT Service
- `{{config.bind_local_prefix}}.3` -> ICANN Mirror
- `{{config.bind_local_prefix}}.5` -> AXFR Only Merged Root



If this is running correctly, you can run the program `./bin/check` and it will run a few basic checks.

The output you should see is the SOA RR for each of these four services. Three of the hour should give the same result, 
which is the SOA RR of the merged & locally signed ROOT zone and one will give the ICANN ROOT SOA RR.


You should be able to run `set-up` either as `root` or an ordinary user (I tested it as an ordinary user), but some operations on some platforms
require `root` permission, so I have preceded these with `sudo`. If you don't use `sudo`, just modify the `set-up`
script to whatever you do use.

`set-up` will install a `cron` job which runs every 15 minutes to execute `${base}/bin/handshake-bridge-cronjob`. This
polls the SOA serial of both ROOT zones to check if either needs refreshing. This is not ideal, but neither change
that often, so I'm sure its fine.

Once `hsd` has AXFR support and the handshake SOA Serial only rolls when the data is updated, this can be improved


## Using `rndc` to monitor the services

An `rndc` key `conf` file is story in `etc/rndc.conf` and `bind` will accept `rndc commands for any of the
services on the IP Address `{{config.bind_local_prefix}}.5` (default `127.9.0.5`).

To see the status of the `bind` run `rndc -c etc/rndc.conf status`

If you edit the file `named/etc/named.conf` you can tell `bind` to reload it with `rndc -c etc/rndc.conf reconfig`. 
I strongly recommend you run `named-checkconf` on it first.



## How do I actually use this?


### Use it directly from your client

If you have given the resolver service an externally addressable IP Address, all you have to tell your client's to
use this resolver and you will be able to resolve both ICANN & Handshake DNS with full DNSSEC support

If you wish to continue to use an external resolver, you can do this in one of two ways



### Forward your existing resolver to this Resolver

In `bind` you can forward queries to a higher level resolver with the following

	forward only;
	forwarders { [config.bind_recusive_addr] };



### Slave the signed ROOT from the IXFR service into your existing resolver

In `bind` you can slave the merged ROOT, tell your bind to trust the local KSK and continue to use your 
existing `bind` resolver.

- Copy the `trusted-key.inc` file from this directory
- Run `ip addr add 127.12.12.12/8 dev lo` on the resolver (`bind` is fussy about this)
- Add this into your resolvers `named.conf`

	include "trusted-key.inc";
	view root {
		match-destinations { 127.12.12.12; };
		zone "." {
			type slave;
			file "rootzone.db";
			notify no;
			masters { [config.bind_slave_addr]; };
			};
		};
	zone "." { type static-stub; server-addresses { 127.12.12.12; }; };

This will cause your resolver to mirror the merger & signed ROOT zone from the IXFR service and use
that in preference to the public Internet ROOT data.



## More I want to get done

Desired mods to `hsd` (in order)

- Add db locks, so the db can't be transferred or dumped while it is being updated - this maybe unnecessary,
if the SOA only changes after a DB update
- SOA only changes when the database is updated (every 36 blocks)
- Add AXFR support for easier container use


Once AXFR is working, it will be far easier to run this in a separate container from `hsd` as it will have access to the handshake ROOT 
data over a socket.



## Background References

If you are unfamiliar with the concept & practice of signing the ROOT zone with your own keys, 
[this is](https://dnsworkshop.de/local-augmented-root-zone.html) an excellent reference.

And [here is](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/) an excellent
general background in DNSSEC from Cloudflare.

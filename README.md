# handshake-bridge
## Bridging Handshake &amp; ICANN TLDs

The purpose of this project is to provide a bridge between the existing (ICANN maintained) DNS operation and the Handshake project.

To do this it will
- Take a copy of both ROOT zones
- Merge them
- Sign them with your own set of keys (KSK & ZSK)
- Provide an AXFR/IXFR Service & resolver service of the merged, signed ROOT zone

You can then tell any DNSSEC aware DNS software to trust you own KSK (by giving it the public key) then use this ROOT service / resolver
and all the ICANN & Handshake data will correctly validate.

You can do this by either using the resolver service directly, or by running a DNSSEC-aware stubb resolver on your desktop
which has been told to trust your own KSK's Public Key.


To use this project, you **must** apply the `dumpzone` patch included as `diffs/dumpzone.diff`

I would also strongly recommend you apply the `diffs/nosign.diff` patch.

There is a `README` in the `diffs` directory which gives more information.



## Getting it working

### Prerequisits

- Ensure you have `hsd` running corrrectly
- Ensure you have the full `bind` package installed, including `named`, `rndc`, `rndc-confgen`, `dnssec-keygen` & `dnssec-signzone`


**NOTE:** Until XFR is working in `hsd`, the transfer of handshake data requires chared file space. This can be done in containers, but is more painful
then just running on the same host. So for the time being running `hsd` and this ont he same host is probably the easiest option.



### Actions

- Clone this project and go into the project durectory
- Edit the file `config` to meet your needs
- Run `./bin/setup`

If the `set-up` runs OK you should now have `bind` running four separate DNS sevices. They are
- A local mirror of the ICANN ROOT zone
- A authritative master with the signed merged ROOT zone, which only offers AXFR support
- A authritative master with the signed merged ROOT zone, which can offer AXFR & IXFR support
- A validating resolver that fully supports DNSSEC and trusts your locla ROOT KSK for validting any DNSSEC data (ICANN or Hansshake)


If this is running correctly, you can run the program `./bin/check` and it will run a few basic checks.

The output you should see is the SOA RR for each of these four services. Three of the hour should give the same result, 
which is the SOA RR of the merged & locally signed ROOT zone and one will give the ICANN ROOT SOA RR.


You should be able to run `set-up` either as `root` or an ordinary user, but soem operations on some platforms
require `root` permission, so I have preceeded this with `sudo`. If you don't use `sudo`, just modify the `set-up`
script to whatever you do use.

`set-up` will install a `cron` job which runs every 15 minutes to execute `bin/handshake-bridge-cronjob`. This
polls the SOA serial of both ROOT zones to check if either needs refreshing. This is not ideal, but neither change
thaty often, so I'm sure its fine.



## How do I actually use this?

If you have given the resolver service an externally addressable IP Address, all you have to 


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

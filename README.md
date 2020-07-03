# handshake-bridge
## Bridging Handshake &amp; ICANN TLDs

The purpose of this project is to provide a bridge between the existing DNS operation and the Handshake project.

To do this it will
- Take a copy of both ROOT zones
- Merge them
- Sign them with your own set of keys (KSK * ZSK)
- Provide an ROOT Server, AXFR/IXFR Service & resolver service of the merged, signed data

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
- Ensure you have the full `bind` package installed - `named`, `rndc`, `rndc-confgen`, `dnssec-keygen` & `dnssec-signzone`


### Actions

- Clone this project and go into the project durectory
- Edit the file `config` to meet your needs
- Run `./bin/setup`



## More I want to get done

Desired mods to `hsd` (in order)

- Add db locks, so the db can't be transferred or dumped while it is being updated - this maybe unnecessary,
if the SOA only changes after a DB update
- SOA only changes when the database is updated (every 36 blocks)
- Add AXFR support for easier container use


## References

If you are unfamiliar with the concept & practice of signing the ROOT zone with your own keys, 
[this is](https://dnsworkshop.de/local-augmented-root-zone.html) an excellent reference.

And [here is](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/) an excellent
general background in DNSSEC from Cloudflare.

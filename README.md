# handshake-bridge
## Bridging Handshake &amp; ICANN TLDs

The purpose of this project is to provide a bridge between the existing DNS operation and the Handshake project.

The goals are
- Be able to resolve handshake & ICANN TLDs in a single instance of `bind` with full DNSSEC support
using your own (private) ROOT zone keys (KSK & ZSK), which you can choose to trust.
- Ensure full DNSSEC capability, so the data can be trusted no matter how it is come-by

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

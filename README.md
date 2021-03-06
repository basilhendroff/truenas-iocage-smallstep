# truenas-iocage-smallstep
This script will create an iocage jail on TrueNAS CORE 12.0 with the latest Smallstep release.
## Status
This script will work with TrueNAS CORE 12.0. It's designed to operate on FreeBSD 12.2 or later, which uses [certctl](https://www.freebsd.org/cgi/man.cgi?query=certctl&apropos=0&sektion=0&manpath=FreeBSD+11.4-stable&arch=default&format=html) to manage the local trust store.
## Usage
Use Smallstep for complete certificate lifecycle managment on the local network (private PKI). It can manage private TLS/SSL certificates for internal workloads, devices and people. Smallstep supports the ACME protocol, single sign-on, one-time tokens, VM APIs, and other methods for automating certificates.

This script enhances the SmallStep environment and establishes some best practices as follows:
1. It runs the `step-certificates` package within a jail to enable the following behaviour: 
   - It uses a modified rc script that works better for jails.
   - Modular operation - The modified rc script separates set up of the `step-ca` jail environment from initialisation of the Smallstep CA.
   - Sequenced operation - The CA must be initialised and a password saved before `step-ca` can be enabled. This is handled by the rc `required_files` parameter.
   - The modified rc script sets root:wheel as the owner:group of `step-ca` to allow the use of low order port 443.
   - It uses the local trust store of the jail rather than the TrueNAS systemwide trust store.  
2. Sets up the STEPPATH enviromental variable in the jail shell.
3. Stores public certificates, private keys and other assets outside the jail.

### Prerequisites
Although not required, it's recommended to create a Dataset named `apps` with a sub-dataset named `smallstep` on your main storage pool.  Many other jail guides also store their configuration and data in subdirectories of `pool/apps/` If this dataset is not present, a directory `/apps/smallstep` will be created in `$POOL_PATH`.
### Installation

Download the repository to a convenient directory on your TrueNAS system by changing to that directory and running `git clone https://github.com/basilhendroff/truenas-iocage-smallstep`. Then change into the new truenas-iocage-smallstep directory and create a file called `smallstep-config` with your favorite text editor. In its minimal form, it would look like this:

```
JAIL_IP="192.168.1.199"
DEFAULT_GW_IP="192.168.1.1"
```

Many of the options are self-explanatory, and all should be adjusted to suit your needs, but only a few are mandatory. The mandatory options are:

- JAIL_IP is the IP address for your jail. You can optionally add the netmask in CIDR notation (e.g., 192.168.1.199/24). If not specified, the netmask defaults to 24 bits. Values of less than 8 bits or more than 30 bits are invalid.
- DEFAULT_GW_IP is the address for your default gateway.

In addition, there are some other options which have sensible defaults, but can be adjusted if needed. These are:

- JAIL_NAME: The name of the jail, defaults to `smallstep`.
- POOL_PATH: The path for your data pool. It is set automatically if left blank.
- DATA_PATH: This is the path for SmallStep configuration and storage, defaults to `$POOL_PATH/apps/smallstep`. This is mounted inside the jail at `/var/db/step_ca`. 
- INTERFACE: The network interface to use for the jail. Defaults to `vnet0`.
- VNET: Whether to use the iocage virtual network stack. Defaults to `on`.

### Execution

Once you've downloaded the script and prepared the configuration file, run this script (`./smallstep-jail.sh`). The script will run for several minutes. When it finishes, your jail will be created and Smallstep will be installed.

### Testing

Enter the jail and check the version of step and step-ca.
```
root@smallstep:~ # step-ca -version
Smallstep CA/0.17.2 (freebsd/amd64)
Release Date: 2021-12-05 02:24 UTC
root@smallstep:~ # step -version
Smallstep CLI/0.17.2 (freebsd/amd64)
Release Date: 2021-12-05 02:24 UTC
```

### Where to from here?

Note: Wherever you see `$(STEP PATH)` used in the Smallstep guides, replace it with `${STEPPATH}`. 

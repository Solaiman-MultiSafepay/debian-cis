# CIS Debian 7/8 Hardening

Modular Debian 7/8 security hardening scripts based on [cisecurity.org](https://www.cisecurity.org)
recommendations. We use it at [OVH](https://www.ovh.com) to harden our PCI-DSS infrastructure.

```console
$ bin/hardening.sh --audit-all
[...]
hardening [INFO] Treating /opt/cis-hardening/bin/hardening/13.15_check_duplicate_gid.sh
13.15_check_duplicate_gid [INFO] Working on 13.15_check_duplicate_gid
13.15_check_duplicate_gid [INFO] Checking Configuration
13.15_check_duplicate_gid [INFO] Performing audit
13.15_check_duplicate_gid [ OK ] No duplicate GIDs
13.15_check_duplicate_gid [ OK ] Check Passed
[...]
################### SUMMARY ###################
      Total Available Checks : 191
         Total Runned Checks : 191
         Total Passed Checks : [ 170/191 ]
         Total Failed Checks : [  21/191 ]
   Enabled Checks Percentage : 100.00 %
       Conformity Percentage : 89.01 %
```

## Quickstart

```console
$ git clone https://github.com/ovh/debian-cis.git && cd debian-cis
$ cp debian/default /etc/default/cis-hardening
$ sed -i "s#CIS_ROOT_DIR=.*#CIS_ROOT_DIR='$(pwd)'#" /etc/default/cis-hardening
$ bin/hardening/1.1_install_updates.sh --audit-all
1.1_install_updates [INFO] Working on 1.1_install_updates
1.1_install_updates [INFO] Checking Configuration
1.1_install_updates [INFO] Performing audit
1.1_install_updates [INFO] Checking if apt needs an update
1.1_install_updates [INFO] Fetching upgrades ...
1.1_install_updates [ OK ] No upgrades available
1.1_install_updates [ OK ] Check Passed
```

## Usage

### Configuration

Hardening scripts are in ``bin/hardening``. Each script has a corresponding
configuration file in ``etc/conf.d/[script_name].cfg``.

Each hardening script can be individually enabled from its configuration file.
For example, this is the default configuration file for ``disable_system_accounts``:

```
# Configuration for script of same name
status=disabled
# Put here your exceptions concerning admin accounts shells separated by spaces
EXCEPTIONS=""
```

``status`` parameter may take 3 values:
- ``disabled`` (do nothing): The script will not run.
- ``audit`` (RO): The script will check if any change *should* be applied.
- ``enabled`` (RW): The script will check if any change should be done and automatically apply what it can.

Global configuration is in ``etc/hardening.cfg``. This file controls the log level
as well as the backup directory. Whenever a script is instructed to edit a file, it
will create a timestamped backup in this directory.

### Run aka "Harden your distro"

To run the checks and apply the fixes, run ``bin/hardening.sh``.

This command has 2 main operation modes:
- ``--audit``: Audit your system with all enabled and audit mode scripts
- ``--apply``: Audit your system with all enabled and audit mode scripts and apply changes for enabled scripts

Additionally, ``--audit-all`` can be used to force running all auditing scripts,
including disabled ones. this will *not* change the system.

``--audit-all-enable-passed`` can be used as a quick way to kickstart your
configuration. It will run all scripts in audit mode. If a script passes,
it will automatically be enabled for future runs. Do NOT use this option
if you have already started to customize your configuration.

``--sudo``: Audit your system as a normal user, but allow sudo escalation to read
specific root read-only files. You need to provide a sudoers file in /etc/sudoers.d/
with NOPASWD option, since checks are executed with ``sudo -n`` option, that will
not prompt for a password.

## Hacking

**Getting the source**

```console
$ git clone https://github.com/ovh/debian-cis.git
```

**Building a debian Package** (the hacky way)

```console
$ debuild -us -uc
```

**Adding a custom hardening script**

```console
$ cp src/skel bin/hardening/99.99_custom_script.sh
$ chmod +x bin/hardening/99.99_custom_script.sh
$ cp src/skel.cfg etc/conf.d/99.99_custom_script.cfg
```

Code your check explaining what it does then if you want to test

```console
$ sed -i "s/status=.+/status=enabled/" etc/conf.d/99.99_custom_script.cfg
$ ./bin/hardening/99.99_custom_script.sh
```

## Disclaimer

This project is a set of tools. They are meant to help the system administrator
built a secure environment. While we use it at OVH to harden our PCI-DSS compliant
infrastructure, we can not guarantee that it will work for you. It will not
magically secure any random host.

Additionally, quoting the License:

> THIS SOFTWARE IS PROVIDED BY OVH SAS AND CONTRIBUTORS ``AS IS'' AND ANY
> EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
> WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
> DISCLAIMED. IN NO EVENT SHALL OVH SAS AND CONTRIBUTORS BE LIABLE FOR ANY
> DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
> (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
> LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
> ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
> (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
> SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Reference

- **Center for Internet Security**: https://www.cisecurity.org/
- **CIS recommendations**: https://benchmarks.cisecurity.org/downloads/show-single/index.cfm?file=debian7.100
- **CIS recommendations**: https://benchmarks.cisecurity.org/downloads/show-single/index.cfm?file=debian8.100

## License

3-Clause BSD


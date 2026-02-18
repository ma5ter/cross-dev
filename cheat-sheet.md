# Crossdev Cheat-sheet

## Prerequisites

0. Install Gentoo!
1. Install crossdev package in Gentoo `emerge --ask sys-devel/crossdev` and create crossdev overlay 
```
mkdir -p /var/db/repos/crossdev/{profiles,metadata}
echo 'crossdev' > /var/db/repos/crossdev/profiles/repo_name
echo 'masters = gentoo' > /var/db/repos/crossdev/metadata/layout.conf
chown -R portage:portage /var/db/repos/crossdev
```
2. Build cross toolchain with "_musl_" tuple on target. In the example bellow the command is given for device named "_ma35h0_" (can be customized to any device name) which has "_armv7a_" architecture ```crossdev --show-fail-log --stable -t armv7a-ma35h0-linux-musleabihf --ex-gdb`
3. After successful finishing of cross toolchain building go to `cd /usr/armv7a-ma35h0-linux-musleabihf/etc/portage`, there you can find the toolchain's `make.conf` file. Fix there in the `CFLAGS` parameters build option from `-02` to `-0s`. Also add `MAKEOPTS` parameter according to the numbers of cores of your CPU (e.g. `MAKEOPTS="-j4"` for 4 core CPU), and `ACCEPT_LICENSE="*"` with yours the best options of gentoo source mirrors `GENTOO_MIRORS="paste_the_link_to_the_source_mirror"` (refer to [this page with gentoo mirrors](https://www.gentoo.org/downloads/mirrors/)). Make sure that `CHOST` and `CBUILD` parameters is correct for your toolchain.
4. For the builded via crossdev toolchains the portage package masking and using structure also can be used. Again go to `cd /usr/armv7a-ma35h0-linux-musleabihf/etc/portage`. Create `package.mask` and `package.use` directories, then add the following uses:


**/usr/armv7a-ma35h0-linux-musleabihf/etc/portage/package.mask/02_not_welcomed**
   ```
    >=sys-kernel/linux-headers-5.11
    >=sys-kernel/gentoo-sources-5.11
    >=dev-lang/python-3.13
   ```
**/usr/armv7a-ma35h0-linux-musleabihf/etc/portage/package.use/00_base**

    */* -udev -nls -dbus
    sys-apps/util-linux pam

**/usr/armv7a-ma35h0-linux-musleabihf/etc/portage/package.use/02_libs**

    dev-libs/glib -introspection

**/usr/armv7a-ma35h0-linux-musleabihf/etc/portage/package.use/03_python**

    dev-lang/python -ensurepip ncurses readline -sqlite -ssl -bluetooth


    ############ WARNING !!!! Portage bug! ###############

    # Host package.mask block installation in overlays too !

    ######################################################

    # Python 12 is the main version despite profile settings

    */* PYTHON_TARGETS: -* python3_12
    */* PYTHON_SINGLE_TARGET: -* python3_12


    # Safer upgrade procedure
    # =======================
    # A safer approach is to add Python 3.13 support to your system first,
    # and only then remove Python 3.12.  However, note that this involves two
    # rebuilds of all the affected packages, so it will take noticeably
    # longer.
    #
    # First, enable both Python 3.12 and Python 3.13, and then run the upgrade
    # commands:
    #
    #     */* PYTHON_TARGETS: -* python3_12 python3_13
    #     */* PYTHON_SINGLE_TARGET: -* python3_12
    #
    # Then switch PYTHON_SINGLE_TARGET and run the second batch of upgrades:
    #
    #     */* PYTHON_TARGETS: -* python3_12 python3_13
    #     */* PYTHON_SINGLE_TARGET: -* python3_13
    #
    # Finally, switch to the final version and upgrade:
    #
    #     */* PYTHON_TARGETS: -* python3_13
    #     */* PYTHON_SINGLE_TARGET: -* python3_13

5. Create directory for binary packages `mkdir -p /usr/armv7a-ma35h0-linux-musleabihf/var/cache/binpkgs`. Then updates system files of toolchain via portage `emerge-armv7a-ma35h0-linux-musleabihf --update --newuse --deep -av @system`. There will may be some file collisions and portage may propose to abort process by pressing Ctrl+C. Ignore it, because portage just will successfully overwrite those files, which are builded by crossdev and were not defined by portage.
If on this stage you have any troubles, refer to the troubleshooting's section 1 below.


## Troubleshooting

1. **(Deprecated)** If you have problems with the toolchain system packages emerging, probably portage tries to update not only toolchain packages, but the host system packages too. As long as the builded toolchain uses _musl_, there is no need _glibc_, and our toolchain just builded we can ignore already existing package and only add "newuse" packages. So, then use the command bellow to emerge new packages with ignoring _glibc_:
`emerge-armv7a-ma35h0-linux-musleabihf --newuse --deep -av --exclude="sys-libs/glibc" @system`
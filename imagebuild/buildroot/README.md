# Buildroot Ironic Python Agent

**NOTE: This is a work in progress**

This uses the upstream Buildroot Git repository, and one specific to IPA,
to create the following for running Ironic Python Agent:

* Linux kernel
* Initial ramdisk
* Bootable ISO image

For more details on how the build works, please see Buildroot IPA
documentation:

https://github.com/csmart/ipa-buildroot#openstack-ironic-python-agent

The following scripts in this repo can help you to build and customise your
IPA image. Each script serves a primary purpose and can take several arguments.

| Script | Description |
| --- | --- |
| build-buildroot.sh | Fetch source, configure and build IPA image |
| buildroot-lib.sh | Holds common variables and functions |
| clean-buildroot.sh | Delete files created by the build |
| customise-buildroot.sh | Make changes to Buildroot images |
| install-deps.sh | Installs all packages required for the build |
| Makefile | Use make targets to facilitate build process |

Only the install-deps.sh script requires root privileges (via sudo). All
other Buildroot tasks are run as a non-privileged user.

The build is performed under the following subdirectory by default:

	./build

Finalised images are found under the following subdirectory by default:

	./build/output/images

## Help
To see available Makefile targets, simply run the help target:

	make help

Help is also available for the shell scripts if you pass the --help option:

	./build-buildroot.sh --help
	./clean-buildroot.sh --help
	./customise-buildroot.sh --help

## Build instructions
To build a complete image with the default options, simply run:

	make

or

	./build-buildroot.sh --all

These actions will perform the following tasks automatically:

* Fetch the Buildroot Git repositories
* Load the default IPA Buildroot configuration
* Download and verify all source code
* Build the toolchain
* Use the toolchain to build:
  * System libraries and packages
  * Linux kernel
  * IPA and Python dependencies
* Create the kernel, initramfs and ISO images

The following finalised images will be found under _./build/output/images_:

	bzImage (kernel)
	rootfs.cpio.xz (ramdisk)
	rootfs.iso9660 (ISO image)

These files can be uploaded to glance for use with Ironic.

### Customising the image
There are a number of reasons why you may wish to customise the Buildroot
image. For example:

* Enable/disable packages and libraries
* Enable the root account and set a password
* Provide SSH keys
* Tweak Busybox
* Tweak the Linux kernel

The Makefile has a few targets to help assist with this. For example, to
modify the Buildroot configuration run:

	make menuconfig

The following are also available for Busybox and Linux:

	make busybox-menuconfig
	make linux-menuconfig

Make your changes in the menuconfig, then save and exit.

See the documentation for more details on how to modify the images:

https://github.com/csmart/ipa-buildroot#making-changes

### Saving changes
Changes are only made in the output build directory and can be lost if you
do a clean.

Again, make has some helper targets if you wish to copy the changes into
the IPA Buildroot Git repository. Once there, they can be re-used (or
perhaps committed).

	make saveconfig
	make busybox-saveconfig
	make linux-saveconfig

## Individual step instructions

You can also perform the individual steps at any time, if you require.

For example, if you wanted to do this:
* Fetch the Buildroot Git repos and load the default IPA config
* Copy in some SSH keys for root
* Customise the Buildroot configuration (e.g. enable root) and copy to Git
* Do the build and generate license information

Run these commands:

	./install-deps.sh
	./build-buildroot.sh --fetch --config
	rsync -Pa ~/.ssh ./build/overlay/root/
	./customise-buildroot.sh --menuconfig --saveconfig
	./build-buildroot.sh --build --legal

Or with make:

	make dependencies fetch config
	rsync -Pa ~/.ssh ./build/overlay/root/
	make menuconfig saveconfig build legal

## Cleaning up the build environment
Due to the way Buildroot works, there are a few extra clean options available
(see help for details).

The default action _deletes files created by build_, however it leaves the
following in place:

* Buildroot configuration
* Any source files
* Any Git changes

Run:

	make clean

or

	./clean-buildroot.sh

You can also do a _complete clean of everything_, leaving only pristine
Buildroot Git repositories with:

	make reset

From here you can start a new fresh build.

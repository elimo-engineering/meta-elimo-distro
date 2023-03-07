# meta-elimo-distro

Elimo Initium/Impetus distro Layer for OpenEmbedded/Yocto.

## Description

This is the Elimo Poky reference distro for the Initium EVK and, more generally, Elimo Impetus based devices; together with its dependencies, it provides the definition of a complete image.

This layer can be used in two scenarios:

1. as a distribution for the Initium EVK
1. as a template to create your own distro layer for your custom board based on the Elimo Impetus SoM

This layer is a thin wrapper around Poky; as such, most of the value it adds is in the configuration templates in `conf/distro/*sample*`; if you need to create your own distro (the second scenario above), we suggest you create your own distro layer by copying the contents of this repo, then customising the files in `conf/distro/`

:warning: TL;DR: if you just want to build an image to get started with your Initium, this repo is ok, but it's probably simpler to fetch all dependencies in one go using [elimo-yocto-manifest](https://github.com/elimo-engineering/elimo-yocto-manifest). :warning:

## Dependencies

This layer depends on:

* URI: git@github.com:linux-sunxi/meta-sunxi.git
  * branch: kirkstone
  * revision: HEAD

* URI: https://git.yoctoproject.org/git/poky.git
  * branch: kirkstone
  * revision: kirkstone-4.0.5

* URI: git@github.com:elimo-engineering/meta-elimo.git
  * branch: kirkstone
  * revision: kirkstone-v1.0.0


## Getting started

Follow the usual steps to setup OpenEmbedded and bitbake.

We suggest you clone this repo and all its dependencies in one go using [elimo-yocto-manifest](https://github.com/elimo-engineering/elimo-yocto-manifest) (see "Description", above), but since you are reading here we will assume you decided to do it manually =)

In this case, you will need to go through the following steps. Please note that the commands in the following steps assume you want to use the `kirkstone-v1.0.0` revision, change branch/tags accordingly if you want to use a different one.

### Step 1: clone this repo

```shell
git clone https://github.com/elimoengineering/meta-elimo-distro.git -b kirkstone-v1.0.0
```

### Step 2: clone the dependencies

```shell
git clone git://git.yoctoproject.org/poky -b kirkstone-4.0.5
git clone https://git.openembedded.org/meta-openembedded.git -b kirkstone
git clone https://github.com/linux-sunxi/meta-sunxi.git -b kirkstone
git clone https://github.com/elimoengineering/meta-elimo.git -b kirkstone-v1.0.0
```

### Step 3: initialise a build environment

```shell
. meta-elimo-distro/oe-init-build-env build-elimo-initium
```

You are probably familiar with this if you have used `poky` before.

This script will initialise the environment of your shell so that you can call `bitbake`; it will also give you a default build configuration in `build-elimo-initium/conf/local.conf` and set up your build to reference the appropriate layer dependencies in `build-elimo-initium/conf/bblayers.conf`.

### Step 4: build!

You are now ready to start a build; all usual builds from Poky, from which Elimo Poky is derived, are possible. For a (relatively!) quick test build, you can use `core-image-minimal`, which will give you a textual console on both the USB-UART bridge and the LCD on the Initium EVK.

```shell
bitbake core-image-minimal
```

### Step 5: deploying to SD card

If you're doing this in a Linux environment, you can use the following process to transfer the image to an SD Card.
In this example we're using the `core-image-minimal-elimo-initium.wic` image, if you have built a different image, update the paths accordingly.

First check your SD card path using `lsblk`; in the following commands, replace <X> with your results from `lsblk`.

#### With `dd`

`dd` is already installed on most systems, so this is probably the quickest way to go:

```shell
tmp/deploy/images/elimo-initium
sudo dd if=core-image-minimal-elimo-initium.wic of=/dev/sd<X> bs=4M iflag=fullblock oflag=direct conv=fsync status=progress
```

#### With `bmaptool`

[BMAP Tools](https://github.com/intel/bmap-tools) is a tool specifically designed by Intel to flash embedded system images to mass storage. It has several advantages over `dd`, including efficiency over sparse images, built-in verification and the ability to fetch images remotely over HTTPS, ssh and other protocols.
It is available on Debian based systems to be installed with a simple `sudo apt install bmap-tools`

```shell
cd tmp/deploy/images/elimo-initium
sudo umount /dev/sd<X>
sudo bmaptool copy core-image-minimal-elimo-initium.wic.gz --bmap core-image-minimal-elimo-initium.wic.bmap /dev/sd<X>
```


## Defining your own machine

If you are using the Impetus SoM on your own carrier board, you will most likely want to create a custom `MACHINE` to support the specific hardware on your carrier board. The approach we suggest to this end is to create your own BSP layer based on our one, [meta-elimo](https://github.com/elimo-engineering/meta-elimo)`, either by:

- forking `meta-elimo` and modifying your fork. This is quicker and probably more convenient for small changes.
- creating a new layer altogether and making it depend on `meta-elimo`. You can model this on how `meta-elimo` depends on `meta-sunxi`. This approach is probably more flexible, but requires a bit more initial work.

Once that is done, you can replace `meta-elimo` with your own BSP in the layer config for your build, using `bitbake-layers` (or manually editing `conf/bblayers.conf`)


## Troubleshooting and bug reporting

We hope troubleshooting won't be necessary, but the reality of engineering is that likely it will be. 
If you find an issue in using this code, please create a bug report on the [GitHub issue tracker](https://github.com/elimo-engineering/meta-elimo-distro/issues)


## Submitting patches

Please submit any patches against the `meta-elimo-distro` layer to the [GitHub repo](https://github.com/elimo-engineering/meta-elimo-distro/pulls) as a PR.

Maintainer: Matteo Scordino <matteo@elimo.io>

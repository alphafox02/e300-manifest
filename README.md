Repo Manifests for building systems based on meta-sdr
=============================================
This repository provides Repo manifests to setup the OpenEmbedded build system
with meta-sdr and some interesting boards

OpenEmbedded allows the creation of custom linux distributions for embedded
systems. It is a collection of git repositories known as *layers* each of
which provides *recipes* to build software packages as well as configuration
information.

Repo is a tool that enables the management of many git repositories given a
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup an OpenEmbedded build environment for you!

Getting Started
---------------
1.  Install Repo.

    Download the Repo script.

        $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo

    Make it executable.

        $ chmod a+x repo

    Move it on to your system path.

        $ sudo mv repo /usr/local/bin/

2.  Initialize a Repo client.

    Create an empty directory to hold your working files.

        $ mkdir e300-build
        $ cd e300-build

    Tell Repo where to find the manifest

        $ repo init -u git://github.com/Geontech/e300-manifest.git -b rocko_dev

    A successful initialization will end with a message stating that Repo is
    initialized in your working directory. Your client directory should now
    contain a .repo directory where files such as the manifest will be kept.
    ***
3.  Fetch all the repositories.

        $ repo sync

    Now go put on the coffee machine as this may take 20 minutes depending on
    your connection.

4.  Go into the sdr-build directory:

        $ cd sdr-build

5.  Update the submodules:

        $ git submodule update --init

6.  Move the meta-redhawk-sdr directory into the sdr-build directory   

7.  Initialize the build system

        $ TEMPLATECONF=`pwd`/meta-sdr/conf/conf-e3xx/ source ./openembedded-core/oe-init-build-env ./build ./bitbake

8.  Go into the build directory and edit the bblayers.conf file by adding the line:

    ```PATH_TO_sdr-build/meta-redhawk-sdr \```

    where "PATH_TO_sdr-build" is the absolute path to the directory "sdr-build"

9.  Build an image.

    This process downloads several gigabytes of source code and then proceeds to
    do an awful lot of compilation so make sure you have plenty of space (25GB
    minimum). Go drink some beer.

        $ export MACHINE=ettus-e3xx-sg3
        $ bitbake redhawk-usrp-uhd-image

    If everything goes well, you should have a compressed root filesystem
    tarball as well as kernel and bootloader binaries available in your
    *work/deploy* directory.  If you run into problems, the most likely
    candidate is missing packages.  Check out
    http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources
    for the list of required packagaes for operating system. Also, take
    a look to be sure your operating system is supported:
    https://wiki.yoctoproject.org/wiki/Distribution_Support

10. After building the image and getting it up and running, ssh into the E310
and run the script:

        $ /usr/lib/uhd/utils/uhd_images_downloader.py

Stand-alone Build
-----------------

Do you want your E310 to be a stand-alone REDHAWK Domain complete with components, a GPP, etc.?  The `redhawk-usrp-uhd-image` does not include many of those elements, but each can be easily added by either defining a new image or extending the target image from your `build/conf/local.conf` file:

```
CORE_IMAGE_EXTRA_INSTALL_append = "\
    omniorb-init \
    omnievents-init \
    domain-init \
    node-deployer \
    gpp gpp-node \
    packagegroup-redhawk-basic-components \
    packagegroup-redhawk-basic-softpkgs \
"
COMPATIBLE_MACHINE_pn-sse2neon = "ettus-e300"
```

The first change to `CORE_IMAGE_EXTRA_INSTALL` ensures that the init.d services for OmniNames, OmniEvents, and the Domain are all installed.  The inclusion of the gpp and node-deployer packages will result in a `-ALL` device manager definition being installed that contains both the GPP and the USRP UHD Device.  The last two lines are the package groups for components and softpkgs defined in the layer.

The second change to `COMPATIBLE_MACHINE` addresses the lack of `arm` in the E3xx build environment's `MACHINEOVERRIDES` variable, which prevents the associated `sse2neon` package from being built, which would then prevent the `rh.DataConverter` from being built, and so on.

Between these two changes, the next time you run `bitbake redhawk-usrp-uhd-image`, your root file system will be ready to act as a stand-alone REDHAWK Domain.


Staying Up to Date
------------------
To pick up the latest changes for all source repositories, run:

    $ repo sync

Enter the OpenEmbedded environment:

    $ . openembedded-core/oe-init-build-env ./build ./bitbake

    If you forget to setup these environment variables prior to running bitbake,
    your OS will complain that it can't find bitbake on the path.  Don't try
    to install bitbake using a package manager, just run the command.

You can then rebuild as before:

    $ bitbake redhawk-usrp-uhd-image

Starting from Fresh
-------------------
So it is borked.  You're not really sure why.  But it doesn't work any more.

There are several degrees of *starting fresh*.

 1. clean a package: bitbake <package-name> -c cleansstate
 2. re-download package: bitbake <package-name> -c cleanall
 3. destroy everything but downloads: rm -rf build (or whereever your sstate and work directories are)
 4. destroy it all (not recommended): rm -rf build && rm -rf sources
There are several degrees of *starting fresh*.

Customize
---------
Sooner or later, you'll want to customize some aspect of the image either
adding more packages, picking up some upstream patches, or tweaking your kernel.
To this, you'll want to customize the Repo manifest to point at different
repositories and branches or pull in additional meta-layers.

Clone this repository (or fork it on github):

    $ git clone https://github.com/Geontech/e300-manifest.git -b rocko_dev

Make your changes (and contribute them back if they are generally useful :) ),
and then re-initialize your repo client

    $ repo init -u <file:///path/to/your/git/repository.git>

Please send success stories to philip@balister.org.

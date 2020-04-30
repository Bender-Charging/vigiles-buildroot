![Timesys Vigiles](https://www.timesys.com/wp-content/uploads/vigiles-cve-monitoring.png "Timesys Vigiles")


Timesys Vigiles For Buildroot
=============================

This is a collection of tools for image manifest generation used for security monitoring and notification as part of the **[Timesys Vigiles](https://www.timesys.com/security/vigiles/)** product offering.


What is Vigiles?
================

Vigiles is a vulnerability management tool that provides build-time CVE Analysis of Buildroot target images. It does this by collecting metadata about packages to be installed and uploading it to be compared against the Timesys CVE database.A high-level overview of the detected vulnerabilities is returned and a full detailed analysis can be viewed online.


Register (free) and download the API key to access the full feature set based on Vigiles Basic, Plus or Prime:
https://linuxlink.timesys.com/docs/wiki/engineering/LinuxLink_Key_File


Using Vigiles CVE Check
=======================

To generate a vulnerability report follow the below steps: 


1. Clone vigiles-buildroot repository at the same level as Buildroot directory.

    ```sh
    git clone https://github.com/TimesysGit/vigiles-buildroot
    ```

2. Download your LinuxLink Key File here and store it at the (recommended) path.

    ```sh
    mkdir $HOME/timesys
    cp $HOME/Downloads/linuxlink_key $HOME/timesys/linuxlink_key
    ```

    > Note: If the key is stored elsewhere, the location can be specified via the Buildroot configuration interface.
    >
    > See below for instructions.

3. Instruct Buildroot to include the Vigiles interface in its configuration.
    ```sh
    make BR2_EXTERNAL=/path/to/vigiles-buildroot menuconfig
    ```

    > Note: If you are already using an external Buildroot tree/interface, multiple paths can be concatenated, e.g.:

    >   ```sh
    >   make BR2_EXTERNAL=/path/to/other/external/buildroot/interface:/path/to/vigiles-buildroot menuconfig
    > ```

    > For more information on using external Buildroot interfaces, please see **[This Section of the Buildroot Documentation](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)**

4. Execute Make with the Vigiles target
    ```sh
    make vigiles-check
    ```


5. View the Vigiles CVE (Text) Report Locally

    The CVE report will be located in the ```vigiles/``` subdirectory of your Buildroot build output, with a name based on the Target configuration; e.g.:
    ```sh
    wc -l output/vigiles/buildroot-imx8mpico-report.txt
        3616 output/vigiles/buildroot-imx8mpico-report.txt
    ```

6. View the Vigiles CVE Online Report

    The local CVE text report will contain a link to a comprehensive and graphical report; e.g.:
    ```
    -- Vigiles CVE Report --
            View detailed online report at:
              https://linuxlink.timesys.com/cves/reports/< Unique Report Identifier>
    ```

    #### The CVE Manifest
    The Vigiles CVE Scanner creates a manifest that it sends to the LinuxLink
    Server describing your build configuration. This manifest is located in the
    ```vigiles/``` subdirectory of your Buildroot output (the same location as
    the text report it receives back).
    ```sh
    wc -l output/vigiles/buildroot-imx8mpico-manifest.json 
        557 output/vigiles/buildroot-imx8mpico-manifest.json
    ```
    In the event that something goes wrong, or if the results seem incorrect,
    this file may offer insight as to why. It's important to include this file
    with any support request.


Configuration
=============

If included, Timesys Vigiles will be enabled by default with the Kconfig option
**```BR2_EXTERNAL_TIMESYS_VIGILES```**. In addition, there are other configuration
options available to control the behavior of the subsystem.

> **Note About Pathnames**
>
> Pathnames that are specified follow the same semantics as throughout the
> Buildroot build system
> * Non-absolute paths are relative to the top of the Buildroot source.
>
> * Variables from both the shell and make environments are accessible using
> 'make' syntax; e.g. *```$(HOME)```, ```$(BUILD_DIR)```, ```$(TOPDIR)```*
>
> * Shell expansion is not available; i.e. *```~/```* will not reference
> a user's home directory. Use *```$(HOME)/```* instead.

### Base Configuration

Using ```make menuconfig```, the Vigiles configuration menu can be found under
**```External Options```**.

```
External options  ---> 
    *** Timesys Vigiles CVE Checker (in /home/mochel/projects/buildroot/vigiles) ***
    [*] Enable Timesys Vigiles CVE Check  --->
```

### Reporting and Filtering

Linux Kernel and U-Boot .config filtering can be enabled/disabled using the options
**```VIGILES_ENABLE_KERNEL_CONFIG```** and **```VIGILES_ENABLE_UBOOT_CONFIG```**.

If using a custom location for either the Kernel or U-Boot .config files, the
paths can be specified using **```BR2_EXTERNAL_VIGILES_KERNEL_CONFIG```** and
**```BR2_EXTERNAL_VIGILES_UBOOT_CONFIG```**.

The default for both paths is _```auto```_ which results in automatically using
the .config from the package's configured build directory. It is recommended
that this value is used unless it is absolutely necessary to specify an
alternative path.

```
              *** Vigiles Kernel CVE Reporting Options ***
        [*]   Enable Linux Kernel .config Filtering
        (auto)  Location of Linux Kernel .config file
        [*]   Enable U-Boot .config Filtering
        (auto)  Location of U-Boot .config file
```

> **Note:**
> 
> Linux Kernel .config filtering is only enabled if ```Kernel ---> [*] Linux Kernel ```
>  (**```BR2_LINUX_KERNEL```**) is enabled.
>
> U-Boot .config filtering is enabled only if ```Bootloaders  --->  [*] U-Boot ```
> (**```BR2_TARGET_UBOOT```**) is enabled.


### LinuxLink Credentials

To specify an alternative location for the Timesys LinuxLink Key File, it can
be specified with the string **```BR2_EXTERNAL_VIGILES_KEY_FILE```**.


```
               *** Timesys LinuxLink Account Options ***
        $(HOME)/timesys/linuxlink_key) Timesys LinuxLink Key Location
```

### Vigiles Dashboard Configuration

A custom LinuxLink Dashboard configuration can be set by first
enabling **```VIGILES_ENABLE_DASHBOARD_CONFIG```** and specifying the path in
the string **```BR2_EXTERNAL_VIGILES_DASHBOARD_CONFIG```**.

```
                *** Timesys Vigiles Dashboard Options ***
         [*]   Use a custom Vigiles Dashboard Configuration
         ($(HOME)/timesys/dashboard_config) Timesys Vigiles Dashboard Config Location
```

By default your manifest will be uploaded under the "Private Workspace"
product on the Vigiles Dashboard. To upload your manifest to a particular
product or existing manifest on the Vigiles Dashboard, download the
associated Dashboard Config to `~/timesys/dashboard_config`.

Dashboard Configs downloaded from the [Products page](https://linuxlink.timesys.com/vigiles/)
will upload new manifests to the associated product with a default name.
The link can be found under the Actions column for each product.

_Note_: Click on "New Product" if you have not yet created one.


### Advanced Options

For development purposes, some "Expert" options are available by first enabling
**```VIGILES_ENABLE_EXPERT```**. These allow for debugging of the metadata that
is collected.

These features are not supported and no documentation is provided for them.


```
               *** Advanced Vigiles / Debug Options ***
        [*]   Enable Vigiles Advanced and Debugging Options (Expert)
        [ ]     Generate Manifest but don't Submit for Checking
        [ ]     Write Intermediate JSON Files of Collected Metadata
```


Maintenance
===========

The Vigiles CVE Scanner and Buildroot support are maintained by
[The Timesys Security team](mailto:vigiles@timesys.com).

For Updates, Support and More Information, please see:

[Vigiles Website](https://www.timesys.com/security/vigiles/)

and

[vigiles-buildroot @ GitHub](https://github.com/TimesysGit/vigiles-buildroot)

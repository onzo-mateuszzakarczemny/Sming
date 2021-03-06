Basic rBoot
===========

.. highlight:: bash

Introduction
------------

This sample integrates :component:`rboot` and Sming, for the many people who have
been asking for it. It demonstrates dual rom booting, big flash support,
OTA updates and dual spiffs filesystems. You must enable big flash
support in rBoot and use on an ESP12 (or similar device with 4MB flash).
When using rBoot big flash support with multiple 1MB slots only one rom
image needs to be created. If you don’t want to use big flash support
(e.g. for a device with smaller flash) see the separate instructions
below. You can easily take the ota files and add them to your own
project to add OTA support.

Building
--------

1) Set :envvar:`ESP_HOME` & :envvar:`SMING_HOME`, as environment variables or edit
   component.mk as you would for general Sming app compiling.
2) Set :envvar:`WIFI_SSID` & :envvar:`WIFI_PWD` environment variables with your wifi details.
3) Edit the OTA server details at the top of ``app/application.cpp``.
4) Check overridable variables in component.mk, or set as env vars.
5) ``make && make flash``
6) Put *rom0.bin* and *spiff_rom.bin* in the root of your webserver for OTA.
7) Interact with the sample using a terminal (``make terminal``). Sorry - no web-gui (yet).

Flashing
--------

If flashing manually use *esptool.py* to flash rBoot, rom & spiffs e.g.::

   esptool.py –port write_flash -fs 32m 0x00000 rboot.bin 0x02000 rom0.bin 0x100000 spiffs.rom

Using the correct -fs parameter is important. This will be ``-fs 32m`` on an ESP12.

You can also flash rom0.bin to 0x202000, but booting and using OTA is quicker!

Technical Notes
---------------

``spiffs_mount_manual(address, length)`` must be called from init.

.. note::

   This method is now deprecated. Please configure partitions appropriately,
   use PartitionTable methods to locate the desired partition, then mount it::

      auto part = PartitionTable().find('spiffs0');
      spiffs_mount(part);

   See :ref:`hardware_config` for further details.

Important compiler flags used:

-  BOOT_BIG_FLASH - when using big flash mode, ensures flash mapping code is built in to the rom.
-  RBOOT_INTEGRATION - ensures Sming specific options are pulled in to the rBoot source at compile time.

Flash layout considerations
---------------------------

If you want to use, for example, two 512k roms in the first 1MB block of
flash (old style) then Sming will automatically create two separately linked 
roms. If you are flashing a single rom to multiple 1MB flash blocks, all using
the same offset inside their 1MB blocks, only a single rom is created.
See :component:`rboot` for further details.

-  If using a very small flash (e.g. 512k) there may be no room for a
   spiffs fileystem, so use *HWCONFIG = standard*
-  After building copy all the rom*.bin files to the root of your web
   server.

If you want more than two roms you must be an advanced user and should
be able to work out what to copy and edit to acheive this!

Credits
-------

This sample was made possible with the assistance of piperpilot,
gschmott and robotiko on the esp8266.com forum.

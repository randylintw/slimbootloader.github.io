.. MTL-MCL-board:

|MTL-MCL| Board
---------------------------------

The Intel McLaren Island Reference Design Board (|mtl-mcl|) is an x86 maker board based on Intel platform Meteor Lake. The boards are used in IoT, industrial automation, digital signage areas, etc.

Prerequisites
^^^^^^^^^^^^^^^^

|SPN| supports |Intel McLaren Island Reference Design Board|. To start developing |SPN|, the following equipment, software and environments are required:

* |Intel McLaren Island Reference Design Board|
* DediProg SF600 programmer
* Linux host or windows host (see :ref:`running-on-linux` or :ref:`running-on-windows` for details)
* Internet access

.. |Intel McLaren Island Reference Design Board| raw:: html

   <a href="https://www.seavo.com/en/products/products-info_itemid_558.html" target="_blank">Intel McLaren Island Reference Design Board</a>

.. |instructions| raw:: html

   <a href="https://wiki.up-community.org/BIOS_chip_flashing_on_UP_Squared" target="_blank">instructions</a>

Board Setup
^^^^^^^^^^^^^^^^^

.. image:: /images/mcl-setup.jpg
   :width: 600
   :alt: |MTL-MCL| Board Setup
   :align: center


Before You Start
^^^^^^^^^^^^^^^^^

.. warning:: As you plan to reprogram the SPI flash, it's a good idea to backup the pre-installed BIOS image first.


Boot the board and enter BIOS setup menu to get familiar with the board features and settings.

Debug UART
^^^^^^^^^^^

For |MTL| platforms, serial port connector location can be found from the above table for each supported target board.

.. note:: Configure host PuTTY or minicom to 115200bps, 8N1, no hardware flow control.

Building
^^^^^^^^^^

|Intel McLaren Island Reference Design Board| is based on Intel |MTL|. To build::

    python BuildLoader.py build mtl

The output images are generated under ``Outputs`` directory.

Stitching
^^^^^^^^^^

Stitch |SPN| images with factory BIOS image using the stitch tool::

    python Platform/MeteorlakeBoardPkg/Script/StitchLoader.py -i <BIOS_IMAGE_NAME> -s Outputs/mtl/SlimBootloader.bin -o <SBL_IFWI_IMAGE_NAME> -p 0xAA00001F

    <BIOS_IMAGE>     : Input file. Factory BIOS extracted from McLaren Island board.
    <SBL_IFWI_IMAGE> : Output file. New IFWI image with SBL in BIOS region.
    -p <value>       : 4-byte platform data for platform ID (e.g. 1F) and debug UART port index (e.g. 00).

.. Note:: StitchLoader.py script works only if Boot Guard in the base image is not enabled, and the silicon is not fused with Boot Guard enabled.
          If Boot Guard is enabled, please use StitchIfwi.py script instead.

See :ref:`stitch-tool` on how to stitch the IFWI image with |SPN|.


Slimbootloader binary for capsule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
A capsule image could ecapsulate a Slimbootloader image to be used in firmware update mechanism. More information is described in :ref:`firmware-update`.

For the process running properly, an IFWI image programmed on board with TOP SWAP size configuration is required.

.. note:: Enabling TOP SWAP size configuration requires additional firmware components and tools.

Creating Slimbootloader binary for capsule image requires the following steps:

Build |SPN| for |UPX12RPLP|::

  python BuildLoader.py build rplp

Edit the 4-byte platform data in Slimbootloader image by *Hexedit* or equivalent hexdecimal editor.

Go to top of TOP SWAP A at address 0xCFFFF4, edit ``14 01 00 AA`` as below
::

  00CFFFF0   90 90 EB B9  14 01 00 AA  A4 50 FF FF  00 50 FF FF  ................

Go to top of TOP SWAP B at address 0xC7FFF4, edit ``14 01 00 AA`` as below
::

  00C7FFF0   90 90 EB B9  14 01 00 AA  A4 50 FF FF  00 50 FF FF  ................

For more details on TOP SWAP regions, please refer :ref:`flash-layout`

Generate capsule update image ``FwuImage.bin``::

  python BootloaderCorePkg/Tools/GenCapsuleFirmware.py -p BIOS Outputs/rplp/SlimBootloader.bin -k KEY_ID_FIRMWAREUPDATE_RSA3072 -o FwuImage.bin

For more details on generating capsule image, please refer :ref:`generate-capsule`.

Triggering Firmware Update
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Please refer to :ref:`firmware-update` on how to trigger firmware update flow.
Below is an example:

To trigger firmware update in |SPN| shell:

1. Copy ``FwuImage.bin`` into root directory on FAT partition of a USB key

2. Boot and press any key to enter |SPN| shell

3. Type command ``fwupdate`` from shell

   Observe |SPN| resets the platform and performs update flow. It resets *multiple* times to complete the update process.

Flashing
^^^^^^^^^

Flash the generated SBL_IFWI_IMAGE_NAME to the target board using a DediProg SF600 programmer.

.. note:: Refer the table above to identify the connector on the target board for SPI flash programmer. When using such device, please ensure:


    #. The alignment/polarity when connecting Dediprog to the board. 
    #. The power to the board is turned **off** while the programmer is connected (even when not in use).
    #. The programmer is set to update the flash from offset 0x0.


**Good Luck!**

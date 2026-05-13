Requirements
------------

The requirements listed below are the ones used for the development of ``pibooth``,
but other configuration may work fine. **All hardware buttons, lights and printer
are optional**, the application can be entirely controlled using a keyboard, a
mouse or a touchscreen.

.. warning:: Using a Pi Camera, the preview is visible only on a screen connected
             to the HDMI or DSI connectors (the preview is an overlay managed at
             GPU low level). It also means that ``pibooth`` can not be started
             throught SSH tuneling. Even with X11 forwarding enabled (``ssh -X ...``)
             the preview will not be visible.

Hardware
^^^^^^^^

* 1 Raspberry Pi 3 Model B (or higher)
* 1 Camera (Raspberry Pi Camera v2.1 8 MP 1080p
  or any DSLR camera `compatible with gPhoto2 <http://www.gphoto.org/proj/libgphoto2/support.php>`_
  or any webcam `compatible with OpenCV <https://opencv.org>`_ )
* 2 push buttons
* 2 LEDs
* 2 resistors of 100 Ohm
* 1 printer

Software
^^^^^^^^

* **Debian 13 (trixie)** or Raspberry Pi OS based on it (64-bit desktop image recommended), or another current Linux with Python 3.11 or newer
* Python **3.13** (the default ``python3`` on trixie; 3.11+ is supported)
* **Pillow** 12 or newer (installed automatically with ``pip``)
* **pygame** 2.2+ with SDL2 (``libsdl2-2.0-0`` and ``sudo apt install libsdl2-*``)
* On a **Raspberry Pi** with GPIO buttons or LEDs: ``sudo apt install python3-lgpio`` so **gpiozero** 2.x can use the default pin factory (Pi 5 and trixie do not use ``RPi.GPIO``)
* **Raspberry Pi Camera** on current OS is normally used via **OpenCV** (step 7); the legacy ``picamera`` module is optional (``pip install 'pibooth[legacy_picamera]'``) and only applies to old MMAL-based systems
* **libgphoto2** from trixie (or newer) if you use a DSLR; optional **CUPS** / ``libcups2-dev`` for building printing extras


Install
-------

Here is a brief description on how to set-up a Raspberry Pi to use this software.

If you intend to develop on ``pibooth``, an editable/customizable version can be
installed. Instead of doing step 8. of the below procedure, follow
:ref:`instructions here<install_developing_version>`.

Manual procedure
^^^^^^^^^^^^^^^^

1. Download a current `Raspberry Pi OS <https://www.raspberrypi.com/software/>`_ (or Debian) image and set up an SD card. You can follow
   `these instructions <https://www.raspberrypi.com/documentation/computers/getting-started.html>`_.

2. Insert the SD-card into the Raspberry Pi and fire it up. Use the
   ``raspi-config`` tool to configure your system (e.g., expand partition,
   change hostname, password, enable SSH, configure to boot into GUI, etc.).

   .. hint:: Don't forget to enable the camera in raspi-config.

3. Upgrade all installed software:

   .. code-block:: bash

        sudo apt-get update
        sudo apt-get full-upgrade

4. Install SDL2 (and extras) which is required by ``pygame`` 2.x:

   .. code-block:: bash

        sudo apt-get install libsdl2-*

   On a **Raspberry Pi** where you use GPIO (buttons, LEDs), also install the
   default pin driver for **gpiozero** 2.x:

   .. code-block:: bash

        sudo apt install python3-lgpio

5. Optionally install or upgrade ``gPhoto2`` (required only for a DSLR camera).
   On Debian trixie, ``sudo apt install gphoto2`` is often enough; a third-party
   updater script can install a newer upstream build if your camera needs it:

   .. code-block:: bash

        wget https://raw.githubusercontent.com/gonzalo/gphoto2-updater/master/gphoto2-updater.sh
        wget https://raw.githubusercontent.com/gonzalo/gphoto2-updater/master/.env
        chmod +x gphoto2-updater.sh
        sudo ./gphoto2-updater.sh

6. Optionally install ``CUPS`` to handle printers (more instructions to add a
   new printer can be found `here <https://www.howtogeek.com/169679/how-to-add-a-printer-to-your-raspberry-pi-or-other-linux-computer>`_
   ):

   .. code-block:: bash

        sudo apt-get install cups libcups2-dev

7. Optionally install ``OpenCV`` to improve images generation efficiency or if a
   Webcam is used:

   .. code-block:: bash

        sudo apt-get install python3-opencv

8. Install ``pibooth`` from the `PyPI repository <https://pypi.org/project/pibooth/>`_.
   On Debian trixie, system Python is *externally managed*, so use a virtual
   environment (recommended) or pass ``--break-system-packages`` only if you
   accept modifying the OS Python.

   .. code-block:: bash

        python3 -m venv ~/pibooth-venv
        source ~/pibooth-venv/bin/activate
        python3 -m pip install --upgrade pip
        python3 -m pip install pibooth[dslr,printer]

   .. hint:: If you don't have ``gPhoto2`` and/or ``CUPS`` installed (steps 5. and/
          or 6. skipped), remove **printer** and/or **dslr** under the ``[]``.

          If you only use a DSLR (printer / step 6 skipped):

          ``python3 -m pip install pibooth[dslr]``

          If you only use the printer (DSLR / step 5 skipped):

          ``python3 -m pip install pibooth[printer]``

          ``python3 -m pip install pibooth`` installs ``pibooth`` without those extras.

          Other optional extras: ``legacy_picamera`` (obsolete MMAL Pi Camera stack
          only; not used on trixie / libcamera), ``rpi_gpio`` (``RPi.GPIO`` pin
          factory for gpiozero on older Raspberry Pi OS, not Pi 5).

Automated procedure
^^^^^^^^^^^^^^^^^^^

Alternatively, you can use Ansible to install pibooth automatically.
`A playbook can be found here <https://github.com/TiJof/pibooth_ansible>`_
(thank you **TiJof**).


Circuit diagram
---------------

Here is the diagram for hardware connections. Please refer to the
:ref:`default configuration file<Default configuration>`.
to know the default pins used (`physical pin numbering <https://pinout.xyz>`_).

.. image:: ../images/sketch.png
   :align: center
   :alt: Electronic sketch

An extra button can be added to start and shutdown properly the Raspberry Pi.
Edit the file ``/boot/config.txt`` and set the line:

.. code-block:: bash

    dtoverlay=gpio-shutdown

Then connect a push button between physical *pin 5* and *pin 6*.

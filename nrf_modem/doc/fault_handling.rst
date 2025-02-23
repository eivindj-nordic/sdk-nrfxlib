.. _fault_handling:

Modem fault handling
####################

The application core and the modem core are separate cores on an nRF9160 SiP.
When the modem core on the nRF9160 SiP crashes, it sends a fault signal to the application core.

Detecting a fault
*****************

The Modem library implements a fault handling mechanism, allowing the application to be notified when the modem core has crashed.
The application can set a modem fault handler function using the Modem library initialization parameters during initialization.
For a complete list of modem fault reasons, see :ref:`nrf_modem_fault`.

In |NCS|, the :ref:`nrf_modem_lib_readme` sets the modem fault handler during Modem library initialization.

Recovering from a fault
***********************

When the modem has crashed, it can be reinitialized by the application.
The application core does not need to be reset to recover the modem from a fault state.
The application can reinitialize the modem by reinitializing the Modem library through calls to :c:func:`nrf_modem_shutdown` and :c:func:`nrf_modem_init` functions.

.. important::

   The reinitialization of the Modem library must be done outside of the Modem library fault handler.

Networking sockets
******************

Socket APIs that require communication with the modem, return ``-1`` and set ``errno`` to ``NRF_ESHUTDOWN`` if the modem has crashed or if it is uninitialized.

Although the modem has crashed, any data which was stored by the Modem library, including data that was delivered to Modem library by the modem, remains available until the Modem library is shut down.
This data includes incoming network data, which was received before the crash but has not been read by the application.

The application can read that data as normal, using the :c:func:`recv` function.
When no more data is available, the :c:func:`recv` returns ``-1`` and sets ``errno`` to ``NRF_ESHUTDOWN``.

Ongoing API calls
*****************

The modem can crash when a function call in Modem library is ongoing, for example, a socket API call.
When the Modem library detects that a fault has occurred in the modem, it immediately exits all calls that are waiting for a modem response.

Following are the two categories of APIs in Modem library:

* Socket APIs - Exit immediately returning ``-1`` and sets errno to ``NRF_ESHUTDOWN``
* Non-Socket APIs - Exit immediately returning ``-NRF_ESHUTDOWN``

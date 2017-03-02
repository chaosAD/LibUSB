Compiling LibUSB-1.0.21
=======================
If compiled using MinGW, the compiler will dump an error stating that storage
size of some variable isn't known. Those variables are declared as
`struct timespec` type. The reason is because the data structure is not
defined under `time.h` header. It is included in `unistd.h` header. So by
including the additional header into the offending C files, the problem
will be solved.

Note: If compiled under MinGW, do not use `Git Bash`. Use `MYS Bash`, instead.
      This is to avoid failure during `configure` or `make`.

Installing PyUSB
================
First install PyUSB through pip if you haven't done so:
```
pip install pyusb
```

Backend USB Library
===================
PyUSB relies on a USB library backend to function. In our case, we will use
`libusb-1.0.dll` obviously. After running:
```
./configure --prefix=/C/Tools
make
make install
```
The DLL can be found in `/C/Tools/bin` folder. Copy it to `/C/Windows/System32`
and another copy to `/C/Windows/SysWOW64`. Note that PyUSB uses
`ctypes.util.find_library()` function to locate the library. If a copy of the
DLL is not copied into `/C/Windows/SysWOW64`, the function will fail to locate
it. I don't know why. Once that is done, open up Python IDLE and type:
```
from ctypes.util import find_library
find_library(r"libusb-1.0.dll")
```
It should return `C:\\Windows\\system32\\libusb-1.0.dll` and you are good to
go. If nothing is displayed, it means it cannot find the library. Re-examine
the steps given above to make sure you have not done any mistake.

Using PyUSB
===========
To use the PyUSB type:
```
import usb.core
dev = usb.core.find()
```
which should return no result. If there is any exception raised, probably the
backend USB library is not there. Please read the previous section.

The following example is for ST-Link dongle:
```
import usb.core
dev = usb.core.find(idVendor=0x0483, idProduct=0x3748)
dev.set_configuration()
cfg = dev.get_active_configuration()          # Get configuration descriptor
intf = cfg[(0,0)]                             # Get the 1st interface descriptor
ep = usb.util.find_descriptor(                # Get the 1st OUT endpoint descriptor
  intf,
  # match the first OUT endpoint
  custom_match = \
  lambda e: \
      usb.util.endpoint_direction(e.bEndpointAddress) == \
      usb.util.ENDPOINT_OUT) print(ep)       # Print the OUT end-point
print(intf)                                  # Print the interface
print(cfg)                                   # Print the configuration

usb.core.util.get_string(dev, 1)             # Returns 'STMicroelectronics'
usb.core.util.get_string(dev, 2)             # Returns 'STM32 STLink'
usb.core.util.get_string(dev, 4)             # Returns 'ST Link'
```
We can show all USB devices:
```
usb.core.show_devices()
```
When there are multiple ST-Link dongles, pyusb is able to list each one of them.
They are differentiated by `bus` and `address`:
```
usb.core.show_devices(idVendor=0x483, idProduct=0x3748)
```
If the details of the devices are also needed, type:
```
usb.core.show_devices(verbose=True, idVendor=0x483, idProduct=0x3748)
```
The following code snippet shows another way to find all devices on the USB bus
with the same `idVendor` and `idProduct` (identical devices):
```
devices = usb.core.find(find_all = True, idVendor=0x483, idProduct=0x3748)
for device in devices:
	print("device=%x:%x, bus=%d, address=%d" % \
	      (device.idVendor, device.idProduct, device.bus, device.address))
```
For further examples, refer to [1].

Source Code
===========
This source code was retrieved from [2].


References
==========
1.   https://github.com/walac/pyusb/blob/master/docs/tutorial.rst
2.   https://sourceforge.net/projects/libusb/files/libusb-1.0/libusb-1.0.21/libusb-1.0.21.tar.bz2/download
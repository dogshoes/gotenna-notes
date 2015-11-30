## The goTenna

The goTenna is a neat little device which permits peer to peer communication over distances without the need for an established wireless backbone (cell phone network) or radio license.  Kind of neat.  Here are some of my notes on the device.

## Connecting

**Connecting to your goTenna outside the official app will probably brick your unit.  It might cause it to launch off your desk and off in to space.  Who knows.  Read this document at your own risk.**

Connect the goTenna over USB to any desktop computer and it shows up as a "goTenna" (swell!), and exposes a serial port over USB.  The driver seems to load in Windows 10, but not Windows 7.  Linux is happy to load a generic ACM driver (like TTY but for modems.)  Connecting to this serial port at 9600 baud gives us an undocumented CLI.  Neat-o.  I can't determine whether this mode is purely diagnostic or whether it might be possible to fully control the goTenna.

Periodically send a message over the serial port or else you'll get this message and the serial port will drop!

```
[1085000-18984] USB Asleep
```

Send a CR to get a `goTenna>` prompt.  You don't need to be at a `goTenna>` prompt to enter a command.

### The Periodic Event

Every minute (?) goTenna outputs diagnostic information about the current state.

```
[984000-40984] Periodic Event
[984000-000] Batt Voltage : 3981, selected pap: 6
[984015-015] scan rssi[0=81  1=76  2=0  3=91  4=75]
[984016-001] MAC bg_rssi=73
```

Battery voltage is what it says on the tin.  PAP appears to be "PA Power", refer to the `pap` command documented below.  [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indication) is a pretty familiar RF concept.

### CLI Behavior

The CLI seems to accept some valid commands with any extraneous trailing characters.  For example, the `version` command can be invoked by `versionxy`, or `version;`, or `version;temp;`.

### Version Command

Show the goTenna firmware version.

```
[095843] goTenna> version
[121435] goTenna> 
[121445-39429] cli_parse_command >> version
[121445-000] FW Version : 00.12.02
[121445-000] Build Date : Oct 22 2015 13:05:48

[121445-000] Boot Version: 01.07
[121445-000] printer: cli-cmd = version
```

### RSSI Command

Get information about the RSSI.

```
[165182] goTenna> rssi
[166444] goTenna>
[166454-2438] cli_parse_command >> rssi
[166454-000] CLI cmd=RSSI
[166454-000] printer: cli-cmd = rssi
[166475-021] log received.
[166490-015] RSSI :
Channel 0 RSSI 85
Channel 1 RSSI 70
Channel 2 RSSI 83
Channel 3 RSSI 80
Channel 4 RSSI 82
```

### PAP Command

Set the PA Power (PAP).  Not sure what this is, or whether it's a great idea to change it.  However, changing it doesn't seem to change the "selected pap" value that appears in the periodic event. 

```
[364773] goTenna> pap 2
[366776] goTenna> 
[366786-2745] cli_parse_command >> pap 2
[366786-000] PA Power value changed to 2
[366786-000] printer: cli-cmd = pap 2
```

### Batt Command

Get information about the battery.

```
[541125] goTenna> batt
[544393] goTenna>
[544404-4274] cli_parse_command >> batt
[544405-001] Raw Value :40627 Battery= 4.091
[544405-000] printer: cli-cmd = batt
```

### Flash Command

Perform a flash check?  I'm a bit afraid of this command.

```
[650554] goTenna> flash
[651526] goTenna> 
[651536-21415] cli_parse_command >> flash
[651536-000] Flash check passed.
[651536-000] printer: cli-cmd = flash
```

### Power Command

Seems to immediately reboot the device.  Does not return any output.  Seems to disregard arguments.

```
[946820] goTenna> power
[948519] goTenna>
```

### Log Command

Not at all sure what this does.

```
[804819] goTenna> log
[806217] goTenna> 
[806227-2421] cli_parse_command >> log
[806227-000] printer: cli-cmd = log
[806237-010] log received.
```

### Temp Command

Show the temperature of some internal component (NXP processor?).

```
[014864] goTenna> temp
[022266] goTenna>
[022276-21064] cli_parse_command >> temp
[022296-020] Temp raw: e
[022296-000] Temp: 14
[022296-000] printer: cli-cmd = temp
```

```
[193855] goTenna> temp
[194621] goTenna>
[194631-30615] cli_parse_command >> temp
[194651-020] Temp raw: 11
[194651-000] Temp: 17
[194651-000] printer: cli-cmd = temp
```

### Res Command

Reset RF hardware?  The [Si4460](https://www.silabs.com/Support%20Documents/TechnicalDocs/Si4464-63-61-60.pdf) is a High-Performance, Low-Current Tranceiver by Silicon Labs.

```
[072043] goTenna> res
[072690] goTenna>
[072701-1150] cli_parse_command >> res
[072701-000] printer: cli-cmd = res
[072711-010] TRX Current State: STATE_POWER_UP *
[073277-566] Si4460 initialized.
[073284-007] Batt Voltage : 4163, selected pap: 6
[073284-000] Batt Voltage : 4165, selected pap: 6
[073300-016] scan rssi[0=88  1=78  2=87  3=93  4=71]
[073300-000] MAC bg_rssi=42
[073300-000] MAC : STATE_POWER_UP*
[073300-000] Reset Cause : 0, 4
[073301-001] MAC Current State: Idle*
[073301-000] Batt Voltage : 4164, selected pap: 6
[073316-015] scan rssi[0=87  1=89  2=86  3=80  4=87]
[073317-001] MAC bg_rssi=44
[073317-000] TRX Current State: STATE_POWER_UP *
[073883-566] Si4460 initialized.
[073890-007] Batt Voltage : 4165, selected pap: 6
```

### Additional Notes and Things to Look at

This is a pile of things to look at later.

This appears at boot.

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19XXX0, Wrt Msg Add: 19XXX8, Del Msg Num: 0, Total Msg Nu1
[000095-00
```

How can we read this diag block?  Is there wear leveling in place or does this block just get constantly overwritten?  Diag block is always 433 and 434 on my goTenna.

```

Write Diag Block, 433 CRC: 56364

Write Diag Block, 434 CRC: 56364
[330141-2125] Storing Diag Info in Flash
```

Issuing a command of `?` displays some interesting bits.  It appears as though this is *part* of another command, or chain of commands?  This command *does not* disregard extraneous trailing characters.

Perhaps this simply displays network stats.

```
[511539] goTenna> ?
[512308] goTenna>
[512319-20303] cli_parse_command >> ?
[512319-000] res->       Reset network SM IDLE.
[512319-000] ctx {0/1}->         Continuous transmit(en=1).
[512319-000] cfp {0/1}->         Corrupt First Packet(en=1).
[512319-000] cap {0/1}->         Corrupt All Packets(en=1).
[512320-001] printer: cli-cmd = ?
```

This appeared once after issuing `?`, but I haven't been able to make it appear again.  Perhaps unrelated and caused by noise on the radio?  Nobody near by has a goTenna.

```
[523195] INT PREAMBLE_DETECT
[523195-10875] Preamble detect1
[523196-001] TRX Current State: RX_SYNC_WORD_WAIT *
[523196-000] MAC Current State: MAC_RX_PRE_RTS*
[523376-180] TRX State Expired: RX_SYNC_WORD_WAIT *
[523377-001] TRX channel id : 4
[523378-001] TRX Current State: PDU_IDLE_LONG *
[523378-000] turning off the HW
[523378-000] PA ------------ off
[523399-021] MAC Receive PreRTS Timeout expired
[523399-000] MAC Current State: Idle*
```

## USB

### Output From `lsusb`

```
Bus 002 Device 004: ID 1fc9:2047 NXP Semiconductors 
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            2 Communications
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x1fc9 NXP Semiconductors
  idProduct          0x2047 
  bcdDevice            3.10
  iManufacturer           1 goTenna, Inc
  iProduct                2 goTenna
  iSerial                 3 123456789ABCDEF
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           67
    bNumInterfaces          2
    bConfigurationValue     1
    iConfiguration          4 123456789ABCDEF
    bmAttributes         0xc0
      Self Powered
    MaxPower               64mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         2 Communications
      bInterfaceSubClass      2 Abstract (modem)
      bInterfaceProtocol      1 AT-commands (v.25ter)
      iInterface              0 
      CDC Header:
        bcdCDC               1.10
      CDC Union:
        bMasterInterface        0
        bSlaveInterface         1 
      CDC ACM:
        bmCapabilities       0x00
      CDC Call Management:
        bmCapabilities       0x01
          call management
        bDataInterface          1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval             128
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass        10 CDC Data
      bInterfaceSubClass      0 Unused
      bInterfaceProtocol      0 
      iInterface              0 
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x01  EP 1 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
Device Status:     0x0000
  (Bus Powered)
```

### Output From `dmesg`

```
[125726.694882] usb 2-2.1: new full-speed USB device number 6 using uhci_hcd
[125726.826823] usb 2-2.1: New USB device found, idVendor=1fc9, idProduct=2047
[125726.826826] usb 2-2.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[125726.826828] usb 2-2.1: Product: goTenna
[125726.826830] usb 2-2.1: Manufacturer: goTenna, Inc
[125726.826831] usb 2-2.1: SerialNumber: 123456789ABCDEF
```

## Possible Leads

* [http://tieba.baidu.com/p/3858941725]
* [https://github.com/akimtke/akim-gotenna/blob/04223025e1a93c9e518d54000696509abdedcc85/src/tests/wireless.py]
## The goTenna

The goTenna is a neat little device which permits peer to peer communication over distances without the need for an established wireless backbone (cell phone network) or radio license.  Kind of neat.  Here are some of my notes on the device.

## Connecting

**Connecting to your goTenna outside the official app will probably brick your unit.  It might cause it to launch off your desk and off in to space.  Who knows.  Read this document at your own risk.**

Connect the goTenna over USB to any desktop computer and it shows up as a "goTenna" (swell!), and exposes a serial port over USB.  The driver seems to load in Windows 10, but not Windows 7.  Linux is happy to load a generic ACM driver (like TTY but for modems.)  Connecting to this serial port at 9600 baud gives us an undocumented CLI.  I can't yet determine whether this mode is purely diagnostic or whether it might be possible to fully control the goTenna.

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

The CLI seems to accept some valid commands with any extraneous trailing characters.  For example, the `version` command can be invoked by `versionxy`, or `version;`, or `version;temp;`.  Some commands will default given no arguments (`pap` for example), but some commands will report as unrecongnized unless an argument is passed (`chn` and `ctx` to name a few).

As of the latest firmware release, 0.23.2, the `goTenna>` prompt is no longer displayed.

### Version Command

Show the goTenna firmware version.  This version appears different from what the goTenna app reports.

| Version displayed in CLI | Version displayed in app |
|--------------------------|--------------------------|
| 00.12.02                 | 00.18.02                 |
| 00.15.04                 | 00.21.04                 |
| 00.17.02                 | 0.23.2                   |

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

Get the [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indication) of the five [MURS](https://en.wikipedia.org/wiki/Multi-Use_Radio_Service) channels used for communication.

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

Set the PA Power (PAP).  Not sure what this is, or whether it's a great idea to change it.  However, changing it doesn't seem to change the "selected pap" value that appears in the periodic event.   The largest PAP that can be entered is 255.  Larger values will be truncated to 255.

```
[364773] goTenna> pap 2
[366776] goTenna> 
[366786-2745] cli_parse_command >> pap 2
[366786-000] PA Power value changed to 2
[366786-000] printer: cli-cmd = pap 2
```

### Batt Command

Get information about the battery.  Issuing the command as `battery` works just as well, but this could be due to the cli ignoring extraneous trailing characters.  The values this command gives are sometimes wonky, so several readings should be taken any outliers should be discarded.

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

Seems to immediately reboot the device.  Does not return any output.  Your serial connection will drop as the USB device disconnects and reconnects.

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

Read the internal temperature.  Could be the temperature of the Freescale ARM processor or the Silicon Labs RF chip, not entirely sure yet.  Given the preoccupation with the TRX (tranceiver?) temperature during `send` I would imagine this is temperature of the Silicon Labs RF chip.

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

Reset RF hardware?  The [Si4460](https://www.silabs.com/Support%20Documents/TechnicalDocs/Si4464-63-61-60.pdf), mentioned in the resulting output, is a High-Performance Low-Current Tranceiver by Silicon Labs.

Interesting to note, this command is sesensitive to trailing extraneous characters.

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

### ? (Help) Command

Issuing a `?` to the CLI returns some help.

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

### Ctx Command

Enable continuous transmission mode.

```
[1477193] goTenna> ctx 1
[1480391] goTenna>
[1480402-4386] cli_parse_command >> ctx 1
[1480402-000] printer: cli-cmd = ctx 1
[1480413-011] Continuous Transmission Enabled.
[1480421-008] PA ------------on
[1480421-000] vapc value  3400
```

Disable continuous transmission mode.

```
[024889] goTenna> ctx 0
[027132] goTenna>
[027143-4549] cli_parse_command >> ctx 0
[027143-000] printer: cli-cmd = ctx 0
[027154-011] PA ------------ off
[027720-566] Si4460 initialized.
[027727-007] Continuous Transmission Disabled.
```

### Cfp Command

Enable first packet corruption.  Used for testing?

```
[056441] goTenna> cfp 1
[058044] goTenna>
[058054-27934] cli_parse_command >> cfp 1
[058054-000] printer: cli-cmd = cfp 1
[058065-011] First Packet Corruption Enabled.
```

Disable first packet corruption.

```
[059257] goTenna> cfp 0
[060669] goTenna>
[060679-2614] cli_parse_command >> cfp 0
[060679-000] printer: cli-cmd = cfp 0
[060690-011] First Packet Corruption Disabled.
```

### Cap Command

Enable all packet corruption.

```
[133133] goTenna> cap 1
[134233] goTenna>
[134243-73553] cli_parse_command >> cap 1
[134243-000] printer: cli-cmd = cap 1
[134254-011] All Packet Corruption Enabled.
```

Disable all packet corruption.

```
[134750] goTenna> cap 0
[135746] goTenna> 
[135756-1502] cli_parse_command >> cap 0
[135756-000] printer: cli-cmd = cap 0
[135767-011] All Packet Corruption Disabled.
```

### Chn Command

Select a channel.  The maximum channel that can be selected is 2147483647 (32 bits).  However, during `send` it appears as though the channel is handled as only 8 bits and wraps around, implying maximum of 256 channels.

```
[188229] goTenna> chn 1
[189336] goTenna>
[189347-1726] cli_parse_command >> chn 1
[189347-000] printer: cli-cmd = chn 1
[189358-011] Channel 1 selected.
```

### Send Command 

Send... something?  Uses previously set values from `vapc`, `chn`, `pap` commands.  Unsure how to set the payload, destination, or anything else about what's being sent.

```
[342917] goTenna> send
[343867] goTenna>
[343877-13038] cli_parse_command >> send
[343877-000] printer: cli-cmd = send
[343888-011] Sending PN9 packets
[344454-566] Si4460 initialized.
[344462-008] Before Temp
[344472-010] TRX *** Temp AD Value= 0x5ea0 ***
[344472-000] TRX Temp 17
[344473-001] TRX valid temp
[344473-000] Sending PN9 on chan 0, at vapc adc 3400 and pap 1
[344473-000] PA ------------on
[344474-001] vapc value  3400[344731] INT PACKET_SENT
[344775-301] PA ------------ off
[344775-000] After Temp
[344785-010] TRX *** Temp AD Value= 0x5dbd ***
[344785-000] TRX Temp 29
[344785-000] TRX valid temp
[344786-001] TRX Current State: STATE_POWER_UP *
[345301] INT Sync Word
[345352-566] Si4460 initialized.
[345359-007] Batt Voltage : 4131, selected pap: 6
[345359-000] Batt Voltage : 4131, selected pap: 6
[345375-016] scan rssi[0=98  1=82  2=91  3=96  4=87]
[345375-000] MAC bg_rssi=42
[345375-000] MAC : STATE_POWER_UP*
[345375-000] Reset Cause : 82, 0
[345376-001] MAC Current State: Idle*
[345376-000] Batt Voltage : 4134, selected pap: 6
[345391-015] scan rssi[0=93  1=81  2=91  3=94  4=77]
[345392-001] MAC bg_rssi=45
[345392-000] TRX Current State: STATE_POWER_UP *
[345958-566] Si4460 initialized.
[345965-007] Batt Voltage : 4132, selected pap: 6
```

### Vapc Command

Set the VAPC?  Don't set this.  Leave it at 3400?

```
[484758] goTenna> vapc 3400
[486969] goTenna>
[486980-35964] cli_parse_command >> vapc 3400
[486980-000] vapc value changed
[486980-000] printer: cli-cmd = vapc 3400
```

### Read_eflash command

Output the contents of the internal flash chip over the serial port.  I've created some tools to aid with extracting the flash this way: https://github.com/dogshoes/gotenna-flash-dumper.

```
[1359491] goTenna> read_eflash
[1361976] goTenna> 
000000: 3A1008000000600020C10800004D2A01 
000010: 00512A0100AB0A3A10081000552A0100 
000020: 592A01005D2A0100612A0100C00A3A10 
000030: 082000652A0100692A01006D2A010071 
...
```

### The Button

There's a tiny button near the USB port on the goTenna.  A single press results in the following message.

```
[038760] BUTTON RELEASED
[039378] BUTTON PRESSED
```

Holding the button down for over 10 seconds does this!  Does this reset the data stored in flash?

```
[051174-10159] Block: 433, Invalid CRC, Expected:29739, IS:-1
[051181-007] Block: 434, Invalid CRC, Expected:29739, IS:-1
[051181-000] Diag Block corruption. Erasing Diag Block.
Write Diag Block, 433 CRC: 0

Write Diag Block, 434 CRC: 0
[051309-128] Block: 433, Invalid CRC, Expected:10804, IS:0
[052028] BUTTON RELEASED
[056970-5661] Binary Log deleted, 1
[056970-000] Current Write Index, 0222
[056971-001] Current Write Offset, 0032
[056971-000] Current Read Index, 0222
[056971-000] Current Read Offset, 0032
[056971-000] Device Data Reset successful
```

Additionally, for extra excitement, there's a red LED which illuminates while the device is charging.

### Additional Notes and Things to Look at

This is a pile of things to look at later.

This appears at boot.

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19XXX0, Wrt Msg Add: 19XXX8, Del Msg Num: 0, Total Msg Num: 251. Max Msg: 261
[000095-00
```

This also appears at boot, sometimes?

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19a000, Wrt Msg Add: 19a000, Del Msg Num: 0, Total Msg Nu1
[000095-002] PAP Offset 0
[000095-000] SERIAL no: MX1542XXXX[000096-001]
[000096-000]
Bluetooth flag 255
[000107-011] Flash Initialized.
[000598-491] Si4460 initialized.
[000605-007] Batt Voltage : 4157, selected pap: 6
[000605-000] Batt Voltage : 4152, selected pap: 6
[000621-016] scan rssi[0=44  1=44  2=35  3=44  4=44]
[000621-000] MAC bg_rssi=39
[000621-000] MAC : STATE_POWER_UP*
[000621-000] Reset Cause : 82, 0
[000622-001] MAC Current State: Idle*
[000622-000] Batt Voltage : 4153, selected pap: 6
[000637-015] scan rssi[0=47  1=40  2=35  3=47  4=40]
[000637-000] MAC bg_rssi=39
[000638-001] TRX Current State: STATE_POWER_UP *
[000639-001] Flash Message invalid Metadata
[001204-565] Si4460 initialized.
[001211-007] Batt Voltage : 4155, selected pap: 6
[001212-001] TRX channel id waiting idle: 4
```

How can we read this diag block?  Is there wear leveling in place or does this block just get constantly overwritten?  Diag block is always 433 and 434 on my goTenna.

```

Write Diag Block, 433 CRC: 56364

Write Diag Block, 434 CRC: 56364
[330141-2125] Storing Diag Info in Flash
```

Had this appear.

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

Happened again?

```
[539606] INT PREAMBLE_DETECT
[539607-6591] Preamble detect1
[539608-001] TRX Current State: RX_SYNC_WORD_WAIT *
[539608-000] MAC Current State: MAC_RX_PRE_RTS*
[539611] INT Sync Word
[539611-003] TRX Sync Word detected. *
[539612-001] MODEM STATUS : 00 01 56 58 00 00 00
[539612-000] MAC RX_Pre-RTS stage=0, sync detect received
[539613-001] TRX Current State: RX_PDU_PKT_HEADER_WAIT *
[539671] INT RX_FIFO_ALMOST_FULL
[539672-059] RX fifo len 18/n
[539672-000] crc is being checked
[539673-001] crc is corrupted
[539681-008] crc packet not recovered
[539682-001] TRX channel id : 4
[539684-002] TRX Current State: PDU_IDLE_LONG *
[539684-000] turning off the HW
[539684-000] PA ------------ off
[539694-010] MAC Receive PreRTS Timeout expired
[539694-000] MAC Current State: Idle*
[539695-001] Batt Voltage : 4154, selected pap: 6
[539710-015] scan rssi[0=89  1=84  2=90  3=90  4=86]
[539710-000] MAC bg_rssi=55
[539711-001] TRX channel id : 4
[539712-001] TRX Current State: PDU_IDLE_LONG *
[539713-001] turning off the HW
[539713-000] PA ------------ off
```

This occurs when pairing the goTenna.

```
BLE set to connected
[026584-8392] Bluetooth Response Sent.
[026703-119] NRF Message sent
[026734-031] system info asked.
[026744-010] TRX *** Temp AD Value= 0x5efd ***
[026744-000] TRX Temp 12
[026745-001] TRX valid temp
[026789-044] Bluetooth Response Sent.
[026943-154] NRF Message sent
[027014-071] Bluetooth Response Sent.
[027094-080] NRF Message sent
[027123-029] Bluetooth Response Sent.
[027274-151] NRF Message sent
[027303-029] Received get stored message.
[027304-001] Get message NACK sent.
[027304-000] Bluetooth Response Sent.
[027424-120] NRF Message sent
[027574-150]  Received date and time
[027575-001] Bluetooth Response Sent.
[027693-118] NRF Message sent
```

On setting the GID.

```
[156336-33320] stored hash17920
[156338-002] Bluetooth Response Sent.
[156455-117] NRF Message sent
```

This happened after.

```
[175686-11670]  Received date and time
[175687-001] Bluetooth Response Sent.
[175805-118] NRF Message sent
```

Beginning firmware upgrade.

```
[083825-151]  Received date and time
[083825-000] Bluetooth Response Sent.
[083944-119] NRF Message sent
[083974-030] FW Exchange initiated.
[088346-4372] Bluetooth Response Sent.
[088384-038] NRF Message sent
[089225-841] FW data packet received
[089226-001] seqid 36[089226-000] Bluetooth Response Sent.
[089344-118] NRF Message sent
[090155-811] FW data packet received
[090156-001] seqid 37[090156-000] stored pkt 1 of 655 stored length 250 of total length 163652
[090167-011] Bluetooth Response Sent.
[090274-107] NRF Message sent
[091085-811] FW data packet received
[091086-001] seqid 38[091086-000] stored pkt 2 of 655 stored length 500 of total length 163652
[091098-012] Bluetooth Response Sent.
[091205-107] NRF Message sent
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

The serial number here has not been obfuscated.  Every goTenna I've tried reports this serial number.

```
[125726.694882] usb 2-2.1: new full-speed USB device number 6 using uhci_hcd
[125726.826823] usb 2-2.1: New USB device found, idVendor=1fc9, idProduct=2047
[125726.826826] usb 2-2.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[125726.826828] usb 2-2.1: Product: goTenna
[125726.826830] usb 2-2.1: Manufacturer: goTenna, Inc
[125726.826831] usb 2-2.1: SerialNumber: 123456789ABCDEF
```

## Possible Leads

* http://tieba.baidu.com/p/3858941725
* https://github.com/akimtke/akim-gotenna/blob/04223025e1a93c9e518d54000696509abdedcc85/src/tests/wireless.py
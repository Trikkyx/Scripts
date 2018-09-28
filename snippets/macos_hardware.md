# macOS Snippets: Hardware 

The following snippets are used to extract hardware information from a running macOS system.

### Index

* [Serial Number (Computer)](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#serial-number-computer)
* [Serial Number (Logic Board)](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#serial-number-logic-board)
* [MAC Address](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#mac-address)
* [MAC Address (Logic Board)](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#mac-address-logic-board)
* [Battery Percentage](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#battery-percentage)
* [Display Inches](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#display-inches)
* [Board ID](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#board-id)
* [Model Identifier / Machine Model](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#model-identifier--machine-model)
* [RAM Installed](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#ram-installed)
* [Marketing Name](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#marketing-name)
* [Boot ROM Version](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#boot-rom-version)
* [Boot Time/Last Reboot](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#boot-time-last-reboot)
* [Uptime](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#uptime)
* [Virtual Machine](https://github.com/erikberglund/Scripts/blob/master/snippets/macos_hardware.md#virtual-machine)

## Serial Number (Computer)

Serial number for the computer

**BASH**
```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:SSN | awk '{ gsub(/\%.*/, ""); print $NF }'
```

Alternate Method:

```bash
ioreg -d2 -c IOPlatformExpertDevice | awk -F\" '/IOPlatformSerialNumber/ { print $(NF-1) }'
```

Output:

```console
C02*****G8WP
```

## Serial Number (Logic Board)

Serial number for the main logic board (MLB)

**BASH**
```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:MLB | awk '{ print $NF }'
```

Output:

```console
C0252******GF2C1H
```

## MAC Address

MAC address for interface ( using `en0` in the example )

**BASH**
```bash
ifconfig en0 | awk '/ether/{ gsub(":",""); print $2 }'
```

Output:

```console
a45e60******
```

Uppercase output:

**BASH**
```bash
ifconfig en0 | awk '/ether/{ gsub(":",""); print toupper($2) }'
```

Output:

```console
A45E60******
```

## MAC Address (Logic Board)

MAC address for the main logic board (MLB)

**BASH**
```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM | awk '{ gsub(/\%/, ""); print $NF }'
```

Output:

```console
0cbc9f******
```

Uppercase output:

**BASH**
```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM | awk '{ gsub(/\%/, ""); print toupper($NF) }'
```

Output:

```console
0CBC9F******
```

## Battery Percentage

Current battery charge percentage:

**BASH**
```bash
ioreg -rd1 -c AppleSmartBattery | awk '/MaxCapacity/ {max=$NF}; /CurrentCapacity/ {current=$NF} END{OFMT="%.2f%%"; print((current/max) * 100)}'
```

Output:

```console
52,96%
```

## Display Inches

Physical size (in inches) for the internal display

**PYTHON**
```python
#!/usr/bin/python
 
import Quartz
from math import sqrt

# Get online display data
(online_err, displays, num_displays) = Quartz.CGGetOnlineDisplayList(2, None, None)

# Loop through all online displays
for display in displays:

  # Make sure we use the built in display
  if (Quartz.CGDisplayIsBuiltin(display)):

    # Get size of display in mm (returns an NSSize object)
    size = Quartz.CGDisplayScreenSize(display)
    
    # Divide size by 25.4 to get inches
    # Calculate diagonal inches using square root of heigh^2 + width^2
    inch = round(sqrt(pow((size.height / 25.4), 2.0) + pow((size.width / 25.4), 2.0)),1)

    print('Internal Display Inches: ' + str(inch))
```

Output:

```console
Internal Display Inches: 15.4
```

## Board ID

ID for the motherboard

**BASH**
```bash
nvram 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:HW_BID | awk '{ print $NF }'
```

Alternate Method

```bash
ioreg -d2 -c IOPlatformExpertDevice | awk -F\" '/board-id/{ print $(NF-1) }'
```

Output:

```console
Mac-06F11F11946D27C5
```

## Model Identifier / Machine Model

Model Identifier / Machine Model for the computer

**BASH**
```bash
sysctl -n hw.model
```

Output:

```console
MacBookPro11,5
```

## Laptop/Desktop

Check if computer is laptop or desktop

**BASH**
```bash
if [[ $( sysctl -n hw.model ) =~ [Bb]ook ]]; then
	printf "%s" "Laptop"
else
	printf "%s" "Desktop"	
fi
```

Output:

```console
Laptop
```

## RAM Installed

RAM installed (in GB without unit)

**BASH**
```bash
# Get RAM installed in GB
ram_gb=$(( $( sysctl -n hw.memsize ) >> 30 ))

# Print value to stdout
printf "%s\n" "${ram_gb}"
```

Output:

```console
16
```

RAM installed (in MB without unit)

**BASH**
```bash
# Get RAM installed in MB
ram_mb=$(( $( sysctl -n hw.memsize ) >> 20 ))

# Print value to stdout
printf "%s\n" "${ram_mb}"
```

Output:

```console
16384
```

## Marketing Name

**NOTE! Requires an internet connection**

Marketing name for computer

**BASH**
```bash
curl -s http://support-sp.apple.com/sp/product?cc=$( ioreg -c IOPlatformExpertDevice -d 2 | awk -F\" '/IOPlatformSerialNumber/{ sn=$(NF-1); if (length(sn) == 12) count=3; else if (length(sn) == 11) count=2; print substr(sn, length(sn) - count, length(sn)) }' ) | xpath '/root/configCode/text()' 2>/dev/null
```

Output:

```console
MacBook Pro (Retina, 15-inch, Mid 2015)
```

## Boot ROM Version

Return the current boot ROM (EFI) version

**BASH**
```bash
ioreg -p IODeviceTree -n rom@0 -r | awk -F\" '/version/{ print $(NF-1) }'
```

Output:

```console
MBP114.88Z.0172.B10.1610201519
```

## Boot Time/Last Reboot

Get boot time (last reboot) in unix timestamp or as a date string

**BASH**
```bash
# Get boot time from kernel in unix timestamp  
boot_time_unix=$( sysctl -n kern.boottime | awk -F'[^0-9]*' '{ print $2 }' )

# Get boot time from kernel in unix timestamp and convert to date string  
# Using this format as output: "+%F %T %z"  
boot_time_date=$( sysctl -n kern.boottime | awk -F'[^0-9]*' '{ print $2 }' | xargs date -jf "%s" "+%F %T %z" ) 

# Print value to stdout
printf "%s\n" "boot_time_unix=${boot_time_unix}"  
printf "%s\n" "boot_time_date=${boot_time_date}"
```

Output:

```console
boot_time_unix=1508047230
boot_time_date=2017-10-15 08:00:30 +0200
```

## Uptime

Get system uptime in seconds

**BASH**
```bash
# Get boot time from kernel and calculate difference from current time
uptime=$(( $( date +%s ) - $( sysctl -n kern.boottime | awk -F'[^0-9]*' '{ print $2 }' ) ))

# Print value to stdout
printf "%s\n" "${uptime}"
```

Output:

```console
1107
```

## Virtual Machine

Check if system is running in a virtual machine

**BASH**
```bash
if sysctl -n machdep.cpu.features | grep -q "VMM"; then
	printf "%s" "VM"
else
	printf "%s" "Not VM"	
fi
```

Output:

```console
Not VM
```

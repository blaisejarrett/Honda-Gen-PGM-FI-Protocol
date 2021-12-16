# Honda EU7000IS "Data Link Connector"
This project's goal is to understand the Honda EU7000IS "DLC" "Data Link Connector".
The DLC connector is used for diagnostics with Honda's "Dr.H" Diagnostics tool.

Using Honda's Diagnostics tool enables you to query the Generators Serial Number and
stream real time sensors for the following
* Engine Run Time
* Engine Speed
* Battery Voltage
* Cylinder Head Temperature
* Ignition Timing
* Air Temperature
* Barometric Pressure
* Fuel Injection Time
* Throttle Position
* Total Output Power
* Starter Count
* O2 Sensor Voltage
* For both the Master and Slave Inverter:
    * Status
    * Output Voltage
    * Amperage
    * Temperature
    * Eco Mode

Gaining real time access to the above sensor information could enable some 
interesting custom integrations into smart home or power systems.

# Disclaimer
I am not associated with Honda in any way. The EU7000IS generator model has variance between
regions. This research was done with a Canadian 2021 model. Your mileage may vary. Some 
of the information provided could be incorrect.


# Physical layer
The DLC connector is uses a [Sumitomo HM 6187-4441 Connector](https://prd.sws.co.jp/components/en/detail.php?number_s=61874441)

It has 4 pins.

| Pin | Description | Color |
| --- | ----------- | ----- |
| 1   | TX          | Blue/Red |
| 2   | GND         | Green |
| 3   | VBatt       | Brown/White |
| 4   | RX          | Blue/Yellow |


The RX/TX Pins
* Are standard UART Serial
* Use 5v Logic Levels
* Use 9600 Baud with 8 bits no parity 1 stop bit.
* Are both inverted with Idle state low and Active state high.

Messages are sent and received as ASCII Terminal Messages.

A valid TX message (hex) would be:

```
01 43 42 30 30 30 30 30 31 04
```
The first byte of each message is 0x01 (Terminal Start of Frame)
The last byte of each message is 0x04 (Terminal End of Transmission)

The ascii message above reads as
```
CB000001
```

# Understand the Messages

I have found the following unique TX messages.

```
BA00FF03
BC000001
BD000006
BE000007
CA000002
CB000001
CB100000
CB200003
CB300002
```

This is my understanding of the following TX messages and their responses.


# Sensors

`CBxx` is used to query data. xx appears to be the address of the data request. Valid
addresses appear to be
* 00
* 30
* 10
* 20

### ECU information Part 1

`CB000001` is used to query general engine information from the ECU.

In the following examples the top describes the byte number followed by hex
response examples.

Example Reply
```
1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19
CB 00 00 00 18 09 6F 83 00 60 2F 01 03 01 F2 00 06 7E 0B
CB 00 00 00 18 09 64 80 00 64 2F 01 03 01 F2 00 06 7E 7E
         ** ** runtime in minutes.
               ** ** engine RPM
                     ** Battery voltage * 0.1 
                        ** ** Cylinder head C -50
                              ** Ignition Timing Degrees -10
                                                   ** Output voltage?
            
```


### ECU information Part 2
`CB300002` 

Example Reply
```
1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19
CB 30 3C 63 01 81 0D 90 02 1A 00 10 0A 00 00 00 00 00 00
CB 30 3C 63 01 8F 0D 90 02 1A 00 10 0A 00 00 00 00 00 77
** ** * never any change
       * C or D????
         ** Barometric Pressure
            ** ** Fuel injection time * 0x01 in ms
                  ** Throttle position
                        ** ** Output power in VA ????
                              ** ** Starting count
                                    ** O2 sensor * 0.1 volts
```

### Master Inverter
`CB100000` 

All zeros when the gen is not running

Example Reply.
```
0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18
CB 10 00 07 2F 00 00 03 03 00 00 00 00 00 00 00 06 7E 07
CB 10 00 06 2F 00 00 03 03 00 00 00 00 00 00 00 06 7E 06
CB 10 00 14 2F 00 00 03 03 00 00 17 00 00 00 00 06 7E 03
         ** Output amperage * 0.1
            ** Temperature C -40
                        ** Eco mode. 2 == On 3 == Off
                              ** ** Output watts * 10
                                                   ** Output voltage
```

### Slave Inverter
`CB200003`

All zeros when the gen is not running.
```
0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18
CB 20 00 18 31 00 1E 00 00 00 00 00 00 00 00 00 00 00 7C
CB 20 00 19 2F 00 21 00 00 00 00 00 00 00 00 00 00 00 7C
CB 20 00 1A 30 00 20 00 00 00 00 00 00 00 00 00 00 00 72
         ** Output amperage * 0.1
            ** Temperature C -40
               ** ** Output watts * 10
```

Note: All of these captures are done in 120v mode. The slave probably also reports
volts when in 240v mode.

# Keep Alive
`BA00FF03`
Occurs at the start of every connection. Also occurs frequently while idle.
Probably a sort of start communication or keep alive message.

In my case it is always replied with `BA000201`

# Other
`CA000002` is a query for the generators Serial Number.


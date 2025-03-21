# SPI Bus
## Interface
The AC is the SPI master. MHI uses the signals SCK, MOSI and MISO. A slave select signal is not supported, and must be determined from the clock.

Name | Function |input/output
------------ | ------------- |--------------
SCK | serial clock | Clock output from AC
MOSI | Master Out, Slave In | Data output from AC
MISO | Master In, Slave Out |Data input to AC

## Protocol
Clock polarity: CPOL=1 => clock idles at 1, and each cycle consists of a pulse of 0
Clock timing: CPHA=1 => data is captured with the rising clock edge, data changes with the falling edge
## Timing
T<sub>Frame</sub> Time for one frame   
T<sub>FramePause</sub> Time between two frames   
T<sub>Byte</sub> Time for one byte   
T<sub>BytePause</sub> Time between two bytes   
T<sub>Bit</sub> Time for one bit   

![timing diagram](https://user-images.githubusercontent.com/23119513/144753997-48354b1c-e728-49bf-9f56-ea8085d8a741.png)

The following timing is used by SRK xx ZS-S

T<sub>Frame</sub>|T<sub>FramePause</sub>|T<sub>Byte</sub>|T<sub>BytePause</sub>|T<sub>Bit</sub>|T<sub>Clock</sub>
---|---|---|---|---|---|
10ms|40ms|250µs|250µs|31.25µs|32kHz

Other models could have different timing

There are two kinds of frames currently known. Standard frames, which consists of 20 bytes and consumes 20x2x250µs=10ms, and extended frames, which add 13 additional bytes after a short, approximatly 2ms, pause. Between 2 standard frames is a pause of 40ms. For Standard frames, about 20 frames per second will be transmitted. The following oscilloscope screenshot shows 3 bytes:
![grafik](https://user-images.githubusercontent.com/23119513/144753799-f5fc59df-1925-4a60-9a0c-f6675fd9c838.png)

Yellow: SCK; Purple: MOSI
# SPI Frame
A frame starts with three signature bytes. After the header 15 data bytes and 2 bytes for a checksum follow. In the case of an extended frame, 12 further data bytes follow ending with only a single checusm byte. The following table shows the structure of a frame.
In the ESP8266 program code and in the description mainly the short names are used.

raw byte # | long name | short name
---- | ---- | ----
0|signature byte 0| SB0
1|signature byte 1| SB1
2|signature byte 2| SB2
3|data byte 0| DB0
4|data byte 1| DB1
5|data byte 2| DB2
6|data byte 3| DB3
7|data byte 4| DB4
8|data byte 5| DB5
9|data byte 6| DB6
10|data byte 7| DB7
11|data byte 8| DB8
12|data byte 9| DB9
13|data byte 10| DB10
14|data byte 11| DB11
15|data byte 12| DB12
16|data byte 13| DB13
17|data byte 14| DB14
18|checksum byte high| CBH
19|checksum byte low| CBL
20|data byte 15| DB15
21|data byte 16| DB16
22|data byte 17| DB17
23|data byte 18| DB18
24|data byte 19| DB19
25|data byte 20| DB20
26|data byte 21| DB21
27|data byte 22| DB22
28|data byte 23| DB23
29|data byte 24| DB24
30|data byte 25| DB25
31|data byte 26| DB26
32|checksum byte low| CBL2


In the description we differ between

    MOSI frame -> frame send by AC
    MISO frame -> frame send by remote control, e.g. MHI-AC-Ctrl, received by AC.

For the testing and evaluation of the protocol the remote controls [MH-AC-WIFI-1](https://www.intesisbox.com/de/mitsubishi-heavy-ascii-wifi-ac-mh-ac-wmp-1/gateway/) and [RC-E5](https://www.mhi-mth.co.jp/en/products/pdf/pjz012a087b_german.pdf) were used.

## Header
The MOSI frame starts with a header of 3 bytes. First the version byte, indicating whether a standard (0x6C) or extended (0x6D) frame follows. The second and third signature bytes are 0x80, 0x04. See [Mitsubishi AC models](https://github.com/absalom-muc/MHI-AC-Ctrl/issues/6#issue-558530669) for intial discussion on the version byte.
The MISO frame replies with 0xA9 to indicate it understands only Standard frames, or 0xAA for Extended frames. The remaining signature bytes are 0x00, 0x07. 

## Data
The following clauses describe the MOSI/MISO decoding for power, mode, fan, vanes, temperature setpoint and room temperature.
### Power
Power status is coded in MOSI DB0[0].

DB0	| Function
---- | -----
Bit 0| Power
0 | off
1 | on

The same coding is used for setting the Power status. The set bit in the MISO frame is DB0[1].

### Mode
The mode is coded in MOSI DB0[4:2].
<table style="width: 273px; height: 68px;">
<thead>
<tr>
<td style="width: 66.9667px;" colspan="3"><strong>DB0</strong></td>
<td style="width: 66.9667px;"><strong>Function</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td style="width: 66.9667px;">bit 4</td>
<td style="width: 71.4333px;">bit 3</td>
<td style="width: 66.9667px;">bit 2</td>
<td style="width: 66.9667px;">Mode</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">Auto</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">1</td>
<td style="width: 66.9667px;">Dry</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">Cool</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">1</td>
<td style="width: 66.9667px;">Fan</td>
</tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">Heat</td>
</tr>
</tbody>
</table>

The same coding is used for setting the Mode. The set bit in the MISO frame is DB0[5].

### Fan
The fan level is coded in MOSI DB1[2:0].
By default however only fan speeds 1, 2 and 3 are reported. The AC needs to be sent a message with DB1[2] set to acknowledge to the AC extended Fan modes are understood.
<table style="width: 273px; height: 68px;">
<thead>
<tr>
<td style="width: 66.9667px;" colspan="2"><strong>DB1</strong></td>
<td style="width: 66.9667px;"><strong>Function</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td style="width: 66.9667px;">bit 2</td>
<td style="width: 71.4333px;">bit 1</td>
<td style="width: 66.9667px;">bit 0</td>
<td style="width: 66.9667px;">Fan</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">1</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">2</td>
</tr>
<tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">3</td>
</tr>
<tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">0</td>
<td style="width: 66.9667px;">4</td>
</tr>
<tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">1</td>
<td style="width: 66.9667px;">auto</td>
</tr>
</tbody>
</table>

The same coding is used for setting the Fan. The set bit in the MISO frame is DB1[3].

The following table shows the mapping between Fan and IUFANSPEED (opdata) and the IR-RC for Mode=Fan:
Fan | IUFANSPEED | IR-RC
---|---|---
last |0|off
1 |2|1 bar
2|3|2 bars
3|4|3 bars
4|5|4 bars
n/a|variable, up to 7|auto

'last' in the first row (when AC is powered off) means that the value of Fan is according to the last setting.    

For the following diagram Fan was set to auto via IR-RC for Mode=Cool, you see the increase of IUFANSPEED from 0 to 2, 4, 6, 7:
![grafik](https://user-images.githubusercontent.com/23119513/176353823-e1e75c35-50f2-41e4-9889-4ce4ddaba24f.png)

Note that IUFANSPEED=2 is not visible in the diagram due to the low sample rate used for it.

### Vanes
Vanes up/down swing is enabled in DB0[6]. When vanes up/down swing is disabled the vanes up/down position in MOSI DB1[5:4] is used. 

DB0	| Function
---- | -----
Bit 6| vanes up/down swing
0 | off
1 | on

<table style="width: 273px; height: 68px;">
<thead>
<tr>
<td style="width: 66.9667px;" colspan="2"><strong>DB1</strong></td>
<td style="width: 66.9667px;"><strong>Function</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td style="width: 66.9667px;">bit 5</td>
<td style="width: 71.4333px;">bit 4</td>
<td style="width: 66.9667px;">Vanes up/down position</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">1</td>
</tr>
<tr>
<td style="width: 66.9667px;">0</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">2</td>
</tr>
<tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">0</td>
<td style="width: 66.9667px;">3</td>
</tr>
<tr>
<td style="width: 66.9667px;">1</td>
<td style="width: 71.4333px;">1</td>
<td style="width: 66.9667px;">4</td>
</tr>
</tbody>
</table>

note: Vanes status is **not** applicable when using IR remote control. The latest vanes status is only visible when it was changed by the SPI-RC.
The latest vanes status is only visible when MOSI DB0[7]=1 or MOSI DB1[7]=1.

The same coding is used for setting vanes.
The set bit for enabling vanes up/down swing in the MISO frame is DB0[7].
The set bit for vanes up/down position in the MISO frame is DB1[7].

### Room temperature
The room temperature is coded in MOSI DB3[7:0] according to the formula T[°C]=(DB3[7:0]-61)/4
The resolution is 0.25°C. When writing a value <255, this temperature value is used instead of the build in room temperature sensor.

### Temperature setpoint
The temperature setpoint is coded in MOSI DB2[6:0] according to the formula T[°C]=DB2[6:0]/2
The resolution of 0.5°C is supported by the wired remote control [RC-E5](https://www.mhi-mth.co.jp/en/products/pdf/pjz012a087b_german.pdf) and MHI-AC-Ctrl. But the IR remote control supports a resolution of 1°C only.
The same coding is used for setting the temperature. The set bit in the MISO frame is DB2[7].

### Error code (read only)
The Error code is a number 0 ... 255 in MOSI DB4[7:0]. 0 means no error. 
According my understanding the error codes listed [here](https://www.hrponline.co.uk/media/pdf/5f/54/33/HRP_NEW_ServiceSupportHandbook.pdf#page=14) are supported, but I haven't really checked it.

## Checksum (read only)
The two byte checksum is calculated by the sum of the signature bytes plus the databytes. The high byte of the checksum CBH is stored at byte position 18 and the low byte of the checksum CBL is stored at byte position 19.

    checksum[15:0] = sum(SB0:SB2) + sum(DB0:DB14)
    CBH = checksum[15:8]
    CBL = checksum[7:0]

The extended frame continues after last databyte at position 17, skips the two previous checksum bytes and only the low byte is stored in CBL2 at position 32.

    CBL2 = checksum[7:0] = sum(SB0:SB2) + sum(DB0:DB14) + sum(DB15:DB26)


## Settings
For writing MHI-AC - depending on the function - a specific MISO set-bit is used:

function | set-bit
---- | ----
Power|DB0[1]
Mode|DB0[5]
Fan|DB1[3]
Vanes|DB0[7] for swing and DB1[7] for up/down position
Tsetpoint|DB2[7]

Once a set-bit is set to 1, the according bit in the MOSI frame becomes and remains '1' until the IR remote control is used.
All set-bits are cleared when the IR remote control is used. Settings can be done independent from the power state.

## Operation Data

You can read different operating data of the AC related to the indoor and outdoor unit.
The following example shows the reading of the outdoor air temperature:

MISO-DB6 | MISO-DB9 | MISO-DB10 | MISO-DB11 | MISO-DB12
 --------| ---------| ----------| ----------| ----
  0x40   | 0x80     | 0xff      | 0xff      | 0xff

MOSI-DB6 | MOSI-DB9 | MOSI-DB10 | MOSI-DB11 
 --------| ---------| ----------| ----------
 bit7==0 | 0x80     | 0x10      | outdoor temperature 

Please check the program code for further details. You find [here](https://github.com/absalom-muc/MHI-AC-Ctrl/blob/master/SW-Configuration.md#operating-data-mhi-ac-ctrl-coreh) the list of some supported topics related to operating data. And some details related to the operating data are listed in section [Operation Data Details](#operation-data-details). Please consider that the decoding of the data could be imcomplete since only one AC (SRK 35 ZS-S) was available for testing.

## Last Error Operation Data
The AC stores some operation data of the last error event. This error operation data can be read with the command:

MISO-DB6[7] | MISO-DB9 
 ---------- | --------
1           | 0x45    
  
  
AC answers with the following MOSI data sequence:

MOSI-DB6[7] | MOSI-DB9 | MOSI-DB10 | MOSI-DB11 
 ---------- | -------- | --------- | --------  
 1          | 0x45     | 0x11      | error code

If error code > 0 the following MOSI data sequence is send in addition:

MOSI-DB6[7] | MOSI-DB9 | MOSI-DB10 | MOSI-DB11 
 ---------- | -------- | --------- | --------  
 1          | 0x45     | 0x12      | count of the following error operation data

Examples for error operation data

MOSI-DB6[7] | MOSI-DB9 | MOSI-DB10 | MOSI-DB11 
 ---------- | -------- | --------- | --------  
 1          | 0x02     | 0x30=Stop, 0x31=Dry, 0x32=Cold, 0x33=Fan, 0x34=heat
 1          | 0x05     | 0x33      | SET TEMP = MOSI-DB11 / 2
 1          | 0x1e     | 0x30      | TOTAL I/U RUN = MOSI-DB11 * 100

## Summary
MOSI frame:
![grafik](https://user-images.githubusercontent.com/23119513/201548671-3128920e-8aac-4742-8fe9-bfc4ae00f041.png)

## Operation Data Details
Here you find detailed information related to some operating data. It is mainly based on my observations and might be partly not correct. It seems that there are different operating data available for different ACs. So there might be operating data not supported by your AC (e.g. the number of implemented temperature sensors) or there might be additional operating data available for your AC but not supported by the SW. The same is valid for the error operation data. I'm not aware how to find out addtional operation data w/o using a third party wired remote control.

### 1 "MODE"
Stored in the lower 4 bits of DB10:
DB10[3:0] | MODE 
 ---------| ------
 0x0      | Auto or Stop in error mode 
 0x1      | Dry 
 0x2      | Cool
 0x3      | Fan
 0x4      | Heat   

### 2 "SET-TEMP" [°C]
The setpoint for the temperature is coded in MOSI-DB11[6:0] according to the formula T[°C]=DB11[6:0]/2. The value is identical to [Temperature setpoint](#temperature-setpoint) but with the lower resolution of 1°C when the AC is not powered. But when the AC is powered and Mode=HEAT then SET-TEMP=Temperature setpoint+2°C. This behavior can be changed for some ACs, see [here](https://github.com/absalom-muc/MHI-AC-Ctrl/issues/101#issuecomment-1292016712).

### 3 "RETURN-AIR" [°C]
Nearly identical to [Room temperature](#room-temperature). Sometimes the two values differ by <0.5 ° C. Background is not clear. The RETURN-AIR temperature is calculated by T[°C]=(DB11[7:0]-61)/4. The resolution is 0.25°C.

### 12 "TOTAL-IU-RUN" [h]
Operating hours Indoor Unit[h]=DB11*100

### 21 "OUTDOOR" [°C]
Outdoor temperature [°C]=DB11

### 24 "COMP" [Hz]
Compressor frequency=(db10-0x10)*25.6f + 0.1f * db11, for db10>=0x10

### 27 "TD" [°C]
Discharge Pipe Temperature,  
if db11 < 0x12 TD <=30°C,    
else TD = db11 / 2 + 32 (approx)

### 29 "CT" [A]
Current
CT[A]=DB11*14/51

Resolution is 274.5mA. Multiply the current by the voltage (230V) to calculate the power.

### 32 "TDSH" [°C]
TDSH[°C]=DB11/2

### 37 "TOTAL-COMP-RUN" [h]
Operating hours compressor [h]=DB11*100

## Unknown
In the SPI frames are more information coded than known for me. In MOSI-DB13 some bits seem to represent the status of the outdoor unit.
I would appreciate your support to close these gaps.

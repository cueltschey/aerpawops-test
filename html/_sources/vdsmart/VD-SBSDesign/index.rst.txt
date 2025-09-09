SBS Design
==========

The PCB is done in mm (not mil)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the LDO driving AVCC0 turns off, what happens for the ADC input pins?

MCU Connections
---------------

System control
--------------

-  MD (operating mode control): P201 (26)

   -  This pin must be set before releasing reset (RES) and held
      constant for the duration of the MCU’s uptime or until the next
      assertion of RES.

================ ================
Mode-setting pin 
================ ================
MD               Operating mode
1                Single-chip mode
0                SCI boot mode
================ ================

|image1|

An onboard 0.1” header will be used to let the user add a jumper in case
SCI booting is desired

-  RES (active low reset): (5)

   -  The S128 User Manual discusses an internal power-on reset in
      section 5.3.2.
   -  The DK-S128 also lacks a power on reset circuit.

|image2|

A manual reset switch can be optionally populated

The 9-pin JTAG port is also linked to the MCU_nRESET net

Power
^^^^^

S128 datasheet Table 2.1
^^^^^^^^^^^^^^^^^^^^^^^^

-  VCC and VSS need 0.1 uF caps between each pair of these pins
-  VCL connected to VSS through a 4.7-uF capacitor
-  AVCC0 and AVSS0 need a 0.1 uF cap
-  VREFH0 and VREFL0 connect to reference supply and need a 0.1 uF cap

Clocks
------

-  A 16 MHz external crystal will be connected to XTAL and EXTAL.

   -  XTAL: P213 (9)
   -  EXTAL: P212 (10)

-  To enable subosc-speed mode (current draw drops to uA), the sub-clock
   oscillator (connected through XCIN and XCOUT) must also be connected
   (S128 datasheet Table 2.11). This crystal must be 32.768 kHz.

   -  XCIN: P215 (6)
   -  XCOUT: P214 (7)

|image3|

S128 User Manual section 8.3.1 suggests the following connection scheme
for all crystals

Selecting crystal oscillator load capacitors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Crystals will specify a range/typical expected load capacitance. This
load capacitance can be expressed as CL = (CL1*CL2)/(CL1+CL2) + Cp
(parallel combination of CL1 , CL2 and the series combination of those
and parasitic capacitance Cp usually in the 2~5 pF range depending on
traces) ( page 9). For CL1 = CL2 = C and solving for C = 2*(CL - Cp) .
http://ww1.microchip.com/downloads/en/appnotes/00826a.pdf

Capacitors selected should have low temperature coefficients (e.g. C0G).

16 MHz Oscillator

CL = 12 pF and Cp = ~4 pF, so C = 16 pF (each)

`https://media.digikey.com/pdf/Data <https://media.digikey.com/pdf/Data%20Sheets/Jauch%20Quartz%20PDFs/J49SMH_Type_DS.pdf>`__
Sheets/Jauch Quartz PDFs/J49SMH_Type_DS.pdf

32.768 kHz Oscillator

CL = 12.5 pF and Cp = ~4 pF, so C = 17 pF (going to use same caps as for
the 16 MHz oscillator)

Serial Wire Debug
-----------------

Target Interface SWD - SEGGER
`Knowledge <https://wiki.segger.com/SWD>`__ Base

-  SWCLK: P300 (32)
-  SWDIO: P108 (33)

9-pin J-LINK

`J-Link <https://www.segger.com/products/debug-probes/j-link/models/j-link-edu-mini/>`__
EDU Mini

Consumes:

-  SWDIO
-  SWCLK
-  RES connection to S128 MCU
-  TDO and TDI are unused

NOTE: the JLINK_OB/JTAG circuit in the DK-S128 uses U12 (a MCU) and U10B
(tri-state buffer) to translate USB to SWD. U12 receives USB
communication and bitbangs data through U10B to the SWD pins of the
board’s MCU. This does not need to be replicated.

RGB LED
-------

`https://www.digikey.com/en/products/detail/würt <https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/155124M173200/7315806>`__
`h-elektronik/155124M173200/7315806 <https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/155124M173200/7315806>`__

GPIO drive strength (excluding P000-P004, P010-P015, P212, P213,
P500-P502, P408, P409, P914, P915) is +/- 4 mA for low drive, +/- 8 mA
for high drive.

|image4|

Onboard JTAG connection

Desired thru current: 3 mA or less when powered from 3V3 rail and sunk
into MCU GPIO.

===== ================= ========
Color ForwardVoltage(V) Resistor
===== ================= ========
Red   2.0               1kOhm
Green 2.8               750Ohm
Blue  3.0               666Ohm
===== ================= ========

For simplicity’s sake, using 1 kOhm for each of these. Can adjust thru
current based on user feedback.

Piezo
-----

`https://www.digikey.com/en/products/detail/mallory-sonalert <https://www.digikey.com/en/products/detail/mallory-sonalert-products-inc/PK-11N40PQ/4996072>`__\ `products-inc/PK-11N40PQ/4996072 <https://www.digikey.com/en/products/detail/mallory-sonalert-products-inc/PK-11N40PQ/4996072>`__

The piezo drive current should be delivered by a MOSFET instead of a
GPIO. A cheap SOT23-3 FET, capable of handling VGS = 3V was selected. It
should be on the low side since the MCU GPIO will only reach 3V.

|image5|

.. _power-1:

Power
-----

Remainder of board

Powering with a controllable 3V3 LDO.

|image6|

MCU Power
^^^^^^^^^

To provide a stable power rail for the MCU analog blocks (ADC is this
project’s concern), a

|image7|

used on board. The chosen LDO, TPS70933DBVR, recommends using a 1u
ceramics at IN and a 2.2u ceramic at Vout.

An internal pull-up circuit sets the default of enable to active.

A TLV431 can be populated onboard to provide the ADC with a reference
more stable than the MCU’s internal reference.

Reference voltage
^^^^^^^^^^^^^^^^^

+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      |              |                           | TEST      |   |   | T |   |   |
|      |              |                           | C         |   |   | L |   |   |
|      |              |                           | ONDITIONS |   |   | V |   |   |
|      |              |                           |           |   |   | 4 |   |   |
|      |              |                           |           |   |   | 3 |   |   |
|      |              |                           |           |   |   | 1 |   |   |
|      |              |                           |           |   |   | A |   |   |
+======+==============+===========================+===========+===+===+===+===+===+
|      | PARAMETER    |                           |           |   |   | T | M | U |
|      |              |                           |           |   |   | Y | A | N |
|      |              |                           |           |   |   | P | X | I |
|      |              |                           |           |   |   |   |   | T |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      |              |                           | T A =     |   | 1 | 1 | 1 |   |
|      |              |                           | 25°C      |   | . | . | . |   |
|      |              |                           |           |   | 2 | 2 | 2 |   |
|      |              |                           |           |   | 2 | 4 | 5 |   |
|      |              |                           |           |   | 8 |   | 2 |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      | B ( )        | V KA = V REF ,            | (1)       | T | 1 |   | 1 | v |
|      |              |                           |           | L | . |   | . |   |
|      |              |                           |           | V | 2 |   | 2 |   |
|      |              |                           |           | 4 | 2 |   | 5 |   |
|      |              |                           |           | 3 | 1 |   | 9 |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | C |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| VREF | Reference    | :math:`I                  | T A =     | T | 1 |   | 1 | v |
|      | voltage      | _{\rm K} = 10 \text{ mA}` | full      | L | . |   | . |   |
|      |              |                           | range     | V | 2 |   | 2 |   |
|      |              |                           | (1)(see   | 4 | 1 |   | 6 |   |
|      |              |                           | Figure    | 3 | 5 |   | 5 |   |
|      |              |                           | 22)       | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | I |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      |              |                           | (see      | T | 1 |   | 1 |   |
|      |              |                           | rigule    | L | . |   | . |   |
|      |              |                           | zz)       | V | 2 |   | 2 |   |
|      |              |                           |           | 4 | 0 |   | 7 |   |
|      |              |                           |           | 3 | 9 |   | 1 |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | Q |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      |              |                           |           | T |   | 4 | 1 | m |
|      |              |                           |           | L |   |   | 2 | V |
|      |              |                           |           | V |   |   |   |   |
|      |              |                           |           | 4 |   |   |   |   |
|      |              |                           |           | 3 |   |   |   |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | C |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| V    | F(dev) V REF | V KA = V REF , I K = 10   |           | T |   | 6 | 2 |   |
| REF( | deviation    | mA (1)(see Figure 22)     |           | L |   |   | 0 |   |
| dev) | over         | TLV431AI                  |           | V |   |   |   |   |
|      | ful          |                           |           | 4 |   |   |   |   |
|      | ltemperature |                           |           | 3 |   |   |   |   |
|      | range (2)    |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | I |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      | temperature  |                           | (see      |   |   | 1 | 3 |   |
|      | range        |                           | Figure    |   |   | 1 | 1 |   |
|      |              |                           | 22)       |   |   |   |   |   |
|      |              |                           | TLV431AQ  |   |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| :ma  | Ratio of V   | :math:`\label{eq:VKA} \be |           |   |   | - | - | m |
| th:` | REF change   | gin{split} V_{KA} &= V_{R |           |   |   | 1 | 2 | V |
| \Del | in           | EF} \text{ to } 6 \ \text |           |   |   | . | . | / |
| ta V | ca           | {V}, \ \text{I}_{K} = 10  |           |   |   | 5 | 7 | \ |
| _{RE | thodevoltage | \ \text{mA} \\ (\text{see |           |   |   |   |   | \ |
| F}`\ | change       |  Figure 23}) \end{split}` |           |   |   |   |   |   |
|  \ : |              |                           |           |   |   |   |   |   |
| math |              |                           |           |   |   |   |   |   |
| :`\D |              |                           |           |   |   |   |   |   |
| elta |              |                           |           |   |   |   |   |   |
|  V_{ |              |                           |           |   |   |   |   |   |
| KA}` |              |                           |           |   |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| Iret | Reference    | I K = 10 mA, R1 = 10      |           |   | 0 | 0 | μ |   |
|      | terminal     | kΩ,R2 = open(see Figure   |           |   | . | . | Α |   |
|      | current      | 23)                       |           |   | 1 | 5 |   |   |
|      |              |                           |           |   | 5 |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      |              | Lu = 10 mA P1 = 10 k0     |           | T |   | 0 | 0 |   |
|      |              |                           |           | L |   | . | . |   |
|      |              |                           |           | V |   | 0 | 3 |   |
|      |              |                           |           | 4 |   | 5 |   |   |
|      |              |                           |           | 3 |   |   |   |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | C |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| ref( | I ref        |                           |           | T |   | 0 | 0 | μ |
| dev) | deviation    |                           |           | L |   | . | . | Α |
|      | over full    |                           |           | V |   | 1 | 4 |   |
|      | temp         |                           |           | 4 |   |   |   |   |
|      | eraturerange |                           |           | 3 |   |   |   |   |
|      | (2)          |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | I |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      | Tange        |                           | (ace      | T |   | 0 | 0 |   |
|      |              |                           | rigure    | L |   | . | . |   |
|      |              |                           | 20)       | V |   | 1 | 5 |   |
|      |              |                           |           | 4 |   | 5 |   |   |
|      |              |                           |           | 3 |   |   |   |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | Q |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
|      | Minimum      |                           | E: 000    | T |   | 5 | 8 |   |
|      | cathode      |                           |           | L |   | 5 | 0 |   |
|      | current for  |                           |           | V |   |   |   |   |
|      |              |                           |           | 4 |   |   |   |   |
|      |              |                           |           | 3 |   |   |   |   |
|      |              |                           |           | 1 |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | C |   |   |   |   |
|      |              |                           |           | / |   |   |   |   |
|      |              |                           |           | A |   |   |   |   |
|      |              |                           |           | I |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| K(   | regulation   | VKA = VREF (S             | KA = V    |   |   | 5 | 1 | μ |
| min) |              |                           | REF (see  |   |   | 5 | 0 | A |
|      |              |                           | Figure    |   |   |   | 0 |   |
|      |              |                           | 22)       |   |   |   |   |   |
|      |              |                           | TLV431AQ  |   |   |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| K(   | Off-state    | :m                        | = 6 V     | ) |   | 0 | 0 | μ |
| ott) | cathode      | ath:`V_{REF} = 0, V_{KA}` | (see      |   |   | . | . | Α |
|      | current      |                           | Figure 24 |   |   | 0 | 1 |   |
|      |              |                           |           |   |   | 0 |   |   |
|      |              |                           |           |   |   | 1 |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+
| z KA | Dynamic      | :math:`V_{KA} = V_{REF}`  |           |   | 0 | 0 | Ω |   |
|      | impedance    | , f :math:`\leq` 1 kHz,   |           |   | . | . |   |   |
|      | (3)          | :math:`I_K = 0.1` mA to   |           |   | 2 | 4 |   |   |
|      |              | 15 mA(see Figure 22)      |           |   | 5 |   |   |   |
+------+--------------+---------------------------+-----------+---+---+---+---+---+

https://www.ti.com/lit/ds/symlink/tlv431a.pdf

.. math:: V_{O} = (1 + R1 / R2) \times V_{ref} - I_{ref} \times R1

|image8|

https://www.ti.com/lit/ds/symlink/tlv431a.pdf

https://www.ti.com/lit/ds/symlink/tlv431a.pdf section 9.2.2.2.1

The recommended cathode current ranges [0.1, 15] mA with an absolute
maximum of 20 mA. A 180 ohm R\_{SUP} will pass ~9.2 mA.

In the typical case, V\_{ref} = 1.24 V, I\_{ref} = 0.15 uA. This circuit
will not be powered while the SBS is sleeping, so lower resistor values
can be used for better noise immunity. Let R1 = 10 kOhm, so R2 = 30
kOhm. With these resistor values, the actual nominal V_O is ~1.65183 V.

TLV431 Output `Voltage <https://www.desmos.com/calculator/qap5we216u>`__
------------------------------------------------------------------------

+----------+-----------------------------------------------------------+
| V\_{     | Conditions                                                |
| O}Corner |                                                           |
+==========+===========================================================+
| Low      | MinimumR1,maximumR2,minimumV\_{ref},maximumI\_{ref}       |
+----------+-----------------------------------------------------------+
| High     | MaximumR1,minimumR2,maximumV\_{ref},minimumI\_{ref}(zero) |
+----------+-----------------------------------------------------------+

+-----------------------------+---------+--------------+--------------+
| ResistorValues              | Tol     | V            | V            |
|                             | erances | \_{O,MIN}(V) | \_{O,MAX}(V) |
+=============================+=========+==============+==============+
| R110k,R230k==               | 1%      | 1.6243       | 1.6778       |
+-----------------------------+---------+--------------+--------------+
| R110k,R230k==               | 0.1%    | 1.6315       | 1.6702       |
+-----------------------------+---------+--------------+--------------+
| R11k,R23k==                 | 1%      | 1.6287       | 1.6778       |
+-----------------------------+---------+--------------+--------------+
| R11k,R23k==                 | 0.1%    | 1.6360       | 1.6702       |
+-----------------------------+---------+--------------+--------------+

The largest contributor to the accuracy of the output voltage is the
accuracy of the V\_{ref} signal produced inside the TLV431.

The resistor values and tolerances can be changed independent of the
rest of the SBS hardware.

Using a precise external voltage reference reduces the uncertainty of
the reference to under +/- 2% from +/- 10%.

ADC Input Clamping

+---------+-------------------------------+-----------+-------------+---+
| Input   | 5 V tolerant ports*1          | V in      | -0.3 to     | V |
| voltage |                               |           | +6.5        |   |
+=========+===============================+===========+=============+===+
|         | P000 to P004P010 to P015P500  | V in      | -0.3 to     | V |
|         | to P502                       |           | AVCC0 + 0.3 |   |
+---------+-------------------------------+-----------+-------------+---+
|         | Others                        | V in      | -0.3 to VCC | V |
|         |                               |           | + 0.3       |   |
+---------+-------------------------------+-----------+-------------+---+

S128 Datasheet section 2.1 (Absolute Maximum Ratings)

Consulting the S128 User Manual section 1.7 (Pin Lists) shows the ports
related to AVCC0 are ADC inputs AN000 through AN013.

From the of this page, AVCC0 = 3.3 V. An arbitrary upper limit can be
set to voltages input to ADC channels of 3 V with a zener diode and a
currentlimiting resistor. Analog Power
`section <https://aerpaw-uav.atlassian.net/wiki/spaces/VD/pages/182681698/SBS+Design#Analog-Power>`__

This circuit should be used for every ADC input that can theoretically
exceed 3.3 V (e.g. Cell Voltage Difference Amplifiers).

From Smart Battery System \|
`Modeling <https://aerpaw-uav.atlassian.net/wiki/spaces/VD/pages/164331521/Smart+Battery+System#Modeling-ADC-inputs>`__
ADC inputs , the ADC input resistance is on the order of kOhm and input
capacitances on the order of pF. The current-limiting resistor should be
adequately sized to limit worst-case current through the zener diode
based on the analog voltage’s driving capability (but no more than 100
mW dissipated, so 33 mA limit).

Detect Signal
~~~~~~~~~~~

Detect_IN is connected to P106 (AN016), which is tolerant up to VCC.
Detect_OUT , driven by a GPIO, can supply up to 4 mA in low drive mode
at 3V. Thus, the series resistance can be no less than 750 Ohms.

For the given resistor divider, Rx is the Connected resistor. The high
and low side resistors limit the max current and

allows for Rx to be smaller than if only the low-side resistor limited
the current.

Rx = RL \* (Vin/Vout - 1) - RH

Rx = 470 \* (3/Vout - 1) - 470

+---------+----------------+----------------+-------------+-----------+
| Mode    | VoltageMin(V)  | VoltageMax(V)  | RxNominal   | C         |
|         |                |                |             | losestE24 |
+=========+================+================+=============+===========+
| Nohost  | 0              | 0.1            | None/open   | Open      |
+---------+----------------+----------------+-------------+-----------+
| 1       | 0.2            | 0.3            | 4.7k        | 4.7k      |
| 2Sdrone |                |                |             |           |
+---------+----------------+----------------+-------------+-----------+
| 6Sdrone | 0.4            | 0.5            | 2.1933k     | 2.2k      |
+---------+----------------+----------------+-------------+-----------+
| R       | 0.6            | 1.1            |             |           |
| eserved |                |                |             |           |
+---------+----------------+----------------+-------------+-----------+
| Charger | 1.2            | 3              | Short       | 0         |
+---------+----------------+----------------+-------------+-----------+

Thermistor
==========

Create a resistor divider with the thermistor as the low leg and a fixed
R as the top leg.

|image9|

|image10|

`NTCLE413E2103H401 <https://www.vishay.com/docs/29078/ntcle413.pdf>`__ datasheet
~~~~~~~~~~~~~~~~~~~

NTC `Thermistor <https://www.desmos.com/calculator/jdfhc9kqfy>`__
~~~~~~~~~~~~~~~~~~~~~

-  Resistor pole powered by 3V3
-  Maximum readable voltage is 1.3 V (minimum Vref internal)
-  Minimum readable voltage assume is 50 mV
-  With RH = 56k and the given thermistor, able to resolve temperatures
   in range [-1.2, 91] °C

   -  Generally, lowering RH shifts the readable range more negative and
      raising RH shifts it more positive

Cell, Stack, and Output Voltage measurement
-------------------------------------------

-  Max cell voltage expected is 4.3 V → round up to 4.4 V
-  Max expected voltage on either pin (for 6S compatibility) is ( 6 cell

::

   * 4.4 V/cell = 26.8 V )

-  V\_{out} = V\_{ref} and V\_{in} = 26.8 V
-  Using internal reference, V\_{ref_internal,min} = 1.30 V

   -  Ratio = 1.3/26.8
   -  Approximate match: RL = 510, RH = 10k
   -  To limit current, use RL = 51k, RH = 1Meg to limit thru current to
      ~84 uA (for all cells)

      -  Since V1 of SBS uses resistor poles for cell measurement, too,
         the voltage for cell 6 is also the same as the stack voltage,
         so the pole for the stack can be removed

Using external reference, VREFH0\_{min} = 1.60 V (rounded)

-  RL = 750, RH = 24k for a divide ratio of 0.030303030 (inverse is
   33.0)
-  Scale to RL = 75k and RH = 2.4M to reduce thru current to 52.8 uA

Cell Voltage Difference Amplifiers (deprecated for V1 of SBS)

|image11|

|image12|

Example cell measurement resistor pole

If this SBS is to be converted to support 12 cell batteries, this entire
block must be revised (OPAx990 max supply voltage is 40 V) and
current-limiting R based on 6-cell maximum.

A Deeper Look into
`Difference <https://www.analog.com/en/analog-dialogue/articles/deeper-look-into-difference-amplifiers.html>`__
Amplifiers \| Analog Devices

-  Shutdown pins (SHDN) on the OPA4990S use logic levels

   -  Open is pulled down internally to logic low, so must input a high
      to shutdown
   -  SHDN pin cannot exceed (V-) + 20V or (V+) (whichever is lower)

-  Resistor dividers

   -  Max cell voltage expected is 4.3 V → round up to 4.4 V
   -  Must scale down by V\_{ref}/4.4 = (R2/R1) (taken from the Op Amp
      circuit shown)

      -  Note that R1 = R3 and R2 = R4

   -  Using internal reference, V\_{ref_internal,min} = 1.30 V

      -  R2/R1 = 1.30/4.4 :sub:`=` 2.7/9.1

   -  Using external reference, VREFH0\_{min} = 1.60 V (rounded)

      -  R2/R1 = 1.60/4.4 = 1.2/3.3 (both are E24 numbers)
      -  \* This is what is used on V1 of the 6S SBS \*

   -  Problem: as long as a battery is plugged in, the path from V2 to
      GND conducts current

      -  Keep R on the order of 100k-1M to keep this small
      -  Alternative: add a low current FET (~20 mA or higher) between
         R4 and GND. Enable only when taking ADC measurements. Size R on
         the order of 1k or 10k to improve noise immunity since this
         path is closed most of the time.

-  Voltage clamp

   -  Must limit output to 3 V with zener
   -  Highest expected stack voltage is (6 cell \* 4.2 V = 25.2 V)

OPA4990 short-circuit current limit (per amplifier)

|image13|

.. math:: V_{OUT} = \left(\frac{R4}{R3 + R4}\right) \times \left(\frac{RI + R2}{RI}\right) \times V2 - \frac{R2}{RI} VI

(1)

Four resistor Op Amp circuit Note the - input is above the + input

-  Current-limiting resistor shall limit this to 30 mA to not exceed the
   (arbitrarily chosen) 100 mW zener diode limit → 740 Ohm → 1 kOhm can
   be used as a current limit

   -  This situation likely won’t happen since the amplifier would need
      to be driving 25.2 V for (25.2 - 3) V to appear across the 1k, but
      this is a worst-case-make-sure-nothing-breaks spec

FET Array and Gate Driver
~~~~~~~~~~~~~~~

-  LTC7000 is a high-side N-channel MOSFET gate driver
-  This implementation will ignore many features of the LTC7000,
   somewhat treating it as if it were the LTC7000-1

+---+---------------------------------------------+-----------------------+
| P | Summary                                     | Connection            |
| i |                                             |                       |
| n |                                             |                       |
+===+=============================================+=======================+
| R | ActivehighChi                               | Yes                   |
| U | pEnablepin.Iflogiclow,LTC7000entersashutdow |                       |
| N | nmode.ThismustbeactiveforINPtohaveaneffect. |                       |
+---+---------------------------------------------+-----------------------+
| V | Chipmainsupplypin.                          | Yes,usea0             |
| \ |                                             | .1uFceramictodecouple |
| _ |                                             |                       |
| { |                                             |                       |
| I |                                             |                       |
| N |                                             |                       |
| } |                                             |                       |
+---+---------------------------------------------+-----------------------+
| V | InternalL                                   | Yes,usea1.0uFlowES    |
| \ | DOoutputforgatedriversandinternalcircuitry. | Rceramictodecouple.Do |
| _ |                                             | n’tuseforanythingelse |
| { |                                             |                       |
| C |                                             |                       |
| C |                                             |                       |
| } |                                             |                       |
+---+---------------------------------------------+-----------------------+
| V | Setsth                                      | Tominimizeopportunity |
| \ | eUVLOfortheGateDrive(V\_{CC})pin.ShorttoGND | forthegatedrivertocut |
| _ | setsUVLOto3.5V(minimum),opensetsUVLOto7.0V. | offpower,theUVLOshoul |
| { |                                             | dbeminimized,soV\_{CC |
| C |                                             | UV}willbeshortedtoGND |
| C |                                             |                       |
| U |                                             |                       |
| V |                                             |                       |
| } |                                             |                       |
+---+---------------------------------------------+-----------------------+
| ~ | Open                                        | Noconnect             |
| { | DrainFaultOutput-usedtonotifyofanimpendingF |                       |
| F | ETturnoffasaresultofanovercurrentcondition. |                       |
| A |                                             |                       |
| U |                                             |                       |
| L |                                             |                       |
| T |                                             |                       |
| } |                                             |                       |
+---+---------------------------------------------+-----------------------+
| T | “IftheTIMERpinisconnectedtoVCC              | ConnecttoVCCsincei    |
| I | oranyothersupplygreaterthan3.5V(absmax15V), | tisnotpossibleforanov |
| M | anovercurrenteventwillimmediatelypullTGDNto | ercurrenteventtoarise |
| E | TSandtheLTC7000/LTC7000-1willremainthereunt |                       |
| R | iltheINPsignalhascycledlowandthenbackhigh.” |                       |
+---+---------------------------------------------+-----------------------+

LTC7000 Pinout
--------------

+---+-----------------------------------+------------------------------+
| I | Gatedriveenablepin.Fastswitch     | Yes                          |
| N | ingofthegatedriverstate.RUNmustbe |                              |
| P | activeforthisinputtohaveaneffect. |                              |
+===+===================================+==============================+
| O | Overvoltagelockoutinput.          | No-“tiedtoGNDwhennotused”    |
| V |                                   |                              |
| L |                                   |                              |
| O |                                   |                              |
+---+-----------------------------------+------------------------------+
| I | Setsth                            | SNS+andSNS-willh             |
| \ | eovercurrentthresholdmeasuredbyth | ave0Vdifference,sothiscanbel |
| _ | eshuntresistor(acrossSNS+toSNS-pi | eftopentosetathresholdof30mV |
| { | ns).Leaveopenforathresholdof30mV. |                              |
| S |                                   |                              |
| E |                                   |                              |
| T |                                   |                              |
| } |                                   |                              |
+---+-----------------------------------+------------------------------+
| I | SNS+SNS-Voltageoutput=20(-).\*    | Noconnect                    |
| \ |                                   |                              |
| _ |                                   |                              |
| { |                                   |                              |
| M |                                   |                              |
| O |                                   |                              |
| N |                                   |                              |
| } |                                   |                              |
+---+-----------------------------------+------------------------------+
| T | Gatedrivepulldown.                | Connectdirectl               |
| G |                                   | ytoFETgateforfastestturn-off |
| D |                                   |                              |
| N |                                   |                              |
+---+-----------------------------------+------------------------------+
| T | Gatedrivepullup.                  | Co                           |
| G |                                   | nnectdirectlytoFETgateforfas |
| U |                                   | testturn-on;connectthroughre |
| P |                                   | sistortocontrolinrushcurrent |
+---+-----------------------------------+------------------------------+
| T | FETsourceconnection.              | ConnectdirectlytoFETsources  |
| S |                                   |                              |
+---+-----------------------------------+------------------------------+
| B | High-sidebootstrapsupply.Voltage  | Connec                       |
| S | swingonthispinis12VtoV\_{IN}+12V. | ttoTSthroughaminimum0.1uFcap |
| T |                                   |                              |
+---+-----------------------------------+------------------------------+
| S | Currentsensecomparatorinput.Des   | Donotco                      |
| N | iredinputvoltagerangeis3.5to150V. | nnectacrossashuntresistorIns |
| S |                                   | tead,connectbothterminalstot |
| - |                                   | hedrainofthebattery-sideFETs |
| a |                                   |                              |
| n |                                   |                              |
| d |                                   |                              |
| S |                                   |                              |
| N |                                   |                              |
| S |                                   |                              |
| + |                                   |                              |
+---+-----------------------------------+------------------------------+
| G | Groundpad.                        | So                           |
| N |                                   | lderdirectlytoPCBforratedele |
| D |                                   | ctricalandthermalperformance |
| ( |                                   |                              |
| E |                                   |                              |
| P |                                   |                              |
| A |                                   |                              |
| D |                                   |                              |
| ) |                                   |                              |
+---+-----------------------------------+------------------------------+

Limit Inrush Current
--------------------

The turn-on time is controlled by C_G and R_G (both are external
components we get to choose). The load capacitance is something we
estimate. The 10 Ohm resistor can be low power - it’s meant to dampen
oscillations.

Don’t need to limit
-------------------

|image14|

Reverse Current Protection (aka “load switching”)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Just as a reference, this is how the shunt terminals should be connected
in case of back-to-back FETs

|image15|

Flyback diode connection to TS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Schottky (for fast turn-on) should be rated for at least 25.2 V (6
cell \* 4.2 V) → around 50 V or higher for a safety factor.

This diode should not be tiny since it will need to handle (0.7 V)(~10
A) for a short time in the worst case. Since the gate driver won’t cut
off while drone is mid-flight, it would be on the ground. The ideal
diode max tolerable current is 10 A, so assume all the current is
flowing through the FET array.

Choose a schottky in a DPAK/TO-252-3 package so they can be easily
swapped.

|image16|

Current Amplifiers
------------------

Shunt resistance R\_{shunt} = 300 uOhm

+-----+----------------------------------------------------------------+
| F   | Notes                                                          |
| eat |                                                                |
| ure |                                                                |
+=====+================================================================+
| Ac  | V\_{os}=R\_{shunt}\*OpAmpOffsetVoltage:I\_{inaccuracy}         |
| cur |                                                                |
| acy |                                                                |
+-----+----------------------------------------------------------------+
|     | R\_{shunt}=300uOhmI\_{inaccuracy}=V\_{os}/For,(300uOhm)        |
+-----+----------------------------------------------------------------+

+-----+----+-------------+------------------+---------+------+----+---+
|     |    | V\_{os}=+-  | 12uV→            | I       | =(12 | uV |   |
|     |    |             |                  | \_{inac |      | )/ |   |
|     |    |             |                  | curacy} |      |    |   |
+=====+====+=============+==================+=========+======+====+===+
|     | (3 | uOhm)       | =40mA            |         |      |    |   |
|     | 00 |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
| Co  |    | Common-mode | inputrange=      | [-0     | hi   | c  |   |
| mmo |    |             |                  | .2,40]V | gh-s | on |   |
| n-m |    |             |                  |         | ide→ | fi |   |
| ode |    |             |                  |         |      | gf |   |
|     |    |             |                  |         |      | or |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    | V\_{stack}  | <40V             |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
| Dif |    | Diff        | bewithin+/-42V   | canr    | for  | c  |   |
| fer |    | erentialmay |                  | eadthe→ | ward | ur |   |
| ent |    |             |                  |         | /rev | re |   |
| ial |    |             |                  |         | erse | nt |   |
+-----+----+-------------+------------------+---------+------+----+---+
| m   |    |             |                  |         |      |    |   |
| axi |    |             |                  |         |      |    |   |
| mum |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     | Lo | gain(25)    | saturates(above  | ADCVref | at~1 |    |   |
|     | we |             |                  | =1.45V) | 92A. |    |   |
|     | st |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
|     |    |             |                  |         |      |    |   |
+-----+----+-------------+------------------+---------+------+----+---+
| Ou  | T  | ou          | canonlyswingc    | betwe   | and  | ou | b |
| tpu | he | tputvoltage | urrentofopposite | enGNDpo | VS(w | tp | e |
| tsw | 0V | whenreading |                  | larity) | on’t | ut | l |
| ing |    |             |                  |         |      |    | o |
|     |    |             |                  |         |      |    | w |
+-----+----+-------------+------------------+---------+------+----+---+

|image17|

Ideal Diode
===========

-  Must enable load switching and inrush control
-  \* Using same FETs for ideal diode as for FET array on V1 of 6S SBS
   \*

Feature Par ts Notes

+---+---+---------------------------------------------------------------+
| B | Q |                                                               |
| a | 1 |                                                               |
| r |   |                                                               |
| e |   |                                                               |
| M |   |                                                               |
| i |   |                                                               |
| n |   |                                                               |
| i |   |                                                               |
| m |   |                                                               |
| u |   |                                                               |
| m |   |                                                               |
+===+===+===============================================================+
| L | Q | R3preventsQ2parasiticoscillations                             |
| o | 2 |                                                               |
| a | , |                                                               |
| d | R |                                                               |
| S | 3 |                                                               |
| w |   |                                                               |
| i |   |                                                               |
| t |   |                                                               |
| c |   |                                                               |
| h |   |                                                               |
| i |   |                                                               |
| n |   |                                                               |
| g |   |                                                               |
+---+---+---------------------------------------------------------------+
| I | C | ForR4being10kOhm,theinrushcurrentisdescribedbytheeq           |
| n | 1 | uationbelow,whereC\_{LOAD}istheestimatedworstcaseloadcapacita |
| r | , | ncethatthiscircuitisexpectedtodrive.Theidealdiodeinrushcurren |
| u | R | tshouldbelimitedto10Aandtheloadcapacitanceworst-caseisapproxi |
| s | 4 | mately6000uF.ThissetsC1to6nFexactly,buttheclosestcapvaluegrea |
| h |   | terthanthisis6.8nF,whichsetstheinrushto8.82A.Asmallervalueisn |
| c |   | otusedtoavoidexceedingthe10Ainrushcurrentrequirement.REVIEWME |
| o |   |                                                               |
| n |   |                                                               |
| t |   |                                                               |
| r |   |                                                               |
| o |   |                                                               |
| l |   |                                                               |
| ( |   |                                                               |
| a |   |                                                               |
| k |   |                                                               |
| a |   |                                                               |
| “ |   |                                                               |
| s |   |                                                               |
| o |   |                                                               |
| f |   |                                                               |
| t |   |                                                               |
| s |   |                                                               |
| t |   |                                                               |
| a |   |                                                               |
| r |   |                                                               |
| t |   |                                                               |
| i |   |                                                               |
| n |   |                                                               |
| g |   |                                                               |
| ” |   |                                                               |
| ) |   |                                                               |
+---+---+---------------------------------------------------------------+
| R | D | Fromcommutation(switching),reverserecoveryspikescanbeseeno    |
| e | 1 | nOUT,IN,andSOURCE.Unlikethedatasheet,the“inputshortcircuit”co |
| v | , | nditioniscan’toccurbecausethebatteryisnotdisconnectedwhensupp |
| e | R | lyingpowernorwouldashortappearacrossthebatteryinputterminals. |
| r | 1 |                                                               |
| s | , |                                                               |
| e | D |                                                               |
| r | 2 |                                                               |
| e | , |                                                               |
| c | C |                                                               |
| o | O |                                                               |
| v | U |                                                               |
| e | T |                                                               |
| r | , |                                                               |
| y | D |                                                               |
| v | 4 |                                                               |
| o |   |                                                               |
| l |   |                                                               |
| t |   |                                                               |
| a |   |                                                               |
| g |   |                                                               |
| e |   |                                                               |
| s |   |                                                               |
| p |   |                                                               |
| i |   |                                                               |
| k |   |                                                               |
| e |   |                                                               |
| s |   |                                                               |
+---+---+---------------------------------------------------------------+

The ideal diode will also be supplying a current on the order of 0.5 A
to several amps. The gate ramp set up by C1 and R4 will prevent large
inrush currents.

The greatest load step cases are when first enabling the ideal diode and
disconnecting the load. Because the supply current is small, inductive
kickbacks will be as well.

D1 protects IN and SOURCE during load steps and overvoltage conditions.
The only power source to be used by the 6S SBS is a 6-cell battery (4.2
V/cell), so overvoltage protection is not needed. The small supply
current and inrush control will mitigate the load step voltage spikes,
so this feature isn’t required either.

D2 blocks conduction for a range reverse input voltages caused by
reverse recovery. Because of the small current supply, these spikes are
being ignored.

“If reverse input protection and fast turn off time are not required, R1
can be removed and VSS connected to system ground.” COUT is also meant
to preserve fast turn-off times and absorbing reverse recovery energy.
Both of these components are being omitted.

D4 prevents VGS from increasing in case of a negative spike on IN or
SOURCE. These spikes are being ignored because they are expected to be
small.

If this SBS is to be converted to support 12 cell batteries, including
components described in this section is strongly encouraged.

.. |image1| image:: _page_0_Figure_9.jpeg
.. |image2| image:: _page_1_Figure_0.jpeg
.. |image3| image:: _page_2_Figure_0.jpeg
.. |image4| image:: _page_3_Figure_4.jpeg
.. |image5| image:: _page_3_Figure_12.jpeg
.. |image6| image:: _page_4_Figure_0.jpeg
.. |image7| image:: _page_4_Figure_3.jpeg
.. |image8| image:: _page_4_Figure_11.jpeg
.. |image9| image:: _page_6_Figure_12.jpeg
.. |image10| image:: _page_6_Figure_13.jpeg
.. |image11| image:: _page_7_Picture_21.jpeg
.. |image12| image:: _page_7_Figure_22.jpeg
.. |image13| image:: _page_8_Figure_22.jpeg
.. |image14| image:: _page_11_Figure_2.jpeg
.. |image15| image:: _page_11_Figure_7.jpeg
.. |image16| image:: _page_11_Figure_15.jpeg
.. |image17| image:: _page_13_Figure_0.jpeg

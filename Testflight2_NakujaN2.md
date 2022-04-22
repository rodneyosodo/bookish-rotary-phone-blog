# Avionics report

As we are the brains of the rocket we have some tasks we want to achieve at the end of each launch:

- Log flight data
- Transmit flight data
- Check for apogee
- Fire parachute
- Build the flight computer

## 1. Log flight data

We were not able to equip the rocket with an SD card is it was throwing some errors just before flight. We had tested it and it worked previously. So we didn't want to take more time debugging the problem as we had a backup for that. If we could be able to transmit the data well and good. As we would be able to log the data on the ground station.

### Recommendations

- Test the SD card and make sure it is working prior to launch
- Figure out a way to write to the internal flash system of the rocket

## 2. Transmit flight data
We were able to transmit flight data from the rocket to the ground station. We used the Lora module and we were able to send a packet back to the base station

### Recommendations
- Beef up the base station as to not rely on Putty and borrow Propulsion flight logging
- Reduce payload size for transmission
- Handle queue properly. This is due to the fact that tasks in core 0 run 4 times faster than tasks in core 1, which is sending data over Lora. We would need the Lora task to send all the 4 data points pushed to the queue as the current implementation only sent one and read the other 3 out of the queue. This was to remove queue full error
- Change the packet type to binary as some of the text was not UTF-8 encoded. Also, we need to do a parity check to ensure what we are sending is what we are receiving. This would necessitate an encoder and decoder algorithm
- Our last packet sent was in flight. We don't know why it didn't send up to the time we had an impact

```json=
{"counter":1074283032,"sensor altitude":1704.048,"ax":0.269,"ay":-1.459,"az":-10.051,"gx":-0.155,"gy":-5.172,"gz":-0.117,"filtered s":1711.125,"filtered v":-176.426,"filtered a":-41.354,"state":0,"gps altitude"n\BA
7\A52\C8\E4\ED\A2S\F4\FC\BE\98\98vsU̬\BC\C9\00'\81\CB,\FE<\CD\95\EF\AA\iü
```

Some of the good packets sent during the flight were

```json=
{"counter":1074283032,"sensor altitude":1776.631,"ax":0.896,"ay":-0.642,"az":-10.523,"gx":0.267,"gy":-2.070,"gz":-0.320,"filtered s":1870.299,"filtered v":-52.403,"filtered a":-34.547,"state":0,"gps altitude":0.000,"longitude":1.97082710,"latitude":-0.000
```

## 3. Check for apogee
We were not able to detect apogee as our last packet still read we were on state 0. This was due to a few errors on the check state as we were to require burnout time from the propulsion team.

### Recommendations
- Analyse the check state
- Implement a reverse shell to initiate the firing of the parachute as a backup
- Not rely on time but on altitude, velocity and acceleration from our sensors

## 4. Fire parachute
When we did a partial inspection of the avionics system. We saw the ejection charge had been fired. This doesn't correspond to our telemetry data. We will investigate why the two have disparities and try to correct them. The ejection was fired but the parachute wasn't ejected. This might be attributed to the fact that the parachute provided more force hence the ejection charge ejected towards the avionics bay.

### Recommendations
- Test the parachute ejection continuously before launch
- Introduce a redundancy mechanism e.g watchdog timer

## 5. Build the flight computer

We were able to build the flight computer. The one from gearbox was poorly done so we resorted to using etching to fabricate one. Unfortunately, we still had the problems of wires creeping over.

This is the Fabricated PCB

![Fabricated PCB](https://i.imgur.com/IVscvDK.jpg)

This is the assembled PCB

![Assembled PCB](https://i.imgur.com/LNHA1Go.jpg)

Avionics bay assembly

![](https://i.imgur.com/VRjKpRa.jpg)


![](https://i.imgur.com/icKEORD.jpg)


### Recommendations
- try and build the computer days before launch so as to run as many as possible tests on it.

## Flight data

Filtered altitude from the Kalman filter

![](https://i.imgur.com/IlxjAiW.png)

The actual sensor readings from BMP

![](https://i.imgur.com/MRMH8wW.png)

Altitude velocity and acceleration data

![](https://i.imgur.com/Eeey2TT.png)

From this data, we were to have a high chance of detecting apogee as the velocity curve moves from the +ve region to the -ve region nicely. We are using velocity to detect apogee and we have given it a buffer of +1 to -1.

# Airframe report

As we are the fuselage of the rocket we aim to achieve the following tasks before launch:

- Build the fuselage of the rocket
- Integrate all subsystems
- Do recovery mechanisms

## 1. Build the fuselage of the rocket
We were able to build the fuselage of the rocket. We build the body tube attached the fins using glass fibre and also attach the nose cone.

### Recommendations
- Secure the nose cone properly to avoid entry into the rocket. As we landed the nose cone went inside the body tube pressing onto the components inside

## 2. Integrate all subsystems

We were able to integrate all the subsystems of the rocket. We used PVC tubing to secure the avionics and flight control components to the rocket. 

### Recommendations
- Add a way in which avionics can power their board before entry into the fuselage
- Assemble components one day to launch

## 3. Do recovery mechanisms

We were not able to recover the rocket after apogee. This might be due to the fact that the parachute exerted more force hence the explosion didn't let the parachute out. The rocket dug into the ground at impact.

### Recommendations
- Test the recovery system days before launch
- Think of a new way to recover the rocket


# General

- We were able to integrate all the subsystems better than on the first test flight.
- We were not keen on timing and we ended up launching in the evening.
- We took wind measurements before and the FC gave permission to go for launch
- We didn't do an openrocket final simulation before launch.
- We had a checklist for each subdivision and every subdivision passed the checklist
- We were not able to arm the rocket we flight control as we haven't integrated the flight control code into the avionics code.
- We observed the launch sequence to the latter
- The RSO ensured that all people were at a safe distance of 50m from the launch site
- Ignition worked well but we aim for remote ignition
- We achieved an apogee of 280m from the launch.
- We were not able to deploy parachute hence we didn't have good recovery means
- Flight commander will give more details on what went through the launch.


# Data

1. [Filtered data points](https://github.com/nakujaproject/N2flightreport/blob/main/Testflight2/filtered_data.csv)
2. [Raw data points](https://github.com/nakujaproject/N2flightreport/blob/main/Testflight2/rawdata.log)
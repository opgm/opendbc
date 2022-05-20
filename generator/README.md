# Setting up generated DBC

1. Create brand folder in generator/ directory if it doesn't exist (ex: gm)
2. Extract the common elements from the original DBC and add to a file begining with an underscore (ex: `_comma.dbc`)
3. For each variation, add the name without beginning underscore
    each of these will be parsed, and a `{name}_generated.dbc` will be created
4. At the beginning of the variant files, add the includes
```CM_ "IMPORT {common_filename}";```
5. Run `generator/generator.py`


# Pedal Interceptor signal Template
DBC files can include a scale and offset - these are used
by the Pedal commands to scale the RAW ADC values to the range 0-255
(This is to align with typical stock gas pedal position values)

To include support foor the Pedal Interceptor, add the file `_comma.dbc` to the `generated/{brand}` folder.
In the dbc files for which you want to add the messages, add the line:
```CM_ "IMPORT _comma.dbc";```
To the top of the file.

Finally, copy and paste this template - replacing the scale and offset placeholders with the numeric values.

In this template, you must replace {scale_n} and {offset_n} with numbers calculated in the next section.
```
BO_ 512 GAS_COMMAND: 6 NEO
 SG_ GAS_COMMAND : 7|16@0+ ({scale_1},{offset_1}) [0|0] "" INTERCEPTOR
 SG_ GAS_COMMAND2 : 23|16@0+ ({scale_2},{offset_2}) [0|0] "" INTERCEPTOR
 SG_ ENABLE : 39|1@0+ (1,0) [0|1] "" INTERCEPTOR
 SG_ COUNTER_PEDAL : 35|4@0+ (1,0) [0|15] "" INTERCEPTOR
 SG_ CHECKSUM_PEDAL : 47|8@0+ (1,0) [0|255] "" INTERCEPTOR

BO_ 513 GAS_SENSOR: 6 INTERCEPTOR
 SG_ INTERCEPTOR_GAS : 7|16@0+ ({scale_1},{offset_1}) [0|0] "" NEO
 SG_ INTERCEPTOR_GAS2 : 23|16@0+ ({scale_2},{offset_2}) [0|0] "" NEO
 SG_ STATE : 39|4@0+ (1,0) [0|15] "" NEO
 SG_ COUNTER_PEDAL : 35|4@0+ (1,0) [0|15] "" NEO
 SG_ CHECKSUM_PEDAL : 47|8@0+ (1,0) [0|255] "" NEO

VAL_ 513 STATE 6 "INVALID" 5 "FAULT_TIMEOUT" 4 "FAULT_STARTUP" 3 "FAULT_SCE" 2 "FAULT_SEND" 1 "FAULT_BAD_CHECKSUM" 0 "NO_FAULT" ;
```


# Calculating Pedal Interceptor DBC Transformation

## Collecting Raw ADC / DAC min and max values
1. First, install the Pedal Interceptor and attach to the Comma device. Start the car, wait for OP to start
running - dashcam mode is fine.
2. Wait at least 5 seconds, then slowly and linearly press the gas pedal to the floor. Hold the pedal fully depressed for 5 seconds
3. Slowly release the pedal. Then repeat a couple times
4. Shut off the car
5. (While connected to wifi) Go to connect.comma.ai
6. Wait a few and upload the rlogs. Wait for the upload to complete
7. Open Cabana. Load DBC - select upload, paste the following
This is the Pedal DBC with no transform, allowing you to see the raw values
```
BO_ 513 GAS_SENSOR: 6 INTERCEPTOR
 SG_ INTERCEPTOR_GAS : 7|16@0+ (1,0) [0|0] "" NEO
 SG_ INTERCEPTOR_GAS2 : 23|16@0+ (1,0) [0|0] "" NEO
 SG_ STATE : 39|4@0+ (1,0) [0|15] "" NEO
 SG_ COUNTER_PEDAL : 35|4@0+ (1,0) [0|15] "" NEO
 SG_ CHECKSUM_PEDAL : 47|8@0+ (1,0) [0|255] "" NEO

VAL_ 513 STATE 6 "INVALID" 5 "FAULT_TIMEOUT" 4 "FAULT_STARTUP" 3 "FAULT_SCE" 2 "FAULT_SEND" 1 "FAULT_BAD_CHECKSUM" 0 "NO_FAULT" ;
```
8. Search for "GAS_SENSOR", select the signal. In the middle, expand a message and chart "INTERCEPTOR_GAS" and "INTERCEPTOR_GAS2"
9. Carefully analyze the chart SEPARATELY FOR THE TWO SIGNALS. For each of them, select the approx value for MIN and MAX
   Note: You can select a section on the grach to zoom in. It will be noisy - select the most common value.
   You should now have min_1, max_1; and min2_max2.

## Calculate scale and offset:

```scale = 255 / (max-min)```
This will probably be a number between 0 and 1 - calculate to 9 decimal points (eg 0.012534339)

```offset = -((255-min) / (max-min))```
This is usually a larger negative number with many decimal points (eg -59.012234231)

**Remember to calculate these twice - once for each signal. You will have 4 numbers**

Now update the the full template above with your values for {scale_1}, {offset_1}, {scale_2} and {offset_2}

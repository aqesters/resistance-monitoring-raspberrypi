# load-resistance-monitoring
Monitor load health based on calculated resistance 

## Materials
- 1x Raspberry Pi 4
- 1x ADS1115 ADC
- 1x Chanzon 0.1 Ohm metal film resistor, 1% tolerance, power rating of 2 W
- 3x Cutequeen 10 Ohm metal film resistor, 1% tolerance, power rating of 0.25 W
- Breadboard and wires to construct the circuit

## Driving the load
The load is driven at different levels, and the load resistance is measured and compared to nominal values as an indicator of load health. 
To drive the load, either a current source or voltage source can be used. 

In the provided example, a DC power supply is operated in constant-current mode. Since the power supply does not have a programming interface, the current is adjusted manually, and the voltage drops across the current sense resistor and load resistor are measured using a Raspberry Pi 4 and an ADS1115 16-bit ADC. The load is a Cutequeen 10 Ohm metal film resistor with a rated tolerance of 1% and a power rating of 0.25 W. The current sense resistor is a Chanzon 0.1 Ohm metal film resistor with a rated tolerance of 1% and a power rating of 2 W.

## Calculating electrical current
Current is calculated by Ohm's Law using the measured voltage drop across the current sense resistor and the nominal resistance of that resistor. Unlike the load resistor, the current sense resistor is being operated at well below its max power rating (2W), so it is much more reliable for measuring current.

## Data collection
Using the script `src/monitor_adc.py` on the Raspberry Pi 4, the current sense voltage and load voltage are measured continuously via an ADC while the current drive is manually increased on the DC power supply. The script saves the results to CSV files containing info on timestamps, current sense voltage, load voltage, and current (calculated). The ADC gain and offset are determined empirically. 

I2C must be enabled on the Raspberry Pi in order for this script to work. 

## Post-processing
Once voltage and current are measured for multiple resistors, a separate script `src/plot_resistor_data.ipynb` is used to post-process and plot the data from multiple CSV files. Post-processing includes the following steps:

- Read all CSV files into separate DataFrame objects.
- Remove all rows after the first negative current value, which is an erroneous value that shows up when the ADC input voltage exceeds the max level supported by the ADC.
- Calculate nominal load voltage, load resistance, and power dissipation at the load. Add these as new columns to the DataFrame.

## Plotting
The same script supports plotting the data with non-normalized or normalized axes. In this example, multiple 10-Ohm resistors from the same package are studied. 

Note the sharp increase in resistance at the end. I believe that these are not real measurements, but erroneous values caused when the load voltage is above the max input level of the ADC. This is supported by the fact that the ADC produces non-sensical values (negative voltage values) soon after this resistance spike occurs.

![Non-normalized plot](images/unnormalized_plot.png)

![Normalized plot](images/normalized_plot.png)

## Experiment conclusion
At load powers at 1-5x max rating, resistor temperature increases significantly. The load is a metal film resistor, which has a small positive temperature coefficient. Therefore, resistance is expected to increase with temperature (or remain relatively constant in the range of this experiment). However, I've observed a decrease in resistance as power dissipation increases, which suggests that another phenomenon is occurring.

The following are all possible reasons for the observed resistance decrease. 
### Resistor breakdown 
A breakdown within the resistor element could lead to this deviation from nominal behavior. For example, high temperatures could cause carbonization and electrical shorts to form inside the resistive element, thus shortening the resistive path and decreasing resistance. 

Follow-up: Measure the resistance of the resistors used in this experiment at room temp. If they are significantly different from the starting resistances, then the resistors were likely permanent damaged during the experiment.

### ADC temperature sensitivity
The heat from the resistor conducts through the long copper leads and copper wires leading to the ADC input. The increase in temperature could affect the accuracy of the ADC itself. 

Follow-up: Check the ADC datasheet for temperature sensitivity information.

## Next steps
- Add figures to show hardware setup
- Measure resistance afterwards to see if damage is permanent
- Repeat experiment with temperature sensors to gather more info on failure modes

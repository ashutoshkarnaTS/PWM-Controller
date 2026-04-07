# PWM-Controller
A customized solenoid duty controller design

This board reads a 7-bit binary duty cycle command (0–64%) from a Mitsubishi PLC through 8 optocoupler-isolated digital inputs at 24V, and generates a 300Hz PWM signal to drive a 12V solenoid valve (OCV, 1.3A max) through an external MOSFET. DI1–DI7 set the duty percentage using binary weighting (1, 2, 4, 8, 16, 32, 64), DI8 is the master enable, and the output is capped at 64% duty. It also monitors solenoid current through a sense resistor and signals a fault to the PLC via a digital output if the solenoid is disconnected. A small OLED display shows the live duty percentage and fault status.

# DupliCap
Automatic conversion of the internal representation of a capacitor V3.14
## 1. Purpose of This Program

**dupliCap** is a support tool for simulating manufacturer-provided detailed capacitor models in LTspice and transferring their characteristics into LTspice's internal capacitor representation model.

Manufacturer detailed models are precise, but they can become heavy in full-circuit simulations and may significantly increase analysis time.  
This program derives equivalent internal-representation parameters from the frequency characteristics of the manufacturer’s detailed model, enabling the creation of a lighter and faster internal representation model.

The main objectives of this project are as follows:

- Read manufacturer-provided detailed models
- Automatically extract information such as capacitance, voltage rating, and series name from the part number
- Simulate the detailed model in LTspice
- Match parameters on the internal representation side, such as ESR and ESL
- Output the results as a list file for use in downstream processing

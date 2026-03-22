# DupliCap
Automatic conversion of the internal representation of a capacitor V3.14

# dupliCap User Guide

## 1. Purpose of This Program

**dupliCap** is a support tool for simulating manufacturer-provided detailed capacitor models in LTspice and transferring their characteristics into LTspice's internal capacitor representation model.

Manufacturer detailed models are precise, but they can become heavy in full-circuit simulations, resulting in longer analysis times.  
This program derives equivalent parameters for LTspice's internal representation from the frequency characteristics of the manufacturer’s detailed model, allowing the creation of a lighter and faster internal representation model.

The main objectives of this project are as follows:

- Read manufacturer-provided detailed models
- Automatically extract information such as capacitance, voltage rating, and series from the part number
- Simulate the detailed model in LTspice
- Fit internal representation parameters such as ESR and ESL
- Output the results as a list file for downstream use

## 2. Program Structure

### 2.1 JSON Generation Scripts

These scripts define manufacturer-specific part number rules and generate JSON files for part number decoding.

- `Create_murata_grm_json.py`
- `Create_nichemi_json.py`
- `Create_nichicon_json.py`

These scripts define the meaning of each character position in the part number.  
For example, they are designed to extract information such as:

- Series name
- Dielectric type or product type
- Rated voltage
- Capacitance
- Tolerance
- Additional information such as size and package shape

The generated JSON files are used later for part number decoding and validity checking.

### 2.2 Part Number Decoding and Validation

- `model_number_decoding.py`

This program reads the rules defined in the JSON file and parses part number information from the `.SUBCKT` name of the manufacturer’s detailed model.

Its main purposes are:

- Extract fields such as `Series`, `kind`, `volt`, `value`, and `tol` from the part number
- Verify whether the JSON definition is valid for the actual model set

If there is an error in the JSON definition, the program displays which item does not match.

### 2.3 Main Internal-Representation Conversion Program

- `dupliCap.py`

This is the main program.  
According to the settings specified in the LTspice schematic or netlist, it sequentially reads the manufacturer’s detailed models in the target folder, calculates the parameters for the internal representation, and outputs a result list.

## 3. Overall Workflow

The basic workflow of this project is as follows:

1. Edit the manufacturer-specific JSON generation script  
   Write the part number decoding definition at the beginning of the script.

2. Run the JSON generation script  
   This creates a manufacturer-specific JSON file.

Examples:
- `murataGRM.json`
- `nichemi.json`
- `nichicon.json`

3. Run `model_number_decoding.py` for validity checking  
   Specify the generated JSON and the target model folder to confirm that the part numbers are decoded correctly.  
   Definition errors and unsupported codes can be found here.

4. Run `dupliCap.py` to perform internal-representation conversion  
   The models in the target folder specified on the schematic side are simulated one by one, and the internal representation parameters are calculated.

5. Check the output text file  
   Confirm the result file listing capacitance, manufacturer name, part number, type, voltage rating, ESR, ESL, and so on.

## 4. Preparation

### 4.1 Required Items

- LTspice
- Python execution environment
- `spicelib`
- A complete set of manufacturer detailed models
- JSON definition file for part number decoding
- LTspice schematic for conversion

### 4.2 `Autopy.ini`

`dupliCap.py` reads `Autopy.ini` to obtain the location of the LTspice executable file.  
Its contents are written in JSON format.

## 5. How to Use `Create_murata_grm_json.py` / `Create_nichemi_json.py` / `Create_nichicon_json.py`

### 5.1 What These Programs Do

These programs define manufacturer-specific part number rules and create JSON files for part number decoding.

At the beginning of each script, the decomposition rules for the manufacturer’s part number are defined in `pd['form']`.  
The definition format is generally as follows:

- `[start_position, end_position]`
- `[start_position, end_position, conversion_table]`
- `[start_position, end_position, unit_factor]`

### 5.2 Execution Result

When the script is executed, the corresponding JSON file is generated.

- `Create_murata_grm_json.py` -> `murataGRM.json`
- `Create_nichemi_json.py` -> `nichemi.json`
- `Create_nichicon_json.py` -> `nichicon.json`

### 5.3 Notes

If the part number rules are incorrect, the subsequent validation step will report errors.  
Whenever you add a new series or an unsupported code, always run the validation check.

## 6. How to Use `model_number_decoding.py`

### 6.1 Purpose

- Validation of JSON definitions
- Verification of part number decoding from `.SUBCKT` names

### 6.2 Input

- 1st argument: JSON file
- 2nd argument: Folder containing manufacturer detailed models

If no arguments are specified, you can choose them using a file selection dialog and a folder selection dialog.

### 6.3 Example

```bash
python .\model_number_decoding.py .\murataGRM.json ..\Capacitor_Select\murata_grm

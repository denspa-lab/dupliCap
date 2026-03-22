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
````

### 6.4 Validation Details

The program reads the `.SUBCKT` name from each model file and parses it according to the JSON rules.
Under normal conditions, the following information is displayed:

* `Series`
* `kind`
* `value`
* `volt`
* `tol`

In abnormal cases, the result is displayed as `NG`. The main error types are:

* `short`: insufficient part number length
* `table`: not registered in the conversion table
* `numeric`: numeric conversion failed
* `no_subckt`: `.SUBCKT` not found

## 7. How to Use `dupliCap.py`

### 7.1 Purpose

It simulates the manufacturer’s detailed models one by one and determines the parameters for the internal representation model.

### 7.2 Operation Overview

`dupliCap.py` performs the following steps:

1. Read the `Python:` directives from the schematic or netlist
2. Decode the part number using the specified JSON file
3. Enumerate the model files in the target folder
4. Read the `.SUBCKT` name of each model
5. Measure the characteristics of the manufacturer’s detailed model in the first simulation
6. Set the internal representation values `R1` and `L1` based on the measurement results
7. Confirm the degree of match in the second simulation
8. Output the result list to a text file

### 7.3 Required Settings on the Schematic Side

In the conversion schematic, write the `Python:` directives as comments.
In the current implementation, at least the following items are used:

* `Python:exec=dupliCap`
* `Python:debug=0`
* `Python:in_path=...`
* `Python:out_file=...`
* `Python:def_file=...`
* `Python:Mfg=...`
* `Python:max_wait=...`

Their main meanings are as follows:

* `in_path`: target model folder
* `out_file`: result output file name
* `def_file`: JSON file for part number decoding
* `Mfg`: manufacturer name for output
* `debug`: plot display mode
* `max_wait`: plot wait time

### 7.4 Meaning of `debug`

* `0`: no plot
* `1`: display result plots
* `2`: display including the first simulation

You can switch this during execution using the keyboard.

* `0`, `1`, `2`: change display mode from the next model onward
* `Esc`: interrupt execution

### 7.5 Execution Flow

When executed, the program reads the list of models in the target folder and displays the decoded part number information.
After that, if `Y` is entered at the confirmation prompt, the conversion starts.

For each model, the following steps are performed:

* Set the manufacturer detailed model part name to `XU1`
* Set the decoded capacitance to `C1`
* Obtain ESR- and ESL-equivalent values in the first simulation
* Set those values to `R1` and `L1`
* Re-simulate and check the degree of match
* Add the result to the output list

### 7.6 Output File

The results are saved to the text file specified by `out_file`.
In the current implementation, the output columns are as follows:

1. Capacitance `[uF]`
2. Manufacturer name
3. Part number
4. Type
5. Voltage rating
6. Reserved column
7. ESR
8. ESL `[nH]`

Output example:

```text
0.220000    Den_murata    GRM21AR72A224KAC5    X7R    100    0    1.691024e-02    4.548683e-01
```

## 8. Notes

* If the JSON definition does not match the actual model, part number decoding will fail.
* The target model file must contain a `.SUBCKT` definition.
* `dupliCap.py` requires a properly prepared LTspice execution environment and conversion schematic.
* On the schematic side, this program assumes that it can rewrite `XU1`, `C1`, `R1`, and `L1`.
* If `Esc` is pressed during execution, the program can save the results obtained up to that point and exit.

## 9. Typical Operating Procedure

### Murata Case

1. Update the definition in `Create_murata_grm_json.py`
2. Run the script to generate `murataGRM.json`
3. Validate the Murata model folder with `model_number_decoding.py`
4. Open the Murata conversion schematic
5. Run `dupliCap.py`
6. Check `murata_capdupli.txt`

The same applies to Nichemi and Nichicon.

## 10. Benefits of This Program

* It reduces the weight compared with using manufacturer detailed models directly
* It shortens full-circuit simulation time
* It allows large numbers of manufacturer models to be converted into internal representation format using a unified rule
* It enables management of part numbers and conversion results in a list format

## 11. How to Use the Output Text File

The results obtained by this program are output as a text file, and this content is in a format that can be pasted directly into LTspice user model registration.

Therefore, users are encouraged to add the contents of the output text file to LTspice user part registration.
By registering them in advance, the internally converted capacitor models can be referenced more easily in LTspice.


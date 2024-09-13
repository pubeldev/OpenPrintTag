# Prusa Material Data Format

The Prusa Meterial Data Format project aims to:

1. Specify relevant properties of 3D printing materials (FDM, SLA, ...) and define their standardized terminology, units and data types.
1. Define a data format for material tags – storing information about materials on their packages (filament spools, resin containers, ...).

### Site map

* [Terminology](terminology.md)
* [**Format specification for NFC tags**](nfc_data_format.md) (static & dynamic information)
* Format specification for QR codes (static information only)
* [Examples](examples.md)

### Use cases for the data tags

1. Automation/user comfort
	 * When user loads a new filament in the printer, the printer can scan the tag and load up the material information (material type, color, ...) automatically without prompting the user.
2. Material usage tracking (dynamic information)
	 * The printers can use the data tags to monitor material usage and for example warn user when there is not enough material to print a given model.
3. Inventory management
	 * By putting a data tag with an unique ID in a machine readable format on every material package, managing the inventory for warehouses/print farms can become more automated an efficient.

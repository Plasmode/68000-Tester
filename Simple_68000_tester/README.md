This folder contains information for simple tester configuration without CF or MC68681

![Simple 68000 Tester](https://github.com/user-attachments/assets/c50d4b56-f52f-4c88-a139-45b2b9e4f67a)

### Features
- Tester for DIP64 68000
- 256K RAM
- EPM7064S CPLD
  - Bootstrap ROM
  - Serial port
  - Address decode
- Bootstrap via serial port
- Economical 100mm X 100mm 2-layer pc board

### Design files
- [Schematic](68KTSTR_scm.pdf)
- [Gerber photoplots](68KTSTR_R0_Gerber.zip)
- Bill of materials
- [EPM7064S CPLD design](68000tester_serial_load.zip) files
  - CPLD design in [PDF schematic](CPLD_top_serial_boot_scm.pdf)

### Software
[Serial loading process](Software/Serial_Load_process.md)
- [128K RAM test](Software/RAMtest128K.zip)


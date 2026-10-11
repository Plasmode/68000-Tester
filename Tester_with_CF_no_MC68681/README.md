This folder contains information for configuration of Tester-with-CF-no-MC68681.

![68000 Tester with CF interface](https://github.com/user-attachments/assets/5c046f65-3833-4fdf-a91c-bd76b4c1642f)

This is a 68000 tester with ZIF socket, 2x512KB RAM, and CF bootstrap testing one of the ceramic/gold 68000 I have.  Moving from serial bootstrap to CF bootstrap involves several steps.  First is changing the ROM in CPLD to monitor CF Busy status and read the default sector (Master Boot Record) into RAM.  Second is switching off ROM and executing the program loaded into RAM, and third, load additional software into RAM.  The screen capture shows my own monitor along with TUTOR v1.3 are loaded into RAM in third step.
The CF interface is 16-bit wide and source terminated with 100 ohm resistors on all data lines.  It also has 100ohm/100pF filter network on the CF read strobe to further control the ground bounce.  I also accumulated and displayed checksum to monitor the integrity of CF transfer.  I learned to use srec_cat.exe from Wayne Warthen and built the CF image composed of MBR bootloader, TUTOR, and my own monitor.  CF bootstrap is fast and convenient; it seems reliable with several different brands of CF disks.

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
- EPM7064S CPLD design files
  - [PDF schematic](Schematic_of_CPLD_for_Tester_with_CF_no_MC68681.pdf) of CPLD design without MC68681

### Software

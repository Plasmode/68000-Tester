This folder contains information for Tester-with-MC68681 configuration


![68000 tester with CF and MC68681 DUART](https://github.com/user-attachments/assets/28ca767c-593c-4348-8daa-109eed5526b3)

The 68000 tester has evolved from the bare bones serial-bootstrap configuration to CF-bootstrap one-megabyte RAM configuration and now to the external DUART serial port 68000 SBC.  The operation principle is still same as the CF bootstrap configuration, i.e., 68000 executes the ROM inside CPLD after reset which loads/executes the program in CF's Master Boot Record, which, in turn, loads monitor and TUTOR into RAM.  The monitor and TUTOR have been modified to use the external MC68681 serial port.  
 
There are several benefits associated with the external MC68681:
1.  MC68681 has dual serial channels as well as a number of discrete input and output.  It also has timers and robust interrupt capabilities, so it is a significantly more versatile IO device.
2.  MC68681 has independent baud clock so it is decoupled from 68000's system clock.  This means the tester can have different system clock to check out faster or slower 68000.  In fact, I did an overclock test and found the board with 10MHz 68HC000 is capable of 20MHz operation.
3.  Without the embedded serial function, simpler and cheaper CPLD can be used.  In this case a 32-macrocell CPLD, EPM7032S, is used which is equivalent to ATF1502.

The MC68681 configuration has several design mistakes around the interrupt and DTACK functions so requires some modifications.  They are fairly minor and easily fixed with a new board revision.

68k_tester_CPLD_with_MC68681 is the CPLD design file for Tester-with-MC68681 configuration.

Software folder contains software for tester_with_MC68681 configuration

### Features
- Tester for DIP64 68000
- 256K RAM
- EPM7032S CPLD
  - Bootstrap ROM
  - Serial port
  - Address decode
- Bootstrap via serial port
- Economical 100mm X 100mm 2-layer pc board

### Design files
- [Schematic](68KTSTR_with68681_scm.pdf)
- [Gerber photoplots](68KTSTR_R0_Gerber.zip)
- Bill of materials
- EPM7064S [CPLD design files](68K_tester_CPLD_With68681.zip)
  - [PDF schematic](Schematic%20of%20CPLD_for_Tester_with_mc68681.pdf) of CPLD design with 68681

### Software
- [Monitor](Software/68KMon_CF_68681_V04a.zip) for 68000-tester with MC68681
- [EhBasic](Software/Basic68s3.52.S68) ported to 68000-tester with MC68681
- [TUTOR v1.3](Software/TUTOR_EASy68K.S68) ported to 68000-tester with MC68681
- [CP/M68K BDOS, CCP, BIOS](Software/cpm68k_for_tester_with_MC68681.zip) for 68000-tester with MC68681
- CF [disk layout](Software/68KTSTR_CF_layout.txt) of applications
- CF [disk image](Software/image_MC68681.img)
- [Source files](Software/68KTSTR_with_68681_sources_for_creating_CF_image.zip) for creating above CF disk image



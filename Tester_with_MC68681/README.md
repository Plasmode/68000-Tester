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

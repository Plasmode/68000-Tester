# 68000-Tester
This is a simple tester to check out 68000 in DIP64 package.  The original goal of this 68000 tester is a simple, inexpensive board to test 68000 and 68010 in DIP64 packages.  The tester is so simple, there are rooms available to add additional features for a more general purpose 68000 SBC.  The final design resulted in a PC board with multiple configurations including a general purpose 68000 SBC. The design is described in details in two on-line forums: 
* Google forum, https://groups.google.com/g/retro-comp/c/AY7YPEINyhw
* VCFED forum, https://forum.vcfed.org/index.php?threads/a-simple-68000-tester.1256068/

The original motivation for the 68000 Tester was an inexpensive 68000 tester for couple dozens of unknown ceramic/gold DIP64 68000 that I had for many years.  The tester was designed with space for a large 64-pin ZIF socket, and I want the tester to be simple and inexpensive, just enough functions to serially load & run test programs.  The PC board should be 2-layer, 100mmX100mm to keep the cost low.  To simplify assembly and keep part count low, a small 44-pin, 64-macrocell CPLD was used as ROM, serial port, and glue logic so the tester wa consisted of a CPLD and two inexpensive 128Kx8 RAM.

64-macrocell CPLD has limited capacity for ROM and logic; it can support 64 bytes of ROM and enough logic for a serial shift register and buffer for the serial receiver; serial transmitter is just a latch for bit-bang transmit function.

After reset the 68000 executes the bootstrap program in ROM which monitors the serial receive status bit for incoming data.  The incoming data is stored in RAM after the end of ROM program.  When 256 bytes of serial data is received, the ROM is disabled and 68000 executes the 256-byte program that was uploaded in RAM.  This 256-byte program can be memory test, diagnostic, or more sophiscated serial loader to load larger test programs.  This is the basis of operation for the simple 68000 tester.

![68000 tester pc board](https://github.com/user-attachments/assets/617cb28c-eb64-4827-bf28-13880c9ed9a5)
68000 tester blank pc board

![Simple 68000 Tester](https://github.com/user-attachments/assets/2b4201b0-60ff-4e35-8b47-57ff7b9840b6)
[Simple 68000 Tester](Simple_68000_tester) in its simplest configuration with 2x128K RAM and a 64-macrocell CPLD


![68000 Tester with CF interface](https://github.com/user-attachments/assets/4e3732a7-7c46-4507-9ba6-7e67cfbfc795)
Simple 68000 Tester plus a CF interface that allow the board to be bootstrap from a CF disk.  The simple 68000 tester does not fill up a 100mmX100mm pc board.  There is room available to add additional features to enhance the tester operation.  The larger 512Kx8 RAM is pin compatible with 128Kx8 RAM so it is easy to design the board for both 128K as well as 512K RAM.  Another enhancement is an IDE interface to accommodate Compact Flash disk.  Having CF function opens the possibility of bootstrapping from CF disk and load/execute test programs quickly and automatically.   With larger RAM and CF interface, the CPLD ROM is modified to initialize the CF disk and load the 256-byte program in Master Boot Record of the CF disk, which, in turn, loads other programs stored in the CF disk.  This is the basis of operation for the [68000 tester with CF](Tester_with_CF_no_MC68681). 


![68000 tester with CF and MC68681 DUART](https://github.com/user-attachments/assets/384b0a12-032f-47df-bded-772c98ff558d)
CF circuitry of the enhanced 68000 tester does not take much room; there are PC board space available for additional circuitry such as the optional MC68681.  Tester with MC68681 becomes a versatile 68000 SBC.  Since the serial function is moved to a separate I/O device, the CPLD becomes quite simple such that it can be replaced with a smaller, less expensive 32-macrocell CPLD such as EPM7032S or its equivalent ATF1502.   The additional cost of MC68681 is offset by the lower cost CPLD.  ["Tester with MC68681"](Tester_with_MC68681) is the third configurations of 68000 tester.  This is a general purpose 68000 SBC.

The three configurations are described in details in three folders under this project.

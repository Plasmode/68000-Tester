# Simple 68000 Tester Serial Load Process
2/5/26

ROM program inside CPLD, SerLoad (quartwrk81\HomeProjects\68KTSTR\archive\SerialLoad):
SerLoad is converted to Verilog file, SerLoad.v
```
00000000                             9           org $0
00000000= 00000000                  10           dc.l 0                              ;stack
00000004= 00000008                  11           dc.l start
00000008                            12  start:
00000008  11FC 0000 FF13            13           move.b #0,SerTx                     ;send out prompt
0000000E  11FC 00FF FF13            14           move.b #$ff,SerTx                   ;toggle Tx
00000014  43F9 00000034             15           lea RAMhere,a1                      ;move a block of data to $34 (after end of ROM)
0000001A  323C 0100                 16           move.w #$100,d1
0000001E                            17  loadloop:
0000001E  0838 0000 FF11            18           btst.b #0,SerStat                   ;check for RxRdy
00000024  67F8                      19           beq.s loadloop
00000026  12F8 FF13                 20           move.b SerRx,(a1)+
0000002A  51C9 FFF2                 21           dbra d1,loadloop
0000002E  4238 FF1F                 22           clr.b RomDis
00000032  4E71                      23           nop
00000034                            24  RAMhere:                                     ;RAM program starts here   
```
- Serload waiting for 256 bytes of data
- loaded starting from RAM addr $34.
- The program is SRload (easy68k\68KTSTR\SerialLoad\SRload.bin
- SRload.s68 is converted to SRload.bin using Easybin.exe 
- SRload.bin is sent to serial port as binary
```
00000034                            15           org $34
00000034  4E71                      16           nop
00000036  4E71                      17           nop
00000038  4FF9 000FFFF0             18           lea $ffff0,sp                       ;set up stack
0000003E                            19  start:   
0000003E  43F9 0000011A             20           lea signon_,a1             * sign on message
00000044                            21  moreMsg:         
00000044  1019                      22           move.b (a1)+,d0
00000046  6700 0008                 23           beq msgDone
0000004A  6100 00A2                 24           bsr cout
0000004E  60F4                      25           bra.s moreMsg 
00000050                            26  msgDone:         
00000050  4285                      27           clr.l d5
00000052                            28  cmd:
00000052  6172                      29           bsr.s cinq                   *get character input
00000054  B03C 000D                 30           cmp.b #$d,d0                  * carriage return?
00000058  6600 001C                 31           bne doload2
0000005C  4A05                      32           tst.b d5
0000005E  6700 000C                 33           beq doload3
00000062  103C 003F                 34           move.b #'?',d0            *error, put out '?'
00000066  6100 0086                 35           bsr cout
0000006A  60E6                      36           bra cmd
0000006C                            37  doload3:
0000006C  103C 002E                 38           move.b #'.',d0        *no error, put out '.'
00000070  6100 007C                 39           bsr cout
00000074  60DC                      40           bra cmd                  
00000076                            41  doload2:         
00000076  B03C 0053                 42           cmp.b #'S',d0                 *S record?
0000007A  66D6                      43           bne cmd                    *get S record
0000007C                            44  doSload:
0000007C                            45  ;can only handle S0, S1.  Ignore S8
0000007C  6148                      46           bsr.s cinq                 * get 2nd character
0000007E  B03C 0031                 47           cmp.b #'1',d0              *S1 record
00000082  6700 0010                 48           beq doS1
00000086  B03C 0030                 49           cmp.b #'0',d0              *S0 record?
0000008A  67C6                      50           beq cmd                    *discard S0 record
0000008C  4A05                      51           tst.b d5                   *do not jump to $400 if there are errors
0000008E  66AE                      52           bne start
00000090  6000 036E                 53           bra $400                   *jump to $400 for everything else
00000094                            54  
00000094                            55  doS1:
00000094                            56  * 2-byte address field
00000094  4284                      57           clr.l d4
00000096  7401                      58           moveq #1,d2                * get Srecord byte count
00000098  613A                      59           bsr.s char2hex
0000009A  D801                      60           add.b d1,d4                * accumulate checksum
0000009C  3601                      61           move.w d1,d3
0000009E  5943                      62           subq #4,d3                 * don't count CRC byte nor 2-byte address             
000000A0  7403                      63           moveq #3,d2                * get 2-byte starting address
000000A2  6130                      64           bsr.s char2hex
000000A4  2041                      65           move.l d1,a0      
000000A6  D801                      66           add.b d1,d4                * accumulate checksum, lower address
000000A8  E059                      67           ror.w #8,d1                
000000AA  D801                      68           add.b d1,d4                * accumulate checksum, higher address
000000AC                            69  srdata:
000000AC  7401                      70           moveq #1,d2                * get byte data
000000AE  6124                      71           bsr.s char2hex
000000B0  D801                      72           add.b d1,d4                * accumulate checksum
000000B2  10C1                      73           move.b d1,(a0)+            * store value to destination
000000B4                            74  ;no write verify, out of time       
000000B4                            75  ;        cmp.b (a0)+,d1             * write verify
000000B4                            76  ;        beq srdatax
000000B4                            77  ;        moveq #1,d5
000000B4                            78  ;srdatax:         
000000B4  51CB FFF6                 79           dbra d3,srdata
000000B8  4604                      80           not.b d4                   * checksum expected
000000BA  7401                      81           moveq #1,d2
000000BC  6116                      82           bsr.s char2hex
000000BE  B801                      83           cmp.b d1,d4
000000C0  6790                      84           beq cmd
000000C2  7A01                      85           moveq #1,d5                * send out a question mark
000000C4  608C                      86           bra cmd
000000C6                            87  cinq:
000000C6                            88  ;return serial input in regD0       
000000C6  0838 0000 FF11            89           btst.b #0,SerStat                   ;check for Receive Ready
000000CC  67F8                      90           beq.s cinq
000000CE  1038 FF13                 91           move.b SerData,d0 
000000D2  4E75                      92           rts        
000000D4                            93  *
000000D4                            94  * subroutine to get prescribed number of characters in d2+1, convert to hex and put in
000000D4                            95  * d1 
000000D4                            96  char2hex:
000000D4  4281                      97           clr.l d1                   * important to clear all d1 when start
000000D6                            98  c2h1:
000000D6  61EE                      99           bsr cinq
000000D8                           100  ;assuming incoming are 0-9, A-F, no additional checking, relying on checksum for errors
000000D8  B03C 0040                101           cmp.b #'A'-1,d0
000000DC  6302                     102           bls.s aschex3              * 'F'> valid >'A'
000000DE  5F40                     103           subq #$7,d0
000000E0                           104  aschex3:
000000E0  0400 0030                105           sub.b #$30,d0
000000E4  E989                     106           lsl.l #4,d1                
000000E6  D200                     107           add.b d0,d1                * append to existing value
000000E8  51CA FFEC                108           dbra d2,c2h1
000000EC  4E75                     109           rts
000000EE                           110              
000000EE                           111  cout:
000000EE                           112  ;send regD0 to serial port
000000EE                           113  ;bit bang at 115200
000000EE  4282                     114           clr.l d2                   ;d2 is start bit/stop bit
000000F0  11C2 FF13                115           move.b d2,SerData           ;start bit
000000F4  5242                     116           addq #1,d2                 ;initialize stop bit ahead of time  
000000F6  45F9 000FFFFE            117           lea $ffffe,a2               ;delay using movem instruction
000000FC  7207                     118           moveq #7,d1
000000FE  4892 003F                119           movem d0-d5,(a2)
00000102                           120  shift8:
00000102  11C0 FF13                121           move.b d0,SerData          ;bit bang Tx, D[0] connected to TX output              
00000106  E240                     122           asr #1,d0
00000108                           123  cout2:
00000108  4892 007F                124           movem d0-d6,(a2)
0000010C  51C9 FFF4                125           dbra d1,shift8 
00000110  11C2 FF13                126           move.b d2,SerData          ;stop bit       
00000114  4892 000F                127           movem d0-d3,(a2)   
00000118  4E75                     128           rts         
0000011A                           129  
0000011A= 0A 0D 53 2D 72 65 ...    130  signon_: dc.b 10,13,'S-record Loader v0.3',10,13
00000132= 3E 00                    131           dc.b '>',0 
00000134                           132  progEnd:         
```

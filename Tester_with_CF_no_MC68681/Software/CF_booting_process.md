# CF Booting Process
### 68KTSTR booting without 68681
2/6/26
- ROM resides in CPLD is CFBoot.x68 (Easy68k\68KTSTR\CF\CFBoot)
- It is convert to CFBoot.v
```
00000000= 00000000                  12           dc.l 0
00000004= 00000008                  13           dc.l start
00000008                            14  start:
00000008  43F8 FF2E                 15           lea CFstat,a1     
0000000C                            16  start1:
0000000C  0811 0007                 17           btst.b #7,(a1)                      ;check for CF Busy bit
00000010  66FA                      18           bne.s start1
00000012  12BC 0020                 19           move.b #$20,(a1)                    ;issue read MBR sector command
00000016                            20  chkdrq:
00000016  0811 0003                 21           btst.b #3,(a1)                      ;check data request bit set before reading
0000001A  67FA                      22           beq chkdrq
0000001C  41F9 00000034             23           lea RAMstart,a0                     ;load RAM program starting address
00000022  323C 00FF                 24           move.w #$ff,d1                      ;move 256 words from MBR to RAM
00000026                            25  cp2RAM:
00000026  30F8 FF20                 26           move.w CFdata,(a0)+
0000002A  51C9 FFFA                 27           dbra d1,cp2RAM
0000002E  4238 FF1F                 28           clr.b RomDis                        ;page out CPLD ROM
00000032  4E71                      29           nop
00000034                            30  RAMstart:
```
### MBRload
- MBRload resides in CF's MBR (easy68k\68ktstr\cf\mbrload)
- It is converted to MBRload.bin using easybin
- MBRload loads LBA2-LBA40 into memory from $400-$8000
- 68kmon is from $400-$3FFF(EASY68K\68KTSTR\CF\68KMON.X68)
  - while TUTOR is from $4000-$7FFF (easy68k\tutor\68KTSTR\TUTOR-EASY68K.X68)
- checksum is displayed
```
00000034                            16           org $34
00000034  4E71                      17           nop
00000036  4E71                      18           nop
00000038  4FF9 000FFFF0             19           lea $ffff0,sp                       ;set up stack pointer
0000003E  103C 0058                 20           move.b #'X',d0
00000042  6100 00B6                 21           bsr cout 
00000046  11FC 00E0 FF2C            22           move.b #$e0,CF2427                  ;set up LBA mode
0000004C  11FC 0000 FF2A            23           move.b #0,CF1623                    ;set to LBA 2
00000052  11FC 0000 FF28            24           move.b #0,CF815
00000058  47F8 0400                 25           lea $400,a3
0000005C                            26  ;skip over LBA1, monitor program stored start from LBA2        
0000005C  1A3C 0002                 27           move.b #2,d5                        ;d5 keep track of the sector value  
00000060  4244                      28           clr.w d4                            ;d4 keeps track of checksum
00000062                            29  spin:    
00000062  11C5 FF26                 30           move.b d5,CF07
00000066  11FC 0001 FF24            31           move.b #1,CFsectcnt                 ;read one sector       
0000006C  11FC 0020 FF2E            32           move.b #$20,CFstat                  ;read sector command
00000072                            33  chkdrq:
00000072  0838 0003 FF2E            34           btst.b #3,CFstat                    ;check data request bit set before reading
00000078  67F8                      35           beq chkdrq 
0000007A  363C 00FF                 36           move.w #$ff,d3                      ;read 512 bytes
0000007E                            37  readCF1:
0000007E  3038 FF20                 38           move.w CFdata,d0
00000082  D840                      39           add.w d0,d4                         ;accumulate checksum
00000084  36C0                      40           move.w d0,(a3)+                     ;get data and put in destination
00000086  51CB FFF6                 41           dbra d3,readCF1
0000008A  5205                      42           addq.b #1,d5
0000008C  BA3C 0041                 43           cmp.b #$41,d5                       ;load LBA 2-$40, 68KMon, TUTOR
00000090  66D0                      44           bne.s spin
00000092  103C 005A                 45           move.b #'Z',d0
00000096  6100 0062                 46           bsr cout
0000009A  33C4 000FFEDC             47           move.w d4,$ffedc                    ;top of 68KMon variable area
000000A0  1004                      48           move.b d4,d0                        ;print out the checksum
000000A2  6100 0022                 49           bsr hex2asc
000000A6  6100 0052                 50           bsr cout                            ;display the LSB of checksum
000000AA  E058                      51           ror.w #8,d0                         ;display the MLB of checksum
000000AC  6100 004C                 52           bsr cout
000000B0  E05C                      53           ror.w #8,d4
000000B2  1004                      54           move.b d4,d0
000000B4  6100 0010                 55           bsr hex2asc
000000B8  6100 0040                 56           bsr cout                            ;display MUB of checksum
000000BC  E058                      57           ror.w #8,d0                         ;display MSB of checksum
000000BE  6100 003A                 58           bsr cout 
000000C2  4EF8 0400                 59           jmp $400 
000000C6                            60  * hex to ascii
000000C6                            61  * byte value in d0.b is converted to ascii in d0.w
000000C6                            62  hex2asc:
000000C6  48E7 4000                 63           movem.l d1,-(sp)
000000CA  1200                      64           move.b d0,d1
000000CC  E809                      65           lsr.b #4,d1                * get the high nibble first
000000CE  0601 0030                 66           add.b #$30,d1
000000D2  B23C 0039                 67           cmp.b #'9',d1              * '0' to '9'
000000D6  6300 0004                 68           bls hex2asc7
000000DA  5E01                      69           add.b #7,d1                * or 'A' to 'F'
000000DC                            70  hex2asc7:
000000DC  E149                      71           lsl.w #8,d1                * put result in high byte
000000DE  1200                      72           move.b d0,d1               * work on the low nibble
000000E0  C23C 000F                 73           and.b #$F,d1               * isolate low nibble
000000E4  0601 0030                 74           add.b #$30,d1
000000E8  B23C 0039                 75           cmp.b #'9',d1              * '0' to '9'
000000EC  6300 0004                 76           bls hex2asc8
000000F0  5E01                      77           add.b #7,d1                * or 'A' to 'F'
000000F2                            78  hex2asc8:
000000F2  2001                      79           move.l d1,d0               * done
000000F4  4CDF 0002                 80           movem.l (sp)+,d1
000000F8  4E75                      81           rts
000000FA                            82           
000000FA                            83  cout:
000000FA                            84  ;send regD0 to serial port
000000FA                            85  ;bit bang at 115200
000000FA  4282                      86           clr.l d2                   ;d2 is start bit/stop bit
000000FC  11C2 FF13                 87           move.b d2,SerData           ;start bit
00000100  5242                      88           addq #1,d2                 ;initialize stop bit ahead of time  
00000102  45F9 000FFFFC             89           lea $ffffc,a2               ;delay using movem instruction
00000108  7207                      90           moveq #7,d1
0000010A  4892 003F                 91           movem d0-d5,(a2)
0000010E                            92  shift8:
0000010E  11C0 FF13                 93           move.b d0,SerData          ;bit bang Tx, D[0] connected to TX output              
00000112  E200                      94           asr.b #1,d0
00000114                            95  cout2:
00000114  4892 007F                 96           movem d0-d6,(a2)
00000118  51C9 FFF4                 97           dbra d1,shift8 
0000011C  11C2 FF13                 98           move.b d2,SerData          ;stop bit       
00000120  4892 000F                 99           movem d0-d3,(a2)   
00000124  4E75                     100           rts         
00000126                           101           
00000126                           102  CFbuf:
```





# NBResearch
this repository is for my own research about netbios over tcp protocol, and contains all sorts of external resources, in order to help me migrate the work faster from one environment to another.
# Motive
Netbios over tcp is a default protocol on any windows machine, evan in its latest forms. meaning that any security bug that will maybe be found in NetBt.sys / NetBios.sys will effect many operating machines. in a addition such network protocols take arbitrary packets from the outer net and ,(for better efficacy), deal with the data straight in the kernel (the main drivers are listed above). from the latest smb vulnerabilities found in srv.sys (see sleepya work on reversing the 'ethereal-family') we can see that there is big potential in network drivers for RCE's, while the ssesion service over tcp (netbios port-139) takes the user parameters and makes direct analysis over the remote packet data in order to operate. this interesting fact make's a great potential in study'ing this specific protocol, as all we need is an opertunity to control the MDL- for arbitrery write, or better get one pointer for code execution.
# first observation
port 139 (session service) takes in account the data segmant in the remote packet in order to manufacture the return packet, he do that mainlly in Netbios.sys, while from my short expiriance i can tell that when providing that port a short one byte data containing packet the protocol would not return a name and close the socket as usual, but wait for additional data to be recieved from the end port. i will point out that giving the port an extra tail after the 'USER' syntax will let you send up to giga bytes to the server, while this data is not being saved over the session it still hold a sufficant reason to make continue's research on those three protocols, because they simply take data coming out of the network and deal with it inside the kernel, as mentioned earlier all one needs is a cross-border copy in one of the functions inside the network drivers to get arbitrary code-execution.

NORMAL:
<br>
Query:
<br>
<br>0000   00 00 00 01 00 06 a0 ab 1b 5e 44 f2 00 00 08 00  .........^D.....
<br>0010   45 00 00 4e 00 00 40 00 40 11 b7 48 c0 a8 01 01  E..N..@.@..H....
<br>0020   c0 a8 01 05 96 2a 00 89 00 3a a3 4e 03 e8 00 00  .....*...:.N....
<br>0030   00 01 00 00 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01        AAAAAAAAA..!..
<br>
<br>0000   00 00 00 01 00 06 a0 ab 1b 5e 44 f2 00 00 08 00  .........^D.....
<br>0010   45 00 00 4e 00 00 40 00 40 11 b7 48 c0 a8 01 01  E..N..@.@..H....
<br>0020   c0 a8 01 05 e5 ae 00 89 00 3a 53 ca 03 e8 00 00  .........:S.....
<br>0030   00 01 00 00 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01        AAAAAAAAA..!..
<br>
Response:
<br>
<br>0000   00 04 00 01 00 06 9c 2a 70 13 e5 2b 00 17 08 00  .......*p..+....
<br>0010   45 00 00 ef 2b 73 00 00 80 11 8b 34 c0 a8 01 05  E...+s.....4....
<br>0020   c0 a8 01 01 00 89 e5 ae 00 db b7 aa 03 e8 84 00  ................
<br>0030   00 00 00 01 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01 00 00  AAAAAAAAA..!....
<br>0060   00 00 00 9b 06 44 45 53 4b 54 4f 50 2d 32 4f 48  .....DESKTOP-2OH
<br>0070   52 48 49 47 20 04 00 44 45 53 4b 54 4f 50 2d 32  RHIG ..DESKTOP-2
<br>0080   4f 48 52 48 49 47 00 04 00 57 4f 52 4b 47 52 4f  OHRHIG...WORKGRO
<br>0090   55 50 20 20 20 20 20 20 00 84 00 57 4f 52 4b 47  UP      ...WORKG
<br>00a0   52 4f 55 50 20 20 20 20 20 20 1e 84 00 57 4f 52  ROUP      ...WOR
<br>00b0   4b 47 52 4f 55 50 20 20 20 20 20 20 1d 04 00 01  KGROUP      ....
<br>00c0   02 5f 5f 4d 53 42 52 4f 57 53 45 5f 5f 02 01 84  .__MSBROWSE__...
<br>00d0   00 08 00 27 2a 28 ac 00 00 00 00 00 00 00 00 00  ...'*(..........
<br>00e0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
<br>00f0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00     ...............
<br>
Corrupted: Junk Query, that can go up to gigabytes of Transmission.
<br>
<br>cat /dev/urandom | nc 192.168.1.5 139
<br>
<br>0000   00 04 00 01 00 06 e0 db 55 d8 d4 79 ae 67 08 00  ........U..y.g..
<br>0010   45 00 05 dc 00 f3 40 00 40 06 b0 d1 c0 a8 01 02  E.....@.@.......
<br>0020   c0 a8 01 05 ba e8 00 8b a0 3b 22 c5 ad 6e 9d f6  .........;"..n..
<br>0030   80 10 00 e5 3a e0 00 00 01 01 08 0a d8 4c 3d 42  ....:........L=B
<br>0040   00 19 47 ff 1a 20 68 cc 44 5d 5b 2a ad 6e 4e 8d  ..G.. h.D][*.nN.
<br>0050   43 81 eb 19 35 40 97 ba e3 29 20 9b f4 ac db a5  C...5@...) .....
<br>0060   85 8b 26 fa f8 ae e5 9f 10 fc d6 08 ec a4 fd 6f  ..&............o
<br>0070   a3 7d 13 f5 d7 7b 5f 11 f6 de 00 38 1f 2e f6 81  .}...{_....8....
<br>0080   76 e6 93 59 4a 20 70 49 26 51 5e ae 93 ab 14 d2  v..YJ pI&Q^.....
<br>0090   c5 8a b5 d5 70 d0 e2 fb f1 7a f0 03 43 5e a0 4f  ....p....z..C^.O
<br>00a0   c9 0c c9 99 a3 50 4c e0 6c c5 2e 64 d8 27 e4 b7  .....PL.l..d.'..
<br>00b0   59 e6 4e c8 2c 40 a0 58 30 19 b9 f9 32 46 d9 42  Y.N.,@.X0...2F.B
<br>00c0   4d 7e 14 29 3f 51 95 de e8 12 d6 8c 69 2d a7 70  M~.)?Q......i-.p
<br>00d0   7e a3 d4 d8 37 85 53 ae 8f f4 b3 f1 2a 80 f1 54  ~...7.S.....*..T
<br>00e0   1b ac 01 e6 40 21 f7 9a a2 d9 65 ff d1 8a f8 7b  ....@!....e....{
<br>00f0   dd 48 02 2a f1 ca 09 86 03 ea f9 e7 23 57 12 89  .H.*........#W..
<br>0100   f6 ac f4 f2 2e 33 b4 51 74 2c 56 1b 6e 89 51 4d  .....3.Qt,V.n.QM
<br>0110   d8 c3 ae 05 ae e1 5d 83 2a 63 22 20 dd d5 cc a9  ......].*c" ....
<br>0120   f2 5a 19 be c9 b4 9e db 99 d5 f2 8e 99 55 f1 de  .Z...........U..
<br>0130   3d f5 e7 cd f0 fc 05 8b 25 82 31 df 39 58 17 ae  =.......%.1.9X..
<br>0140   f9 9d 91 b9 a4 f8 42 3f 31 fd 6d a3 33 f5 b2 db  ......B?1.m.3...
<br>0150   ab e2 ed 41 dc 42 85 31 fd 48 5c 48 26 72 d1 46  ...A.B.1.H\H&r.F
<br>0160   44 33 ac 3c 65 44 c9 c9 7a 10 eb 9a 16 b7 3c 35  D3.<eD..z.....<5
<br>0170   d3 e0 c8 37 22 3f 67 3d 1a 3a ff 86 06 12 24 d3  ...7"?g=.:....$.
<br>0180   2c de b4 c1 b8 17 4e c6 a8 ea ca c9 02 bb 21 77  ,.....N.......!w
<br>0190   44 7c 09 61 18 46 c9 85 42 a5 b8 02 d2 06 5c d2  D|.a.F..B.....\.
<br>01a0   1a d4 2b 41 02 a9 6a fb a7 83 fa a7 ca 21 1a a2  ..+A..j......!..
<br>01b0   60 68 87 71 fa c6 0f 25 0a 61 e3 d3 a7 34 16 00  h.q...%.a...4..
<br>01c0   1b c9 2d 16 77 69 ca d8 d0 24 ef e7 62 2a a8 f8  ..-.wi...$..b*..
<br>01d0   a2 7a b5 9f de af 8c d4 cc 9e d7 ac e1 7b 8e b2  .z...........{..
<br>01e0   49 0d 86 9f db 5b 2b e7 84 44 84 f8 35 dc 52 39  I....[+..D..5.R9
<br>01f0   94 f6 99 1c 99 28 53 c9 1b 09 c2 b9 31 af 2d ea  .....(S.....1.-.
<br>0200   1d 3c 05 c1 6c b9 b6 91 01 42 40 85 df bc e4 4c  .<..l....B@....L
<br>0210   3f 54 f8 08 b4 e3 a4 e9 02 a1 b5 c5 ad fc 50 93  ?T............P.
<br>0220   b1 a9 7d 12 1e 7f 6f da e0 18 df 30 2a cd a8 97  ..}...o....0*...
<br>0230   95 95 36 71 06 73 20 1c e4 81 2c 26 54 0b 84 18  ..6q.s ...,&T...
<br>0240   1e 24 48 72 22 93 cf 85 27 f1 95 0e 5f 94 a1 dc  .$Hr"...'..._...
<br>0250   51 47 dc 9c 21 e9 0f 66 e6 d6 d8 45 28 3f b7 18  QG..!..f...E(?..
<br>0260   8b 1c 92 1c 50 60 48 7f ca 3a 23 91 68 71 67 15  ....PH..:#.hqg.
<br>0270   1f b4 fe 69 96 65 cf 34 df 90 80 dc a2 64 f1 4a  ...i.e.4.....d.J
<br>0280   0a b9 2e f2 5c 1f 05 f7 24 96 c0 82 6c af 71 7c  ....\...$...l.q|
<br>0290   98 77 57 bf 05 5d 9e a5 81 3b 81 f9 f1 a4 36 15  .wW..]...;....6.
<br>02a0   31 2e c2 1d 7e a2 0b 41 55 75 ba 4a 89 88 bc 3f  1...~..AUu.J...?
<br>02b0   98 f8 b1 07 7b f0 ef 64 ce 39 69 e8 5f 7e 50 cd  ....{..d.9i._~P.
<br>02c0   92 9e 8b e5 0e 10 64 0e 45 5e 8b a2 4d c3 78 6b  ......d.E^..M.xk
<br>02d0   d5 45 53 93 24 1c cd 25 e5 65 41 f4 19 db fe e3  .ES.$..%.eA.....
<br>02e0   d6 21 15 70 81 3a 3b 8c 7d 0f e0 5e 8a 3f 0f a3  .!.p.:;.}..^.?..
<br>02f0   0e 69 48 bb c2 c7 7e e3 1f 6e f1 ed 10 be 34 58  .iH...~..n....4X
<br>0300   f4 fc 0a 22 e4 af 46 a1 a2 87 29 31 12 c4 f6 ec  ..."..F...)1....
<br>0310   20 99 ec 86 da 87 21 e7 96 2c ac 6c 83 9e 2f b1   .....!..,.l../.
<br>0320   c8 40 c0 28 50 2d 33 14 f0 29 cc d4 c3 2b ba b4  .@.(P-3..)...+..
<br>0330   d4 37 33 6c 9d 78 e9 8c 8f aa c7 df 22 db a3 08  .73l.x......"...
<br>0340   49 01 af 0a 1e a2 4d df 9c b3 15 96 f0 de e9 c2  I.....M.........
<br>0350   cc a5 e1 ee 9b 05 f2 9a ad 27 e2 9e 2a ad 51 64  .........'..*.Qd
<br>0360   10 5d 15 a9 7a 96 37 16 2d 06 66 cd a0 a2 11 70  .]..z.7.-.f....p
<br>0370   19 f5 f3 aa a9 0e 13 59 99 24 5f 62 bd 7b 3d 7e  .......Y.$_b.{=~
<br>0380   d6 62 ed 47 d3 02 8c 20 02 83 33 1d 8b 11 59 77  .b.G... ..3...Yw
<br>0390   75 97 3b dd cc ca a5 29 37 0b c9 37 49 17 d0 40  u.;....)7..7I..@
<br>03a0   26 7a 3e 49 f2 e7 f6 ae e0 56 d2 4c 9f 3e 2b 14  &z>I.....V.L.>+.
<br>03b0   cf 84 82 00 ce a3 8b ec 60 14 fc 1b b5 16 b5 e4  ...............
<br>03c0   cd 43 42 8c 96 c6 8e 69 4c 96 a1 0f f4 e3 87 13  .CB....iL.......
<br>03d0   62 c8 c2 20 e7 dd 0b a5 d9 e7 03 74 68 8a 5f 8a  b.. .......th._.
<br>03e0   0e a7 33 72 d1 97 59 fd 90 09 3f 4c d0 0d 47 83  ..3r..Y...?L..G.
<br>03f0   b8 44 30 eb 84 08 cc 1f 24 4f 36 cf 30 a6 2c 57  .D0.....$O6.0.,W
<br>0400   d3 4f b5 9a 6b 07 91 5b 14 b9 ce 1d 8a e2 35 c2  .O..k..[......5.
<br>0410   68 b6 e9 3e 42 19 ef 57 5f aa 69 d0 95 d0 7d 39  h..>B..W_.i...}9
<br>0420   38 fa 14 d4 cd 68 b0 1b 8a ce 2c 10 1d 8f ce 19  8....h....,.....
<br>0430   ce 96 dd 57 61 91 69 08 d7 2f 7c 42 59 cb dd de  ...Wa.i../|BY...
<br>0440   ad 10 c6 5c b5 18 4f e0 2f 4c 8b 31 fa 9d 9f cc  ...\..O./L.1....
<br>0450   c2 59 df 9c db 2c 71 c3 05 ba 31 ff f8 08 23 9a  .Y...,q...1...#.
<br>0460   a6 e5 f8 1a 0c c4 d3 c6 6b a6 1f ba f9 05 65 00  ........k.....e.
<br>0470   34 dd 38 b7 6b 97 10 8e ea 9e 72 ca 21 f4 ba 33  4.8.k.....r.!..3
<br>0480   8d 08 aa 61 d6 51 15 74 aa 46 c0 21 cc 71 36 1f  ...a.Q.t.F.!.q6.
<br>0490   dc 91 7e d6 58 cd 8e d7 cd de 0b 0f 6e fc 2b f9  ..~.X.......n.+.
<br>04a0   fd da e4 45 4b 82 a6 fd 51 ca 17 c6 2d 43 33 b6  ...EK...Q...-C3.
<br>04b0   80 b5 f7 ea 78 0a c9 a2 2f 01 7f b8 4b 45 a9 b4  ....x.../...KE..
<br>04c0   32 30 8c 74 7a f1 42 0e 5b 4e 5a a4 07 b1 42 2c  20.tz.B.[NZ...B,
<br>04d0   f8 09 af b8 24 0f 0f 19 1c 1b ac 49 0b 88 25 01  ....$......I..%.
<br>04e0   5a 50 8a f9 99 5f 40 c0 5b 8e 74 59 1c 2c 4e ed  ZP..._@.[.tY.,N.
<br>04f0   84 19 ab 39 ec 1b 90 45 35 a9 12 12 48 42 d7 77  ...9...E5...HB.w
<br>0500   51 14 aa 5c 64 4f 3e 8a 3f 9b bb da 1a 1c 4a 19  Q..\dO>.?.....J.
<br>0510   3a 21 83 0e 9a 42 14 a3 97 c6 fd b7 49 6d e5 2f  :!...B......Im./
<br>0520   f4 ce d1 38 9c 07 de fe 59 c5 da 8f 0c 15 45 ac  ...8....Y.....E.
<br>0530   20 1f 4c 18 fb 4e a5 c4 02 db fb 19 6c c4 4d 88   .L..N......l.M.
<br>0540   e3 9d 98 9a e3 29 0a 79 99 40 7d d1 cd 9e 17 c3  .....).y.@}.....
<br>0550   40 96 c2 06 b3 7a 46 11 c3 83 01 12 f4 1d f2 ce  @....zF.........
<br>0560   8e c1 c7 9b f2 41 c9 79 e1 f9 a4 b1 28 e2 9f e6  .....A.y....(...
<br>0570   94 42 8c 09 2f 3b a4 c2 4f 5a c8 cb 28 03 ed 92  .B../;..OZ..(...
<br>0580   f3 0e a8 fe f5 52 2f 0e 89 5e 04 3d 83 d1 11 07  .....R/..^.=....
<br>0590   c0 b1 32 20 c8 bb b5 a8 ab d9 d7 e2 9d 8a d5 34  ..2 ...........4
<br>05a0   e5 34 76 27 41 d6 99 63 1c 16 28 11 27 08 ce 38  .4vA..c..(.'..8
<br>05b0   5c 96 98 9d 60 d8 8a 63 7f 47 bf 16 72 e0 80 d1  \.....c.G..r...
<br>05c0   4d 88 39 6d fe bd 85 26 90 51 0f 84 83 31 af b5  M.9m...&.Q...1..
<br>05d0   36 f6 7c 4a db 78 d9 a4 fb 86 73 68 51 39 85 c6  6.|J.x....shQ9..
<br>05e0   10 76 f7 d3 a0 be d6 12 67 a1 21 29              .v......g.!)
<br>
<br>0000   00 04 00 01 00 06 e0 db 55 d8 d4 79 de 1e 08 00  ........U..y....
<br>0010   45 00 05 dc 00 f4 40 00 40 06 b0 d0 c0 a8 01 02  E.....@.@.......
<br>0020   c0 a8 01 05 ba e8 00 8b a0 3b 28 6d ad 6e 9d f6  .........;(m.n..
<br>0030   80 18 00 e5 61 4d 00 00 01 01 08 0a d8 4c 3d 42  ....aM.......L=B
<br>0040   00 19 47 ff 17 23 e4 86 9d 30 44 02 00 5f dd d7  ..G..#...0D.._..
<br>0050   a1 03 75 d3 fb 1c 9e 74 a7 5d b0 64 5c 88 f0 ac  ..u....t.].d\...
<br>0060   2a af a9 69 24 d2 1d f8 81 30 2a 2c 56 ca 3a 7f  *..i$....0*,V.:.
<br>0070   73 c6 77 d9 90 f9 53 24 ba 76 bf 5a 18 25 b8 f7  s.w...S$.v.Z.%..
<br>0080   f8 29 04 c0 4a e8 ae 48 b2 92 8a 77 d2 49 d4 4c  .)..J..H...w.I.L
<br>0090   9b b9 69 8a e8 4f a3 d2 20 ba 25 26 25 16 7e 4e  ..i..O.. .%&%.~N
<br>00a0   7b 7d 98 01 07 49 bb 6f cc 0d 30 8b da b1 67 f7  {}...I.o..0...g.
<br>00b0   eb 51 1f 44 9f 69 92 69 57 1c e9 d7 f0 16 e7 68  .Q.D.i.iW......h
<br>00c0   21 a2 5d c4 bb 41 9c fb 1f 57 ef 7c 79 6b 89 97  !.]..A...W.|yk..
<br>00d0   08 dd 2c a4 1e 6b 10 ae 4f 51 e0 c3 be b2 c4 46  ..,..k..OQ.....F
<br>00e0   e0 a6 2e 58 0c 41 26 b1 7d c1 51 95 dc 8d b0 4e  ...X.A&.}.Q....N
<br>00f0   6b 7b 81 5c c4 70 48 f9 07 4e 71 4a 7d fb fc ad  k{.\.pH..NqJ}...
<br>0100   7a 48 2a ae 12 68 0a 05 e3 8c 31 ff c5 84 2d a3  zH*..h....1...-.
<br>0110   ba 72 09 b9 b0 eb b1 d8 60 bc b5 e3 63 4d 06 c9  .r.........cM..
<br>0120   7c 4b cf 5f da c1 d7 f6 ef 38 be e8 7d 99 47 97  |K._.....8..}.G.
<br>0130   8e 65 13 de e5 4d 22 a8 88 94 bf 9f 04 76 99 1a  .e...M"......v..
<br>0140   6d 5b 89 34 51 5e 85 a9 e4 ae 9f 26 f9 0c 7e b3  m[.4Q^.....&..~.
<br>0150   bc 60 2e 78 d5 cb 7b ea 54 68 db 41 2d 59 9e e3  ..x..{.Th.A-Y..
<br>0160   00 3e 1a 6e c7 4f 6e 4c a3 ef 78 77 87 4a ed 00  .>.n.OnL..xw.J..
<br>0170   f1 8f ea f3 3c a2 38 0e f5 56 d7 dd 50 36 cd 25  ....<.8..V..P6.%
<br>0180   02 49 27 d8 9a 3f 9b d1 aa 29 d3 fd 04 51 f0 0e  .I'..?...)...Q..
<br>0190   c4 70 34 87 83 5a b4 c6 b5 cb 29 d5 96 5f da e3  .p4..Z....).._..
<br>01a0   fe 5e 25 fd dc dd f5 5e 0f 8e 96 52 44 e9 84 21  .^%....^...RD..!
<br>01b0   82 6e 84 65 7d 2f 42 c9 21 15 ac 14 a0 bd 49 40  .n.e}/B.!.....I@
<br>01c0   96 f1 5b 3f 68 ff c7 f9 c9 85 97 35 51 65 2c 90  ..[?h......5Qe,.
<br>01d0   1b d2 fd 12 1a 19 43 1c bd 92 6a 77 15 d2 89 ce  ......C...jw....
<br>01e0   ff d0 16 a3 a3 4a 3e 11 1f a7 4e eb 53 d4 a5 9d  .....J>...N.S...
<br>01f0   22 2b 17 65 2d 0f c7 4f 37 e3 4a 98 ad 44 49 64  "+.e-..O7.J..DId
<br>0200   80 04 8e e0 9f 57 61 4a 06 ed 40 3c f8 be 39 0d  .....WaJ..@<..9.
<br>0210   ce 67 d0 18 08 81 01 fc fb db 14 5d 7f 42 d5 be  .g.........].B..
<br>0220   34 b3 51 df ba e4 83 c0 cb 0b d2 9e 4d 63 97 98  4.Q.........Mc..
<br>0230   07 24 6f cd d7 52 c2 4c db e7 76 2b f3 a1 d0 b6  .$o..R.L..v+....
<br>0240   cd b2 4b 2d 2e 82 20 35 15 02 12 6d 4b 5c c0 a2  ..K-.. 5...mK\..
<br>0250   fc 38 05 af 95 1e 82 99 85 8c 01 55 55 d6 69 a2  .8.........UU.i.
<br>0260   c7 d3 64 7c 96 f0 86 15 52 6f de 53 23 b4 bd 2f  ..d|....Ro.S#../
<br>0270   9a f3 dd f9 2b 04 7f d8 7e 15 99 bd 91 46 d9 d9  ....+...~....F..
<br>0280   c2 bc 20 84 c6 7c 3e 2c 7f cb 8e 4f 34 ae 4d 52  .. ..|>,...O4.MR
<br>0290   8d 5e a1 b5 c5 79 08 12 26 9e 8b 39 03 5d d6 61  .^...y..&..9.].a
<br>02a0   c7 b8 6b dd 32 64 58 53 17 22 0a 86 56 8d 14 df  ..k.2dXS."..V...
<br>02b0   92 2f 8a ae 4a 9b 1e 71 44 b6 a6 64 8b e8 cd 1d  ./..J..qD..d....
<br>02c0   f5 2d 04 5d b8 72 99 98 01 4d 05 a6 81 ea 58 19  .-.].r...M....X.
<br>02d0   de 56 1d 90 7b 2f 42 ae 8a 23 2d 3d d0 67 21 b1  .V..{/B..#-=.g!.
<br>02e0   71 db 26 09 d4 a3 b3 79 4d 87 eb 2f 7e 99 18 05  q.&....yM../~...
<br>02f0   22 6c 12 b6 87 a6 32 08 00 c9 37 a3 43 74 84 4e  "l....2...7.Ct.N
<br>0300   1c ac 4e 04 bf e3 23 4c 24 2d 1a 9e cd 15 92 64  ..N...#L$-.....d
<br>0310   b0 b1 4b 60 7b 22 c2 c0 aa 3d c0 ca 9f 41 1a 43  ..K{"...=...A.C
<br>0320   1a 46 d2 e1 0a 34 00 c9 56 2b 5b 8a 70 60 12 90  .F...4..V+[.p..
<br>0330   c0 90 0a a7 6b 4d 19 d8 14 80 39 6f 13 36 0c 4a  ....kM....9o.6.J
<br>0340   c0 af 2a 7e 13 66 3d ae d9 06 7e c2 86 1a e2 81  ..*~.f=...~.....
<br>0350   71 5b 80 1a f1 0e 18 f0 85 22 b5 5a ef 4a 39 6f  q[.......".Z.J9o
<br>0360   95 73 e5 15 1b 64 c2 c8 86 58 a4 31 da 01 db 30  .s...d...X.1...0
<br>0370   aa 0a 9c 84 7c ca 02 2f 9c 95 92 86 11 65 c6 64  ....|../.....e.d
<br>0380   fa d5 49 d7 15 c8 81 60 ac 8f f0 16 71 18 98 16  ..I........q...
<br>0390   bb a6 1b d7 78 23 fa 47 3e 3b 29 ec e9 f0 b1 10  ....x#.G>;).....
<br>03a0   a7 fc 88 96 43 95 4f b6 93 55 04 2f da 46 4e fe  ....C.O..U./.FN.
<br>03b0   c6 0a 14 da 91 a9 6f 13 e5 3a 12 f6 8d 26 bf af  ......o..:...&..
<br>03c0   31 db cc e4 21 a2 e7 43 1e f4 df c0 63 86 8e 83  1...!..C....c...
<br>03d0   99 d1 8d a8 1c c9 4a 00 1c ec 6a 7e 5f 72 7f 91  ......J...j~_r..
<br>03e0   dc 7d 36 39 80 e8 d0 b6 a0 51 e0 29 da 42 3c 54  .}69.....Q.).B<T
<br>03f0   02 a6 b3 13 16 5a 08 30 c1 81 74 cd 9b 71 90 20  .....Z.0..t..q. 
<br>0400   9c d2 78 c1 df 01 ac 6a 72 05 88 79 f7 ee fb ec  ..x....jr..y....
<br>0410   a4 ed 2b 21 f0 e7 26 a9 06 f1 44 8b bc a3 a8 27  ..+!..&...D....'
<br>0420   d1 dd c5 97 84 45 5a 60 00 e7 1c bf 4f 1d c2 88  .....EZ....O...
<br>0430   a3 21 38 04 fe 59 a8 ab be b3 1f 87 1c 92 13 d4  .!8..Y..........
<br>0440   eb 49 2d bb a7 f0 9b 1a 53 a4 1f f7 bc 84 f5 5b  .I-.....S......[
<br>0450   a3 64 a4 0a c6 3a 2a 6d 54 f7 5d ab 55 2c da 80  .d...:*mT.].U,..
<br>0460   09 25 93 6b 83 5a 5b 88 09 3e b8 24 63 46 a2 ac  .%.k.Z[..>.$cF..
<br>0470   91 c7 7d 10 1f 79 e7 a5 55 e1 cd 05 0b 88 7c 17  ..}..y..U.....|.
<br>0480   d5 6f 27 42 49 3a f2 90 3a 44 7c bb 88 5e 44 b2  .o'BI:..:D|..^D.
<br>0490   0f 85 e3 ef 3b 81 a1 63 bb bc 6e 33 7f ba 7f d5  ....;..c..n3....
<br>04a0   23 5b 3c b0 1f a9 5b 8e 74 f7 a0 46 d7 73 41 73  #[<...[.t..F.sAs
<br>04b0   17 3c ca c3 2a 43 90 28 97 87 50 10 08 49 09 b6  .<..*C.(..P..I..
<br>04c0   98 d8 26 75 96 9f cf 5a f7 88 ee 8c b8 6f ae 09  ..&u...Z.....o..
<br>04d0   f5 62 34 87 10 ca f2 db 2c 70 f6 5b ae 75 44 c9  .b4.....,p.[.uD.
<br>04e0   01 d7 7f b5 79 e4 2d a2 2f 8d 25 fb 06 1f 17 29  ....y.-./.%....)
<br>04f0   7d 1f c1 87 be 32 5c b7 58 47 50 b6 07 e8 dc 7c  }....2\.XGP....|
<br>0500   a3 61 6b 91 e9 f9 7a b8 ba 13 52 55 72 88 fd 55  .ak...z...RUr..U
<br>0510   bf 02 d8 6d e8 d7 e5 b2 e4 af 53 e7 3e 22 7f 58  ...m......S.>".X
<br>0520   23 e9 03 ad d6 de f1 33 56 48 1a f2 92 0b 26 55  #......3VH....&U
<br>0530   2c f0 ee 3c 91 dd 24 18 49 71 7c c7 02 31 17 5a  ,..<..$.Iq|..1.Z
<br>0540   fe 01 e0 c3 48 69 ec c4 f1 59 45 eb 4b 20 8b 41  ....Hi...YE.K .A
<br>0550   4a 7e 3e 69 a0 b3 ab 80 88 aa 74 fe c4 0b 67 0f  J~>i......t...g.
<br>0560   f5 9f 45 1c 5b a4 d4 d0 08 af e2 b7 1a 01 cc 84  ..E.[...........
<br>0570   34 7a 29 22 0a c0 59 a3 1b 04 d2 34 9f 18 a3 fb  4z)"..Y....4....
<br>0580   01 62 2d 41 2f 68 6a 38 55 dc 55 67 3c 00 6a dd  .b-A/hj8U.Ug<.j.
<br>0590   35 86 2c 23 dc 48 6a f9 f8 74 23 54 f3 5c 1f 1b  5.,#.Hj..t#T.\..
<br>05a0   4e 66 f5 f1 44 64 bb be 0a 50 50 f2 ed 38 24 54  Nf..Dd...PP..8$T
<br>05b0   61 11 02 d6 2f 27 3a 9b ef 67 56 19 47 c4 88 b5  a.../':..gV.G...
<br>05c0   80 42 75 86 82 31 27 a5 8d 5b f4 ba 52 19 15 29  .Bu..1'..[..R..)
<br>05d0   ad fd 3b 1c 62 12 70 3a a7 e1 b9 07 47 d8 b6 49  ..;.b.p:....G..I
<br>05e0   20 ff 78 63 5b 10 fe 35 77 62 51 80               .xc[..5wbQ.
<br>
<br>
<br>0000   00 04 00 01 00 06 e0 db 55 d8 d4 79 40 ad 08 00  ........U..y@...
<br>0010   45 00 05 dc 00 f5 40 00 40 06 b0 cf c0 a8 01 02  E.....@.@.......
<br>0020   c0 a8 01 05 ba e8 00 8b a0 3b 2e 15 ad 6e 9d f6  .........;...n..
<br>0030   80 10 00 e5 42 67 00 00 01 01 08 0a d8 4c 3d 42  ....Bg.......L=B
<br>0040   00 19 47 ff fe 95 04 a5 f9 41 69 23 8b 12 36 49  ..G......Ai#..6I
<br>0050   7d 95 b6 a1 2a 38 dc 5b c4 ba 25 c7 b8 73 06 66  }...*8.[..%..s.f
<br>0060   8d 53 02 9b df 48 fc 43 f5 9d 7b 93 6f 98 78 7e  .S...H.C..{.o.x~
<br>0070   b4 24 4c 1e 92 9a 71 7e 90 12 33 7c 2e 9b 13 e0  .$L...q~..3|....
<br>0080   7c fd 8a 4e 9f f4 c1 0f fc d6 03 0c 0d e5 06 44  |..N...........D
<br>0090   50 86 cd 33 58 5a b5 40 13 2c 6d b3 8a 44 91 eb  P..3XZ.@.,m..D..
<br>00a0   db ee 5f 8a ea be da 0a 6c 42 ca 6b 3a 5a 7e aa  .._.....lB.k:Z~.
<br>00b0   23 e6 11 da 22 c5 a6 1c 11 b8 d6 8d 30 b4 12 26  #...".......0..&
<br>00c0   40 3b eb ce a3 00 e0 c3 38 c7 6f 0d 9d 6c a1 58  @;......8.o..l.X
<br>00d0   72 03 7e fc e9 42 8e 0b ee a3 37 cb ab 2c ac 70  r.~..B....7..,.p
<br>00e0   40 c4 e3 35 92 b2 bb a9 75 9e f6 61 f8 a5 cb ff  @..5....u..a....
<br>00f0   e9 16 35 75 04 17 11 d4 43 4b 0b ad c4 e4 a6 9b  ..5u....CK......
<br>0100   bb 11 bd c5 1e 2e 2c 1f c2 07 7e ef c4 b7 76 86  ......,...~...v.
<br>0110   10 14 96 71 02 0f 63 4f 32 00 31 98 15 2b a1 31  ...q..cO2.1..+.1
<br>0120   9e fc 50 0a 3a 45 89 8f c6 23 af ef a5 91 b1 cb  ..P.:E...#......
<br>0130   7e a6 61 60 73 86 2d 31 23 77 3f 3c a8 21 ab 0e  ~.as.-1#w?<.!..
<br>0140   f8 48 b9 a8 6f 46 c2 ea 52 8a db 48 95 66 f3 e5  .H..oF..R..H.f..
<br>0150   3f a1 9b dd a2 b4 05 08 75 f4 f9 14 65 bb af 8f  ?.......u...e...
<br>0160   37 7d 7f a1 73 c9 56 ba ec 4f 8f c4 2d be 31 c9  7}..s.V..O..-.1.
<br>0170   f7 42 5d d7 3b 17 41 f4 e4 fd 89 2f cf 58 c5 7a  .B].;.A..../.X.z
<br>0180   1d 96 00 ff 6e e4 40 c6 ad ce e3 f9 b7 71 fe ac  ....n.@......q..
<br>0190   ba 4f f9 a6 39 50 a4 37 0d ac d7 03 53 9c 9b 64  .O..9P.7....S..d
<br>01a0   61 11 4e be 19 bd df e7 05 26 4f d2 07 c8 23 b5  a.N......&O...#.
<br>01b0   e8 b7 da a6 7c 97 25 96 57 fa e6 d5 70 32 2d 3f  ....|.%.W...p2-?
<br>01c0   fd ef 03 2f 50 5f 9a 9e d8 2d 33 89 70 82 93 0e  .../P_...-3.p...
<br>01d0   61 af 21 4f 0c 76 85 ae 74 94 20 a9 50 0f 49 af  a.!O.v..t. .P.I.
<br>01e0   80 cc 35 bb 0e a1 e3 fc 79 43 3a ba 80 d5 71 da  ..5.....yC:...q.
<br>01f0   2d e1 28 f1 71 47 40 97 8f c3 d0 1f 0e 01 60 29  -.(.qG@.......)
<br>0200   bd 96 2d 10 a2 bc ae e8 da 0a 9f 93 59 f9 2b 71  ..-.........Y.+q
<br>0210   62 12 17 ab 5e bd 37 f0 1c 64 db 43 e2 13 14 e8  b...^.7..d.C....
<br>0220   30 f0 75 46 0b a4 4c 8b d4 a2 3f b7 78 fc 40 22  0.uF..L...?.x.@"
<br>0230   f1 90 62 89 65 13 f0 2c 53 d4 05 0b 20 5e 31 32  ..b.e..,S... ^12
<br>0240   97 bf 8e a7 69 47 49 cc 27 74 3e c2 3a 97 c3 6f  ....iGI.'t>.:..o
<br>0250   5e 80 94 e9 8e 56 72 81 9f b0 82 8c 87 57 9f 98  ^....Vr......W..
<br>0260   1f 86 7e 50 c7 b2 98 5f 29 5b 38 c4 46 97 14 08  ..~P..._)[8.F...
<br>0270   5a 7c 4f b2 81 d6 bd 84 b2 11 cb a8 7b 67 a6 12  Z|O.........{g..
<br>0280   f7 f9 1c 2c 17 b5 be f0 17 8c 2c 36 30 79 8e f4  ...,......,60y..
<br>0290   77 45 32 b6 df c9 c0 93 8f b6 62 bf 67 6c f8 0e  wE2.......b.gl..
<br>02a0   7f f0 19 63 e2 af f4 66 6c c2 33 cc e5 11 aa 71  ...c...fl.3....q
<br>02b0   4f 1d 1f 45 14 01 05 77 81 24 63 26 22 4e 53 06  O..E...w.$c&"NS.
<br>02c0   d9 f1 b0 9e 26 fd 9a 52 aa d0 4d 7b f9 73 6c a7  ....&..R..M{.sl.
<br>02d0   1c 12 85 8e 51 7f ad 31 d7 cd ff 30 1d 0c ea af  ....Q..1...0....
<br>02e0   5e f5 b4 25 e5 cf 28 4d 91 f4 a8 a1 52 4e 3d 4d  ^..%..(M....RN=M
<br>02f0   75 8a 8d 03 ba e5 81 51 74 c0 15 f9 e4 cb 72 d2  u......Qt.....r.
<br>0300   b6 c8 e6 81 4c ef 60 59 81 e3 f6 f4 9e b0 82 f9  ....L.Y........
<br>0310   62 d4 8c 49 ec ed 76 73 ae 23 dd 35 81 b8 fe e2  b..I..vs.#.5....
<br>0320   d5 b1 51 a1 f3 16 eb 75 be 29 0c cc c4 66 7c 19  ..Q....u.)...f|.
<br>0330   07 2b 39 30 12 5c d2 ad e2 da de 9c 8f 62 92 8f  .+90.\.......b..
<br>0340   87 38 61 69 c6 35 a0 86 c3 4d 9d 68 d2 ec 02 13  .8ai.5...M.h....
<br>0350   7e a1 d2 5c 1a 0a 72 f3 f9 bc 04 6a cb a2 de b7  ~..\..r....j....
<br>0360   67 83 4c aa 18 1e 36 6f ce 9c 43 fd c6 66 55 80  g.L...6o..C..fU.
<br>0370   fa f4 52 9f e1 dc c4 fe 0c 73 09 be 4c b2 da 2a  ..R......s..L..*
<br>0380   27 ff 81 23 fc f6 67 6f 40 8e 04 30 ff b1 7a 3a  '..#..go@..0..z:
<br>0390   90 0b 22 2c 3d f0 80 23 41 0d 43 b1 fc ad fb 0e  ..",=..#A.C.....
<br>03a0   51 55 8d 2a 38 29 70 1d 80 1b d1 10 b2 34 79 9d  QU.*8)p......4y.
<br>03b0   e2 b2 15 e6 11 c4 79 49 6d 9c 8d ed bb 7d 07 c9  ......yIm....}..
<br>03c0   6e 4b 21 2a 8a 12 7a 48 98 55 c2 20 41 42 c3 bb  nK!*..zH.U. AB..
<br>03d0   59 f4 e1 ac 1c f9 d8 d6 cb d9 57 58 30 dd f1 a1  Y.........WX0...
<br>03e0   84 82 71 d8 00 a1 92 d0 2a e2 a3 b1 b0 e9 0c 8b  ..q.....*.......
<br>03f0   b5 dd ac 9d db 82 07 80 bf 9a 90 6c b6 3f e0 47  ...........l.?.G
<br>0400   51 dc 27 af bd a1 a1 b2 2b ca ca d7 ef 27 c8 00  Q......+......
<br>0410   2a 2a b8 77 a2 6d b8 a5 33 e5 16 ed 31 7f c4 99  *.w.m..3...1...
<br>0420   31 d7 2d 4a df 9b 39 ce 46 47 5a 17 f5 a5 34 82  1.-J..9.FGZ...4.
<br>0430   81 66 05 96 94 35 9e 4f 36 a0 62 f3 a4 11 2d c5  .f...5.O6.b...-.
<br>0440   9c 3c f7 86 68 c4 2f 0f c8 01 80 d5 ab ae cf 3a  .<..h./........:
<br>0450   c1 81 06 35 2b e1 f9 eb 97 6d fd 62 02 0a 12 8b  ...5+....m.b....
<br>0460   07 a8 f5 b3 2b 85 46 5e e8 ae 38 7e 9d f1 41 fd  ....+.F^..8~..A.
<br>0470   ea b3 96 08 6e b3 48 10 59 ed e7 57 4d df 5b af  ....n.H.Y..WM.[.
<br>0480   1c c6 36 fe 59 4b db e7 57 11 81 0f ed 6c 07 d9  ..6.YK..W....l..
<br>0490   27 7f 44 b2 70 4c d3 d6 75 38 e7 00 be 71 17 e4  '.D.pL..u8...q..
<br>04a0   ff da 05 c8 32 6f 8f db da b0 b6 d0 70 6b 8f a8  ....2o......pk..
<br>04b0   5e 57 2e 9f 82 c0 d6 b8 7e b5 b5 12 31 81 dd bc  ^W......~...1...
<br>04c0   b0 fd c5 58 51 fa b3 4c 83 ed 23 3b 2c 37 1f e2  ...XQ..L..#;,7..
<br>04d0   30 65 2b 19 78 2a 27 65 c9 05 51 e8 8b af 5c cd  0e+.x*'e..Q...\.
<br>04e0   12 5f 22 c4 32 39 9c c0 39 c9 07 1a 88 4d 99 9a  ._".29..9....M..
<br>04f0   1c 8c 64 bb e2 c1 14 b4 82 32 b8 28 c6 b3 68 f5  ..d......2.(..h.
<br>0500   4d 3a 64 61 24 7d e4 04 7a 78 56 b4 d8 09 ad 50  M:da$}..zxV....P
<br>0510   7f 31 28 0a 1e 79 c6 b7 a7 31 08 42 8a fa bc 3e  .1(..y...1.B...>
<br>0520   c6 3b d3 aa 67 12 53 c2 65 60 02 f3 39 60 fe e6  .;..g.S.e..9..
<br>0530   be 1f 5a 5f 6b 33 92 ab c0 c9 7c 44 93 ab 68 d6  ..Z_k3....|D..h.
<br>0540   6a 2f d6 24 de be 74 f7 95 a1 f8 ab 9a eb fa 78  j/.$..t........x
<br>0550   c2 f5 c4 00 69 c7 c0 55 c6 2e c6 7a ac f8 3f 46  ....i..U...z..?F
<br>0560   5e 92 d2 69 cd 73 3c e0 46 ec e5 01 a3 5b be f5  ^..i.s<.F....[..
<br>0570   a7 be 4b 27 61 de b0 6b 86 2b 2c e6 6a eb 4e 4c  ..K'a..k.+,.j.NL
<br>0580   11 74 18 e9 b1 23 ab d9 dd df c9 08 58 fd fb 9e  .t...#......X...
<br>0590   ec 4e b6 c5 a3 0c c4 0a cc 54 65 b7 b7 ce e3 0a  .N.......Te.....
<br>05a0   91 22 62 52 70 49 6c 15 79 e7 47 72 4a a4 c0 13  ."bRpIl.y.GrJ...
<br>05b0   17 1c b8 d5 86 ec d9 51 d1 f0 91 16 14 51 cf a7  .......Q.....Q..
<br>05c0   cc 21 d2 6b 1e 09 53 b1 b7 82 54 35 ed 0b 1c ec  .!.k..S...T5....
<br>05d0   de 28 ce 73 ad a9 8b 60 f0 43 47 c6 75 67 1f a9  .(.s....CG.ug..
<br>05e0   00 32 99 11 8b e5 a2 b3 fb 3b 27 63              .2.......;c
<br>
<br>0000   00 04 00 01 00 06 e0 db 55 d8 d4 79 43 e7 08 00  ........U..yC...
<br>0010   45 00 05 dc 00 f6 40 00 40 06 b0 ce c0 a8 01 02  E.....@.@.......
<br>0020   c0 a8 01 05 ba e8 00 8b a0 3b 33 bd ad 6e 9d f6  .........;3..n..
<br>0030   80 10 00 e5 05 76 00 00 01 01 08 0a d8 4c 3d 42  .....v.......L=B
<br>0040   00 19 47 ff 40 df 99 2f 60 58 a0 d8 b3 12 01 11  ..G.@../X......
<br>0050   7f e4 ca e9 70 67 45 3f cd 5a 4c b8 c9 b9 12 d8  ....pgE?.ZL.....
<br>0060   6c 31 48 a1 ae df e9 72 68 73 fa 54 d4 f0 e9 eb  l1H....rhs.T....
<br>0070   c9 0a b5 32 ae a9 d6 bc f9 30 42 6c df fd 51 79  ...2.....0Bl..Qy
<br>0080   c6 81 bb 90 5f db d4 aa a8 81 db 90 ec fc 3e 80  ...._.........>.
<br>0090   96 6c 22 41 a1 91 38 66 8c 90 03 b4 21 4a dc 5c  .l"A..8f....!J.\
<br>00a0   77 74 35 a0 7c 02 aa 7b 87 99 28 d3 36 1b e7 54  wt5.|..{..(.6..T
<br>00b0   26 f1 bc c1 85 c4 9c d3 85 41 ed 91 04 9e bc 7e  &........A.....~
<br>00c0   15 a8 d0 7d d0 51 32 36 49 d3 b7 13 04 a9 6d 41  ...}.Q26I.....mA
<br>00d0   07 13 56 21 08 1d 0e 13 9d 94 d0 1c 34 fd 8e ae  ..V!........4...
<br>00e0   06 20 ad 7a 10 ad a6 4c 66 1d fd ed f2 a1 4c 9a  . .z...Lf.....L.
<br>00f0   4d 39 3c 8b 98 fe c1 5c 4f 56 38 50 aa 3e 65 b2  M9<....\OV8P.>e.
<br>0100   e6 80 75 53 99 ec 35 fd b9 88 1e a4 39 d6 b6 42  ..uS..5.....9..B
<br>0110   b0 53 d9 f2 b0 46 43 c5 da d2 63 b4 10 c9 a7 4e  .S...FC...c....N
<br>0120   bb 50 92 8c fe f2 4b 5e 5a c4 30 90 7b 38 93 79  .P....K^Z.0.{8.y
<br>0130   35 32 03 07 af 1b 1c 78 88 cc 5f 14 f1 e8 00 6d  52.....x.._....m
<br>0140   28 ae 6c 6b 90 25 03 e1 d2 7d f6 99 d1 fd 5c f0  (.lk.%...}....\.
<br>0150   b4 0a 96 b4 6f 25 dc 65 dc 6c af b0 f4 36 ce f7  ....o%.e.l...6..
<br>0160   bf e9 3d 30 80 02 cb bc 5b 4c 7b 3a 31 e9 61 65  ..=0....[L{:1.ae
<br>0170   6f 10 43 6b d9 15 82 c6 24 7f ad 51 84 1c c0 71  o.Ck....$..Q...q
<br>0180   48 67 32 5f 06 3e e8 0c ae 7e 3e 04 8e f8 ee 18  Hg2_.>...~>.....
<br>0190   8f 62 79 fd 37 1b 16 71 b8 86 11 68 a3 14 23 ee  .by.7..q...h..#.
<br>01a0   b0 5f d7 78 1a 0c 74 20 e1 c6 71 20 b9 25 7f cc  ._.x..t ..q .%..
<br>01b0   11 09 2e 1c 21 e0 2a 16 54 ec 3a 6f 2b c6 07 e9  ....!.*.T.:o+...
<br>01c0   d9 42 71 d2 e3 8f 6d c6 c2 97 24 68 2b ab 48 51  .Bq...m...$h+.HQ
<br>01d0   93 bd 60 ac 3a c8 87 57 46 c4 02 47 9a f4 82 2e  ...:..WF..G....
<br>01e0   e1 60 f0 91 2a ea b9 c7 85 87 7e 20 24 2d c2 48  ...*.....~ $-.H
<br>01f0   74 95 99 86 56 c5 45 6f 96 25 4d 50 ae b1 a0 88  t...V.Eo.%MP....
<br>0200   83 1c 3b 15 23 4f ab 33 dc 92 69 c0 c8 1d 62 9a  ..;.#O.3..i...b.
<br>0210   01 b4 5a 43 38 3c 20 93 2a d2 91 d1 30 5f cb b4  ..ZC8< .*...0_..
<br>0220   09 08 2c cc 78 f1 a6 56 90 24 84 f0 19 3c 7b 00  ..,.x..V.$...<{.
<br>0230   c7 e3 e5 bf 5c 7e a3 55 20 f1 ed 7c cd 47 78 a5  ....\~.U ..|.Gx.
<br>0240   63 65 98 86 34 0d 4a 18 de d6 20 08 b5 8d 4f 30  ce..4.J... ...O0
<br>0250   e2 67 9f ff 7e d1 25 3a 24 3b fa a6 51 1c 1e 1f  .g..~.%:$;..Q...
<br>0260   b8 e2 81 a0 49 7f 50 2b 2d 23 c6 a0 cf 93 9c 05  ....I.P+-#......
<br>0270   65 a7 14 47 86 dc 48 1a 35 57 95 ae 71 6c 0d ce  e..G..H.5W..ql..
<br>0280   ff e1 f1 21 ee 77 8f ac 70 e2 05 c5 7c 92 d3 91  ...!.w..p...|...
<br>0290   3e da ae 83 20 c3 11 62 b5 90 a8 b7 1d 12 58 73  >... ..b......Xs
<br>02a0   93 24 04 1f 52 16 d9 96 05 17 3b f3 c4 6b 92 ef  .$..R.....;..k..
<br>02b0   04 b7 0e 8d c7 c9 f7 a9 cf 91 3a f4 43 8b 09 02  ..........:.C...
<br>02c0   09 1b bc 6a 0b ae 96 0f 84 c1 b6 a3 b2 9c 47 24  ...j..........G$
<br>02d0   c3 2d a1 2d 4c 1c a6 7a 9a b5 b3 c1 15 81 70 98  .-.-L..z......p.
<br>02e0   8a 7c 67 e6 85 2b d8 a8 49 06 ec c6 0b f7 fc ab  .|g..+..I.......
<br>02f0   09 5f eb 62 e6 b5 bd 09 c5 64 3e d0 37 81 db 65  ._.b.....d>.7..e
<br>0300   71 cb 56 da fd b3 a8 93 9e 59 bd 8c 82 ce f9 ce  q.V......Y......
<br>0310   79 b6 46 f7 f7 9b b2 8e 7f 11 2b 99 3d ab 4e 4f  y.F.......+.=.NO
<br>0320   20 e1 76 e4 f4 ff 0d de 88 15 44 ee 67 98 74 9c   .v.......D.g.t.
<br>0330   5d f1 b1 76 36 6b 2d bb 62 46 c1 a3 ae 88 a7 6b  ]..v6k-.bF.....k
<br>0340   e4 03 45 96 93 5c 61 b8 23 7a 25 ce fc 82 c8 f5  ..E..\a.#z%.....
<br>0350   a8 46 84 97 86 34 28 0d 93 21 e1 e7 0a 6a 6d f3  .F...4(..!...jm.
<br>0360   97 6b 17 57 25 42 d2 6a 48 fc a3 b2 b6 59 ed 5c  .k.W%B.jH....Y.\
<br>0370   eb 8c 02 79 37 64 c0 18 b1 34 14 b6 fb e7 13 97  ...y7d...4......
<br>0380   e8 77 b0 2b f8 6f fb e8 37 ee 38 f7 ea 2e d4 e1  .w.+.o..7.8.....
<br>0390   2b 5e d0 40 19 4c 75 21 6e bc 44 b3 5f bf a5 92  +^.@.Lu!n.D._...
<br>03a0   ca 25 5a 08 f7 86 bc 7f ca f0 01 fa ec 32 0a 9b  .%Z..........2..
<br>03b0   81 00 43 dd 9f bb f9 09 9b 7a 1b ac 1e ad c0 fe  ..C......z......
<br>03c0   97 61 20 a5 cf 14 04 b0 48 f7 7f f3 6e a1 41 5a  .a .....H...n.AZ
<br>03d0   18 02 c4 79 26 7a f1 90 e2 b6 1f af 5e 92 0d 01  ...y&z......^...
<br>03e0   42 f1 58 16 bf eb f4 f7 2f f9 72 86 db 77 05 54  B.X...../.r..w.T
<br>03f0   84 35 b3 55 7c e8 ee 4d 8d b1 ae a5 6e c3 10 90  .5.U|..M....n...
<br>0400   16 b3 86 5d 73 2f f7 db 4a bd 4c 44 dd b6 b7 20  ...]s/..J.LD... 
<br>0410   e3 87 2c 89 36 c3 d0 20 e5 ac 4e fc 87 ce 62 b3  ..,.6.. ..N...b.
<br>0420   3e c8 c4 db a5 a7 9c 84 fc 1a ae e5 49 67 22 bb  >...........Ig".
<br>0430   e3 5d 76 20 65 1d d6 a7 f7 45 cd 19 55 be 34 2a  .]v e....E..U.4*
<br>0440   15 b9 33 53 c9 ad 1e 41 5f 61 cc 34 41 e6 9f 80  ..3S...A_a.4A...
<br>0450   b8 d9 37 9f 60 2f c8 8c d9 68 31 12 d7 63 42 5d  ..7./...h1..cB]
<br>0460   37 61 0d a6 85 cb 5d 4a a8 41 63 f5 bb 8e 11 0a  7a....]J.Ac.....
<br>0470   4c c9 54 00 82 c3 5f 84 3d 07 12 58 0a 18 a5 47  L.T..._.=..X...G
<br>0480   c6 9d 81 16 0e 08 ee 92 2a 07 6a b7 97 28 1d cf  ........*.j..(..
<br>0490   4e 99 79 61 dc 44 2a 6c 05 09 eb 4b 20 75 72 a1  N.ya.D*l...K ur.
<br>04a0   50 46 7a 10 f4 17 c8 06 d1 37 c6 4a 10 08 29 ef  PFz......7.J..).
<br>04b0   04 56 99 94 72 51 25 72 7b 9e 46 f3 46 41 72 f0  .V..rQ%r{.F.FAr.
<br>04c0   39 19 52 34 78 da 94 d8 c4 67 81 69 c2 4b a6 f0  9.R4x....g.i.K..
<br>04d0   1a 5c e2 d0 90 bf 64 fe d0 56 84 af 41 8d 0d bb  .\....d..V..A...
<br>04e0   3d 8a fa 07 56 77 cb 27 9d b7 f3 1c 65 29 72 cb  =...Vw.'....e)r.
<br>04f0   2b dc 90 c2 0d 70 a2 ff 30 6c 98 1d 1f d0 56 97  +....p..0l....V.
<br>0500   f0 da f2 0d 3e 53 de c2 01 39 5f 59 9a 3c 14 34  ....>S...9_Y.<.4
<br>0510   5e af c8 02 42 dc 2a f8 de 93 ed 4a d4 96 a5 bc  ^...B.*....J....
<br>0520   39 1f a5 b2 b2 df ad 33 8b 98 77 27 58 5c f4 ff  9......3..w'X\..
<br>0530   dc c2 6e 75 36 c8 31 b6 83 7d 7b e4 1b f7 01 dc  ..nu6.1..}{.....
<br>0540   4a da c3 c1 fc 74 a6 93 3d 0f fe 04 99 c5 ef c4  J....t..=.......
<br>0550   c7 71 d8 7d 10 a2 a3 89 b9 0d ad 18 5f 57 a3 2e  .q.}........_W..
<br>0560   99 47 2d 2e 98 bd 91 20 bc 09 e1 94 41 01 7b 64  .G-.... ....A.{d
<br>0570   29 c7 94 c7 64 50 90 04 8b c2 9e a2 82 f9 e2 2c  )...dP.........,
<br>0580   bb ef 9b e7 6a 9a 08 66 ca d6 06 75 e5 fc a1 70  ....j..f...u...p
<br>0590   d0 88 6c cd 8c 7f d2 47 0f 48 d3 cd 87 75 38 c1  ..l....G.H...u8.
<br>05a0   20 b5 41 b3 a2 d9 88 b2 a9 af 59 08 9d 7f 48 ab   .A.......Y...H.
<br>05b0   a6 45 f1 9d 35 95 12 bf 0c 50 41 81 a4 78 10 33  .E..5....PA..x.3
<br>05c0   99 e2 63 43 5f 41 75 24 c6 d5 d4 b3 ff aa c8 c8  ..cC_Au$........
<br>05d0   e2 e9 61 ee bb 34 e3 83 ef 6c 77 3e e3 53 7e c7  ..a..4...lw>.S~.
<br>05e0   6f 56 85 ba 8a b2 6b 8c 38 c3 d9 8c              oV....k.8...
<br>
Responce: OutOfOrder
<br>
<br>0000   00 00 00 01 00 06 e0 db 55 d8 d4 79 32 00 08 00  ........U..y2...
<br>0010   45 00 05 dc 67 73 40 00 40 06 4a 51 c0 a8 01 02  E...gs@.@.JQ....
<br>0020   c0 a8 01 05 ba f4 00 8b f9 4d 30 f4 39 d6 33 00  .........M0.9.3.
<br>0030   80 10 00 e5 53 2c 00 00 01 01 08 0a d8 4c f3 d5  ....S,.......L..
<br>0040   00 1c 22 4c e9 8b 3e 81 fa 29 88 d2 db aa b8 13  .."L..>..)......
<br>0050   14 0c f3 0b b7 c5 f2 f4 51 cc 53 c7 1f a7 0b 68  ........Q.S....h
<br>0060   60 29 56 30 fb 55 03 98 6e 2c 4b 76 33 8a 3c 33  )V0.U..n,Kv3.<3
<br>0070   82 23 4d 14 f7 1c 23 31 5e 47 42 fe 50 e0 f3 e0  .#M...#1^GB.P...
<br>0080   98 da 88 da e5 76 3e 55 ba e7 a9 73 d3 e5 20 e3  .....v>U...s.. .
<br>0090   8a 25 38 32 8e 42 19 44 4e 08 aa 0d 8b 97 c3 66  .%82.B.DN......f
<br>00a0   96 79 42 0d 36 65 91 3e ac 11 87 31 a5 04 a2 0b  .yB.6e.>...1....
<br>00b0   db 32 93 f0 2f 67 26 69 49 19 f4 a9 1a dc 6c 7f  .2../g&iI.....l.
<br>00c0   db 5b 48 e8 d5 54 6f 6c af af d8 cf 9a fc 2e 1d  .[H..Tol........
<br>00d0   f9 3e e6 41 f7 7e ec 36 74 31 15 c4 74 ff 6a 88  .>.A.~.6t1..t.j.
<br>00e0   4c a1 1d 6f 7a 8d f7 cc 70 d6 02 fc 20 35 c7 af  L..oz...p... 5..
<br>00f0   aa 4d 91 b7 46 93 30 e1 37 65 30 ca a0 27 83 6e  .M..F.0.7e0..'.n
<br>0100   0f 94 07 5e f2 e1 e1 fb 23 a9 01 d9 3e be 8d c3  ...^....#...>...
<br>0110   13 02 e9 78 f3 de 95 44 db 00 f8 87 80 ad 64 cd  ...x...D......d.
<br>0120   0f 67 59 af d8 1f 01 43 df eb 80 3b 2a fd 50 f7  .gY....C...;*.P.
<br>0130   24 36 dd 82 d0 30 df 27 00 ee ac fa 30 0c 8f ff  $6...0.'....0...
<br>0140   8c 8e 49 b5 a8 9a b4 96 8a 86 04 61 da 24 b6 75  ..I........a.$.u
<br>0150   a4 f9 ce 0d 43 ff 29 03 5e f2 2c 50 62 fb d9 4e  ....C.).^.,Pb..N
<br>0160   05 5f bc 9a 13 33 39 4a 21 bb da f4 34 6a 14 6f  ._...39J!...4j.o
<br>0170   ff 8f 0a 91 58 90 fb 19 c4 0a 0c 67 0c 82 55 32  ....X......g..U2
<br>0180   06 d3 e2 19 bb 6f aa 98 e9 a9 45 a2 dd d5 fc 00  .....o....E.....
<br>0190   b2 e4 bf 8f 10 40 fb 12 5c a6 39 1f 8c 9b da 84  .....@..\.9.....
<br>01a0   ed 10 d9 1e 5c 83 04 80 8f 0a 61 b1 1d eb bb e1  ....\.....a.....
<br>01b0   1f 2b ff a9 37 1d 95 57 74 fd d4 31 9c 1e f5 0a  .+..7..Wt..1....
<br>01c0   b0 69 0c 29 a6 15 a4 9f 84 bc 6b 04 a0 87 de 23  .i.)......k....#
<br>01d0   2e d8 40 18 d3 27 d3 ee 0d 32 92 73 2f 6c 7e fb  ..@..'...2.s/l~.
<br>01e0   7f b9 99 cf 28 cc 3e da 62 71 05 8a 9f 6b 37 e4  ....(.>.bq...k7.
<br>01f0   0f 6f 38 1f e4 1d 92 86 8a a9 2b d1 75 1a b8 29  .o8.......+.u..)
<br>0200   20 2d 58 d3 ef 3a 4d f3 d6 f4 35 02 09 2b 6f 19   -X..:M...5..+o.
<br>0210   4c f3 c0 1c be 37 3b e2 03 41 4b 96 5e bb 69 4d  L....7;..AK.^.iM
<br>0220   aa 9a 3b f7 fb b7 94 60 27 48 a9 a5 a9 e3 2e b5  ..;....'H......
<br>0230   e5 6d 8c e6 64 29 dd dd 1a 96 ef b2 3f 9f e0 f5  .m..d)......?...
<br>0240   36 0d 5d 2e 43 30 24 97 ad 95 7f d1 fa 7d d1 de  6.].C0$......}..
<br>0250   ee e3 e3 3a a2 04 63 41 ce 1a ae f6 04 e1 eb dd  ...:..cA........
<br>0260   49 c8 0d aa 08 e1 2d 14 29 18 6b 08 e3 b3 0e ac  I.....-.).k.....
<br>0270   11 ad 1d 30 5e 39 84 24 c2 af 1b 75 1f 1a 64 62  ...0^9.$...u..db
<br>0280   b5 74 b4 4c e4 d1 14 ab cd 8a 12 32 db 7f 2a 4b  .t.L.......2..*K
<br>0290   b9 3b c9 52 f7 ae 60 ba 6b c7 06 43 d9 62 d9 bc  .;.R...k..C.b..
<br>02a0   f5 9f 2f 19 89 9d 52 64 6e 94 ae 70 6c 65 5f 73  ../...Rdn..ple_s
<br>02b0   7c e0 97 00 3b 6c 3f e9 b0 ca aa 34 cd 3b 6f 0d  |...;l?....4.;o.
<br>02c0   24 ac e9 f4 48 cb bc 70 3d d5 cf f7 46 80 af 9c  $...H..p=...F...
<br>02d0   3d ee f8 f2 e2 d5 f2 6e 57 7a 15 53 91 6f 4e fb  =......nWz.S.oN.
<br>02e0   3d cf 2d f9 80 77 8c bf 85 33 f4 f1 cd c3 8d 65  =.-..w...3.....e
<br>02f0   f9 9c 10 e9 f2 86 3e cd 98 7f c2 08 78 27 5a f9  ......>.....x'Z.
<br>0300   4c a9 2d 5a 55 38 cb 4b 85 d2 c1 c9 52 85 f5 d3  L.-ZU8.K....R...
<br>0310   b8 91 07 d8 af 59 b5 9a 52 3d c0 7d 20 0a 89 c8  .....Y..R=.} ...
<br>0320   2e dd 99 a3 ec 3e 6f f0 4e ee 91 df 76 7a fb 81  .....>o.N...vz..
<br>0330   e0 dd d0 b5 8f 19 64 dd b8 cb a6 11 35 c1 3e 0e  ......d.....5.>.
<br>0340   de 14 10 fe d3 57 dd 62 21 9c 24 43 ce d6 4b 35  .....W.b!.$C..K5
<br>0350   4d 1b 30 3a 70 c2 18 05 e7 30 10 77 7b a7 3a 01  M.0:p....0.w{.:.
<br>0360   e9 12 7a f2 d5 3d 76 0d 54 79 5a 89 ad ff 60 2e  ..z..=v.TyZ....
<br>0370   56 8a 88 cf 69 9f 57 9b d1 bc 81 d3 dd 38 db 8e  V...i.W......8..
<br>0380   49 78 b3 35 80 6f af 86 e7 5e c3 56 bd 20 7d 06  Ix.5.o...^.V. }.
<br>0390   d7 e1 90 90 dc 20 43 cc 89 50 a0 90 97 fd 01 a9  ..... C..P......
<br>03a0   f8 4f e9 b7 a0 c8 94 80 43 13 ba c0 a0 07 f9 22  .O......C......"
<br>03b0   02 e6 03 0a 14 41 b7 ec 9b 53 57 90 91 c9 f9 59  .....A...SW....Y
<br>03c0   74 6d 76 0f d2 d0 01 9b 69 8f 03 ed 69 8b 64 a1  tmv.....i...i.d.
<br>03d0   90 8a 0f 2e a2 f2 d7 68 32 a0 bf 3d bf 8a 7a c6  .......h2..=..z.
<br>03e0   0d 46 42 88 95 25 1c 9f dd 5a 86 4e 36 c9 9f 2c  .FB..%...Z.N6..,
<br>03f0   b2 e8 35 ac 86 7c 40 47 31 8f 25 9b 2a 9a a4 d4  ..5..|@G1.%.*...
<br>0400   4d fb 84 e0 6a 8e e2 b3 c0 e7 89 c8 b4 dd c9 a9  M...j...........
<br>0410   e5 ee 7a af 86 26 47 2b a1 5d 01 ff d7 b1 33 9b  ..z..&G+.]....3.
<br>0420   8f 5e 53 03 13 f5 9c c5 8d c0 1e 32 37 a5 6b 80  .^S........27.k.
<br>0430   09 a1 f7 7e 9d 63 7e 8c 06 58 84 89 1c 7e 6c 24  ...~.c~..X...~l$
<br>0440   0e 86 c0 06 81 ac 2c 85 7a 78 69 89 40 d0 47 0f  ......,.zxi.@.G.
<br>0450   fc 54 c1 9b 6f a6 ae dc a9 4c 74 5c 2d d4 e6 48  .T..o....Lt\-..H
<br>0460   d0 bc 30 6c 57 76 e0 6a b6 2c 5f ae 34 83 e6 1f  ..0lWv.j.,_.4...
<br>0470   2e c9 ca 2c a0 60 71 bf 67 fb 35 36 ea 3f 27 a2  ...,.q.g.56.?'.
<br>0480   ff 0e 54 b0 9c 22 c4 c0 83 be ee 02 3f 37 3d 22  ..T.."......?7="
<br>0490   d3 93 3a 2b 95 e1 9b fb 23 bc fc 45 1a ed 3d 73  ..:+....#..E..=s
<br>04a0   ce a2 29 c1 12 78 bb 67 20 83 ba 3d a7 4c ef ce  ..)..x.g ..=.L..
<br>04b0   04 91 69 f3 56 d4 87 84 3d fb 34 3c da 52 12 2c  ..i.V...=.4<.R.,
<br>04c0   3c 51 8d 69 64 33 2c db a4 64 b0 2a d6 8d ad aa  <Q.id3,..d.*....
<br>04d0   09 eb a5 c5 88 76 79 6c b1 b2 3c 7b 3a 38 e8 78  .....vyl..<{:8.x
<br>04e0   94 85 bb 7c 03 54 d3 46 76 1b 0a d2 c5 c2 cd ea  ...|.T.Fv.......
<br>04f0   a9 a1 11 14 6c 1c ac 39 15 9a 46 ef cf 77 87 69  ....l..9..F..w.i
<br>0500   9c 0f 53 72 05 5a 7f d1 3a 92 82 f6 9a d3 5b e1  ..Sr.Z..:.....[.
<br>0510   95 43 a4 6b 45 e7 9f 01 69 0c 6d e0 ee 4b 74 ed  .C.kE...i.m..Kt.
<br>0520   55 26 ef 83 84 b1 9b 1b 78 20 83 ae 64 ed 8c 8f  U&......x ..d...
<br>0530   5d 19 14 ed 45 2b 49 88 c7 7f 0b 89 97 19 3f 05  ]...E+I.......?.
<br>0540   42 42 59 b7 ab 56 f5 49 1f 4b b4 f7 cc 00 e8 98  BBY..V.I.K......
<br>0550   47 79 44 60 54 15 ad 8c 18 a5 9b 87 d0 b5 e8 cb  GyDT...........
<br>0560   3e 84 8e c5 3a 71 66 44 ec af 05 da 59 99 5d 8e  >...:qfD....Y.].
<br>0570   44 39 8f 0c 7d f1 28 04 b7 d0 da 4a 7a 15 8b cc  D9..}.(....Jz...
<br>0580   d7 a9 87 e9 32 ef e0 fb 45 81 4b d6 ea 0e ac 0c  ....2...E.K.....
<br>0590   3b 8a b2 14 80 b1 45 42 36 45 ff e6 79 6b cc b9  ;.....EB6E..yk..
<br>05a0   e4 ab 62 55 82 3a 5b 61 ac 84 99 33 f8 6b 8b b1  ..bU.:[a...3.k..
<br>05b0   67 5c a4 51 47 0d 7a bb 9b 3f 85 ed 3a f6 3a f2  g\.QG.z..?..:.:.
<br>05c0   68 07 75 1a f8 6e 98 5e 91 e0 fb 65 aa 00 7d 26  h.u..n.^...e..}&
<br>05d0   75 73 25 ca 75 99 01 1e 0e 34 f2 56 23 54 28 c7  us%.u....4.V#T(.
<br>05e0   72 9b 1c d7 8e b5 f8 37 9a cc e5 f4              r......7....
<br>
<br>0000   00 00 00 01 00 06 9c 2a 70 13 e5 2b a9 b3 08 00  .......*p..+....
<br>0010   45 00 00 34 4c 33 40 00 80 06 2b 39 c0 a8 01 05  E..4L3@...+9....
<br>0020   c0 a8 01 02 00 8b ba f4 39 d6 33 00 f9 4d 74 d4  ........9.3..Mt.
<br>0030   80 10 01 01 6d 59 00 00 01 01 08 0a 00 1c 22 54  ....mY........"T
<br>0040   d8 4c f3 d5                                      .L..
<br>
Registration:
<br>
<br>0000   00 00 00 01 00 06 9c 2a 70 13 e5 2b da 8e 08 00  .......*p..+....
<br>0010   45 00 00 34 4c 27 40 00 80 06 2b 45 c0 a8 01 05  E..4L'@...+E....
<br>0020   c0 a8 01 02 00 8b ba e8 ad 6e 9d f6 a0 3b 22 c5  .........n...;".
<br>0030   80 10 01 01 ca e4 00 00 01 01 08 0a 00 19 48 00  ..............H.
<br>0040   d8 4c 3d 40                                      .L=@
<br>
<br>0000   00 00 00 01 00 06 a0 ab 1b 5e 44 f2 00 00 08 00  .........^D.....
<br>0010   45 00 00 4e 00 00 40 00 40 11 b7 48 c0 a8 01 01  E..N..@.@..H....
<br>0020   c0 a8 01 05 b7 31 00 89 00 3a 82 47 03 e8 00 00  .....1...:.G....
<br>0030   00 01 00 00 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01        AAAAAAAAA..!..
<br>
<br>0000   00 04 00 01 00 06 9c 2a 70 13 e5 2b 00 00 08 00  .......*p..+....
<br>0010   45 00 00 ef 2b 80 00 00 80 11 8b 27 c0 a8 01 05  E...+......'....
<br>0020   c0 a8 01 01 00 89 b7 31 00 db e6 27 03 e8 84 00  .......1...'....
<br>0030   00 00 00 01 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01 00 00  AAAAAAAAA..!....
<br>0060   00 00 00 9b 06 44 45 53 4b 54 4f 50 2d 32 4f 48  .....DESKTOP-2OH
<br>0070   52 48 49 47 20 04 00 44 45 53 4b 54 4f 50 2d 32  RHIG ..DESKTOP-2
<br>0080   4f 48 52 48 49 47 00 04 00 57 4f 52 4b 47 52 4f  OHRHIG...WORKGRO
<br>0090   55 50 20 20 20 20 20 20 00 84 00 57 4f 52 4b 47  UP      ...WORKG
<br>00a0   52 4f 55 50 20 20 20 20 20 20 1e 84 00 57 4f 52  ROUP      ...WOR
<br>00b0   4b 47 52 4f 55 50 20 20 20 20 20 20 1d 04 00 01  KGROUP      ....
<br>00c0   02 5f 5f 4d 53 42 52 4f 57 53 45 5f 5f 02 01 84  .__MSBROWSE__...
<br>00d0   00 08 00 27 2a 28 ac 00 00 00 00 00 00 00 00 00  ...'*(..........
<br>00e0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
<br>00f0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00     ...............
<br>
Short Massage Will endup with a continuess request Header Response:
<br>
short Query:
<br>
<br>0000   00 00 00 01 00 06 9c 2a 70 13 e5 2b 74 21 08 00  .......*p..+t!..
<br>0010   45 00 00 3c 4c 3d 40 00 80 06 2b 27 c0 a8 01 05  E..<L=@...+'....
<br>0020   c0 a8 01 02 00 8b ba fe 3d ba ca 6f f7 ba fa 6a  ........=..o...j
<br>0030   a0 12 20 00 c8 c3 00 00 02 04 05 b4 01 03 03 08  .. .............
<br>0040   04 02 08 0a 00 20 c3 91 3c af 25 99              ..... ..<.%.
<br>
<br>0000   00 04 00 01 00 06 9c 2a 70 13 e5 2b 00 00 08 00  .......*p..+....
<br>0010   45 00 00 28 4c 40 40 00 80 06 2b 38 c0 a8 01 05  E..(L@@...+8....
<br>0020   c0 a8 01 02 00 8b ba fe 3d ba ca 70 f7 ba fa 6b  ........=..p...k
<br>0030   50 14 00 00 76 9d 00 00                          P...v...
<br>
<br>0000   00 00 00 01 00 06 9c 2a 70 13 e5 2b 39 19 08 00  .......*p..+9...
<br>0010   45 00 00 28 4c 40 40 00 80 06 2b 38 c0 a8 01 05  E..(L@@...+8....
<br>0020   c0 a8 01 02 00 8b ba fe 3d ba ca 70 f7 ba fa 6b  ........=..p...k
<br>0030   50 14 00 00 76 9d 00 00 54 ff 4a 57 d0 67        P...v...T.JW.g
<br>
<br>
Continues response:
<br>
NBNS:
<br>
<br>0000   00 04 00 01 00 06 e0 db 55 d8 d4 79 00 00 08 00  ........U..y....
<br>0010   45 00 00 35 66 75 40 00 40 06 50 f6 c0 a8 01 02  E..5fu@.@.P.....
<br>0020   c0 a8 01 05 ba fe 00 8b f7 ba fa 6a 3d ba ca 70  ...........j=..p
<br>0030   80 18 00 e5 dd c2 00 00 01 01 08 0a 3c af 2e 79  ............<..y
<br>0040   00 20 c3 91 30                                   . ..0
<br>
<br>0000   00 04 00 01 00 06 9c 2a 70 13 e5 2b 04 06 08 00  .......*p..+....
<br>0010   45 00 00 ef 2b 95 00 00 80 11 8b 12 c0 a8 01 05  E...+...........
<br>0020   c0 a8 01 01 00 89 cb 65 00 db d1 f3 03 e8 84 00  .......e........
<br>0030   00 00 00 01 00 00 00 00 20 43 4b 41 41 41 41 41  ........ CKAAAAA
<br>0040   41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA
<br>0050   41 41 41 41 41 41 41 41 41 00 00 21 00 01 00 00  AAAAAAAAA..!....
<br>0060   00 00 00 9b 06 44 45 53 4b 54 4f 50 2d 32 4f 48  .....DESKTOP-2OH
<br>0070   52 48 49 47 20 04 00 44 45 53 4b 54 4f 50 2d 32  RHIG ..DESKTOP-2
<br>0080   4f 48 52 48 49 47 00 04 00 57 4f 52 4b 47 52 4f  OHRHIG...WORKGRO
<br>0090   55 50 20 20 20 20 20 20 00 84 00 57 4f 52 4b 47  UP      ...WORKG
<br>00a0   52 4f 55 50 20 20 20 20 20 20 1e 84 00 57 4f 52  ROUP      ...WOR
<br>00b0   4b 47 52 4f 55 50 20 20 20 20 20 20 1d 04 00 01  KGROUP      ....
<br>00c0   02 5f 5f 4d 53 42 52 4f 57 53 45 5f 5f 02 01 84  .__MSBROWSE__...
<br>00d0   00 08 00 27 2a 28 ac 00 00 00 00 00 00 00 00 00  ...'*(..........
<br>00e0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
<br>00f0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00     ...............
<br>
<br>

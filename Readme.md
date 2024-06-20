A2COLDRESET is a controller card project for Apple 2 computers. The card has an external button, which, when pressed, performs cold restart of the computer, similar to what the open-apple reset does on the Apple 2e.

The benefit is for computers without this capability so that the computer does not need to go through a power cycle to do the cold restart - it is achieved at the push of a buton.

The project is based on my A2RAM128 Saturn clone card, as the circuitry is very similar. I only needed to add a diode and a resistor, reprogram the PLD and change the RAM for an ROM to hold some code.

In essence, what the card does is:

1. When "normal" RESET is issued the card is deactivated.
2. When the external button is pressed, the card activates and issues system RESET in sync with the button and inhibits the Apple2's ROM memory, activating its own ROM memory.
3. Upon release of the button, the $FFFC vector is issued, which starts the program on the ROM of the card.
4. The program self-moves to $0300 and excutes from there, in the meantime zeroing most of $0300-$03FF, including $03F3 anf $03F4.
5. The program disables the card by a softswitch - access to address $3003 (randomly chosen by me :))
6. The program then performs a relative jump to $(FFFC) - the original reset vector as the computer's ROM is already active. As the $03F4 checksum does not check out - the computer performs a cold reboot.

I hope it can be useful to someone.

The code listing:

E000-   A0 00       LDY   #$00

E002-   B9 00 E0    LDA   $E000,Y

E005-   99 00 03    STA   $0300,Y

E008-   C8          INY

E009-   D0 F7       BNE   $2002

E00B-   4C 0E 03    JMP   $030E

E00E-   AD 03 30    LDA   $3003

E011-   AD 03 30    LDA   $3003

E014-   6C FC FF    JMP   ($FFFC)

E017-   00          BRK

E018-   00          BRK

E019-   00          BRK

E01A-   00          BRK

.....   ..          ...   .......

FFF9-   00          BRK

FFFA-   00          BRK

FFFB-   E0 00       CPX   #$00

FFFD-   E0 00       CPX   #$00

FFFF-   E0 00       CPX   #$00


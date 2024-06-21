A2COLDRESET is a controller card project for Apple 2 computers. The card has an external button, which, when pressed, performs cold restart of the computer, similar to what the open-apple reset does on the Apple 2e.

The benefit is for computers without this capability so that the computer does not need to go through a power cycle to do the cold restart - it is achieved at the push of a buton.

The project is based on my A2RAM128 Saturn clone card, as the circuitry is very similar. I only needed to add a diode and a resistor, reprogram the PLD and change the RAM for a ROM to hold some code.

In essence, what the card does is:

1. When "normal" RESET is issued the card is deactivated.
2. When the external button is pressed, the card activates and issues system RESET in sync with the button and inhibits the Apple2's ROM memory, activating its own ROM memory.
3. Upon release of the button, the $FFFC vector is issued, which starts the program on the ROM of the card.
4. The program self-moves to $0300-$03FF and excutes from there, in the meantime zeroing most of $0300-$03FF, including $03F3 anf $03F4.
5. The program disables the card by a softswitch - access to address $3003 (randomly chosen by me :))
6. The program then performs a relative jump to $(FFFC) - the original reset vector as the computer's ROM is already active. As the $03F4 checksum does not check out - the computer performs a cold reboot.

I hope it can be useful to someone.

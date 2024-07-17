A2RESET is a controller card for Apple 2 computers. It has an external button, which, when pressed, performs cold restart of the computer, similar to what the open-apple reset does on the Apple 2e.

The benefit is for computers without this capability so that the computer does not need to go through a power cycle to do the cold restart - it is achieved at the push of a buton.

The project is based on my A2RAM128 Saturn clone card, as the circuitry is very similar. I only needed to add a diode and a resistor, reprogram the PLD and change the RAM for a ROM to hold some code.

In essence, what the card does is:

1. When "normal" RESET is issued the card is deactivated (default state).
2. When the external button is pressed, this drives the RESET line low and the card activates which is inhibiting the Apple2's ROM memory and activating its own ROM memory. Note that pressing the Reset key does not drive the button line low.
3. Upon release of the button, the $FFFC vector is issued, which starts the program on the ROM of the card.
4. The program self-moves to $0300 and excutes from there, in the meantime destroying the correct checksum between $03F3 and #A5 in location $03F4.
5. The program disables the card by a softswitch - access to address $3003 (randomly chosen by me :))
6. The program then performs a relative jump to $(FFFC) - the original reset vector as the computer's ROM is already active. As the $03F4 checksum does not check out - the computer performs a cold reboot.

I hope it can be useful to someone.

I have added a new functionality to the firmware - v2.0. If the button is pressed twice quickly within 1 second then the computer disables any anti-reset routines, performs a warm reset and drops to monitor.

# A2RESET card

A2RESET is a controller card for Apple 2 computers. It has an external button, which, when pressed, performs cold restart of the computer, similar to what the open-apple reset does on the Apple 2e.

The benefit is for computers without this capability so that the computer does not need to go through a power cycle to do the cold restart - it is achieved at the push of a buton.

The project is based on my A2RAM128 Saturn clone card, as the circuitry is very similar. I only needed to add a diode and a resistor, reprogram the PLD and change the RAM for a ROM to hold some code.

In essence, what the card does is:

## Version 1

1. When "normal" RESET is issued the card is deactivated (default state).
2. When the external button is pressed, this drives the RESET line low as well as activates the card which leads to inhibiting the Apple2's ROMs and activating the card's onboard ROM. Note that pressing the Reset key does not drive the button line low.
3. Upon release of the button, the $FFFC vector is fetched on the address bus, which starts the program on the ROM of the card.
4. The program self-moves to $0300 and excutes from there, in the meantime destroying the correct checksum between $03F3 EOR #A5 at location $03F4.
5. Next, the program disables the card ROM and enables the computer ROM by a softswitch - access to address $3003 (randomly chosen by me :))
6. The program then performs a relative jump to $(FFFC) - the original reset vector as the computer's ROMs are already active. As the $03F4 checksum does not check out - the computer performs a cold reboot.

I hope it can be useful to someone.

I have added a new functionality to the firmware - v2.0. If the button is pressed twice quickly within 1 second then the computer disables any anti-reset routines, performs a warm reset and drops to monitor. This feature can be useful for programs where there is an anti-reset implemented but the user wants to still break the program end explore what's in the memory. An example is the game Cannonball Blitz.

## Version 2

As a follow up I made a completely new design based on DMA functionality - called A2DMA-RESET, the design files are in subfolder "V2-DMA".

Upon activation the card creates a single DMA write cycle to address $03F4, while the data bus is not driven, leading to a random byte being written to $03F4. Then, the DMA cycle is ended and the RESET line is asserted briefly which causes cold reboot.

Room for improvements:

As the data bus is not driven, there is theoretical chance that the correct value is randomly hit and entered in $03F4, which would then do a warm reset, although the chances of this hapenning are negligible. Some very good solutions, although more complex, would be to be able to drive the data bus and to do one of:
  - execute 2 DMA cycles - one reading $03F3 and the second - writing the value obtained to $03F4
  - execute 2 DMA write cycles and writing the same byte twice in $03F3 and $03F4

Note: Some Language Cards may remain active despite reset being asserted and this might be remedied by accessing $C081.

## Version 3

The V3 design is the most minimalistic one so far - it uses the same principle as V1, but does not have an EPROM as the PLD can actually hold a few bytes, just enough for a very short program to scramble the contents of $03F4 and then call the reset subroutine. With some clever programming and wiring it all boils down to a single PLD and a few passive elements now.


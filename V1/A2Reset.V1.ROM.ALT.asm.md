; Final ROM version ORG $E000
; LISA Assembler
; ROM: AT28C64B 8K $E000-$FFFF
; BEFORE Flash set: $FFFC/D = 00 E0
;---------------------------------------
        ORG   $E000
        OBJ   $0800
COUT    EQU   $FDED
INIT    EQU   $FB39
HOME    EQU   $FC58
;------------------------ DEBUG ONLY
;       CLC  
;       BCC   START
;------------------------ DEBUG ONLY       
        BIT $C082
        SEI
        CLD
        LDX #$FF
H1      INC $4C
        DEX
        BNE H1
        LDX   #$0F
CLR     LDA   #$00
        STA   $03F0,X
        DEX
        BPL   CLR
        BIT $C050       ; graphics (hide text)
        BIT $C052       ; full screen
        BIT $C054       ; page 1
        BIT $C057       ; hi-res enable
        LDX #$FF
H2      INC $4C
        DEX
        BNE H2    
        LDY #$00
MOVE    LDA START,Y     ; move IO ROM > RAM
        STA $0800,Y
        LDA  #$A0       ; clear text screen
        STA  $0400,Y
        STA  $0500,Y
        STA  $0600,Y
        STA  $0700,Y
        INY
        BNE MOVE
        BIT $C051       ; text mode
        JMP $0800
;---------------------------------------
; runs at $0800 (relocatable)
;---------------------------------------
START   INC   $3003
        INC   $3003
        JSR   INIT        ; Reset system pointers
        JSR   HOME        ; Clear screen
        SEI
        CLD
        LDY   #$09        ; clear FROM $09
PAGE    STY   $4C         ; high byte of pointer
        LDA   #$AE        ; say '.'
        JSR   COUT         
        LDA   #$00        ; fill byte = zero
        STA   $4B         ; low byte always $00 
        TAY               ; Y = 0, start of page
BYTE    STA   ($4B),Y     ; write zero to page
        INY               ; next byte
        BNE   BYTE        ; loop until page done
        LDY   $4C         ; restore page number
        INY               ; advance to next page
        CPY   #$95        ; reached page $95?
        BNE   PAGE        ; no - do next page        
        JSR   HOME
        LDA   #$D3        ; say "Slot:"
        JSR   COUT
        LDA   #$EC
        JSR   COUT
        LDA   #$EF
        JSR   COUT
        LDA   #$F4
        JSR   COUT
        LDA   #$BA
        JSR   COUT
        LDA   #$A0
        JSR   COUT
GETKEY  LDA   $C000       ; read keyboard
        BPL   GETKEY      ; bit 7 clear (no key)
        STA   $C010       ; clear strobe
        AND   #$7F        ; Strip high bit
        CMP   #$30        ; '0' = monitor
        BEQ   MONITOR
        CMP   #$31        ; Less than '1'?
        BCC   RESET       ; Yes — not a slot
        CMP   #$38        ; '8' or above?
        BCS   RESET       ; Yes — not a slot
        AND   #$0F
        CLC
        ADC   #$C0
        STA   $EC
        LDA   #$00
        STA   $EB
        JSR   HOME
        JMP   ($00EB)     ; BOOT the slot
MONITOR JMP   $FF69       ; drop to monitor
RESET   JSR   INIT
        JSR   HOME
        SEI
        LDX   #$0F
KLR     LDA   #$00
        STA   $03F0,X
        DEX
        BPL   KLR
        JMP   $FAA6       ; cold reset
        HEX   0000000000000000
        HEX   0000000000000000
        END
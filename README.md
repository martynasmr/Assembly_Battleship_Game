# Assembly_Battleship_Game
A game we had to create using Assembly language and Atmel boards


;Program to sequence LEDs on port C, uses the MSB on switches to change direction 

;Stack and Stack Pointer Addresses 
.equ     SPH    =$3E              ;High Byte Stack Pointer Address 
.equ     SPL    =$3D              ;Low Byte Stack Pointer Address 
.equ     RAMEND =$25F             ;Stack Address 

;Port Addresses 
.equ     DDRA   =$1A              ;Port A Data Direction Register Address 
.equ     PINA   =$19              ;Port A Input Address 
.equ     PORTC  =$15              ;Port C Output Address 
.equ     DDRC   =$14              ;Port C Data Direction Register Address 
.equ     PORTB  =$18       	      ;Port B Output Address 
.equ     DDRB   =$17              ;Port B Data Direction Register Address
.equ     PORTD =$12                 ;Port D Output Address 
.equ     DDRD  =$11                 ;Port D Data Direction Register Address 
.equ     PIND  =$10                 ;Port D Input Address 


;Register Definitions 
.def 	 score  =r0
.def     leds   =r1               ;Register to store data for LEDs 
.def 	 lives  =r2		  		  ;Ammount of lives 
.def 	 loop	=r3
.def	 loopp  =r4
.def     ini    =r5
.def 	 temp4 	=r15
.def     temp   =r16              ;Temporary storage register
.def     count  =r17 
.def     torp   =r23              ;Register determining if the torpedo hit
.def	 temp2  =r26	   	      ;For ship comparing
.def 	 temp3  =r27 		      ;For checking if torpedo hit ship
 
.def     ZL     =r30              ;Define low byte of Z 
.def     ZH     =r31              ;Define high byte of Z 

;Program Initialisation 

startover1:
;set value for lives
ldi temp,$3F
mov ini,temp
out PORTB,ini
		ldi 	temp, $03
		mov 	lives, temp

;set initial delay value

		 ldi	temp, $FF
		 mov 	loop, temp
		 mov 	loopp, temp

;Set stack pointer to end of memory 

         ldi    temp,high(RAMEND) 
         out    SPH,temp          ;Load high byte of end of memory address 
         ldi    temp,low(RAMEND) 
         out    SPL,temp          ;Load low byte of end of memory address  

;Initialise Input Ports 

         ldi    temp,$00	
         out    DDRA,temp         ;Set Port A for input by sending $00 to direction register 

;Initialise Output Ports 
         ldi    temp,$ff	
         out    DDRC,temp         ;Set Port C for output by sending $FF to direction register 
         ldi   temp,$ff    
         out   DDRB,r16           ;Set Port B for output by sending $FF to direction register 

;Initialise Main Program 
ldi    ZL,low(table*2)   ;Set Z pointer to start of table 
ldi    ZH,high(table*2) 


;randomizer initiation bit
ldi r20, $00
mov temp4, r20
ldi temp3, $00
ldi r18, $07
ldi r19, $66
ldi r20, $01
ldi r21, $02
ldi r22, $E0
mov temp,r19
ldi r19, $03

; randomizer
randomize:
clr leds
out PORTC, leds
ldi r20, $01
ldi r21, $02
and r20, temp
and r21, temp
lsr r21
eor r21, r20
lsl r21
lsl r21
lsl r21
lsl r21
lsl r21
lsl r21
lsl r21
lsr temp
or temp, r21
mov leds, temp
BRPL rightships
BRMI leftships

rightships:
and leds, r18
jmp shipcheck

leftships:
and leds,r22
jmp shipcheck

;Main Program 
forever: out    PORTC,leds        ;Display leds to port C 
rcall torpedo
rcall  sdelay             	  ;Call delay subroutine 
tst leds
brmi   right           		  ;If switch 7= 1 chase right else if if switch 7 = 0 chase left
brpl   left
         
;Rotate leds left
left:    lsl    leds              ;Rotate leds left by 1 bit through carry flag 
out    PORTC,leds
rcall torpedo

rcall  sdelay 
tst leds
breq randomize
         rjmp   left

;Rotate leds right
right:   lsr    leds              ;Rotate leds right by 1 bit through carry flag 
out    PORTC,leds
rcall torpedo
rcall  sdelay 
tst leds
breq randomize
rjmp   right

;Ship fixers

negative1: 
ldi temp2, $E0
mov leds, temp2
jmp forever

negative2:
ldi temp2, $C0
mov leds, temp2
jmp forever

negative3:
ldi temp2, $80
mov leds, temp2
jmp forever


positive1:
ldi temp2, $7
mov leds, temp2
jmp forever

positive2:
ldi temp2, $3
mov leds, temp2
jmp forever

positive3:
ldi temp2, $1
mov leds, temp2
jmp forever


;Check for corect ships

shipcheck:
mov temp2, leds

cpi temp2, $5
breq positive1

cpi temp2, $7
breq positive1

cpi temp2, $6
breq positive2

cpi temp2, $3
breq positive2

cpi temp2, $4
breq positive3

cpi temp2, $2
breq positive3

cpi temp2, $1
breq positive3


cpi temp2, $0
breq randomize1

cpi temp2, $E0
breq negative1

cpi temp2, $A0
breq negative1

cpi temp2, $60
breq negative2

cpi temp2, $C0
breq negative2

cpi temp2, $40
breq negative3

cpi temp2, $20
breq negative3

cpi temp2, $80
breq negative3

jmp forever

randomize1:
jmp randomize


;delay section of code for the speed of the ships 
sdelay:  ldi	r24,$FF         ;Initialise 2nd loop counter
         sub    r24,loopp
loop4:   ldi 	r25,$FF         ;Initialise 1st loop counter
loop3:   dec    r25              ;Decrement the 1st loop counter 
         brne   loop3            ;and continue to decrement until 1st loop counter = 0 
         dec    r24              ;Decrement the 2nd loop counter 
         brne   loop4            ;If the 2nd loop counter is not equal to zero repeat the 1st loop, else continue 
         ret                   ;Return

;delay section of code for normal delays
delay:  ldi	r24,$FF
loopp2:	 ldi 	r25,$FF
loopp1:  dec 	r25
		 brne 	loopp1
		 dec	r24
		 brne 	loopp2
		 ret




;Torpedo
torpedo:

mov temp3,leds
in   torp,PIND        ;Read switches on switch light box

com torp

rcall delay

cpi	 torp,$01
breq torpedo1  
cpi	 torp,$02
breq torpedo2
cpi torp, $04
breq torpedo3
cpi torp, $08
breq torpedo4
cpi torp, $10
breq torpedo5
cpi torp, $20
breq torpedo6
cpi torp, $40
breq torpedo7
cpi torp, $80
breq torpedo8



ret



hit: 

ldi temp, $A
mov loop,temp
add loopp,loop
lpm
adiw   ZL,1 
out PORTB, score
clr leds
ldi temp,$FF
mov leds, temp
out PORTC,leds
rcall delay
jmp randomize


miss:

dec lives
cp lives, temp3
breq startover
jmp forever


;jump
startover:
jmp startover1

;torpedoes

torpedo1:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo2:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo3:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit
torpedo4:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo5:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo6:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo7:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

torpedo8:
and temp3,torp
cp temp3, temp4
breq miss
jmp hit

table:   .DB $06,$5B,$4F,$66,$6D,$7D,$07,$7F,$6F,$77,$7C,$39,$5E,$79,$71

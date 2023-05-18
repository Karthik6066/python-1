# python-1
ELEVATOR INTERFACING
ORG 	0H
sjmp	30h
org	30h
START:  CLR     A
MOV     R0,A	  	 /*Elevator at Ground Floor*/
 MOV     R1,A
LOOP1:  MOV     A,R1
 ORL     A,#0F0H	    /*Clear  all requests by clearing the flipflops*/
 mov P0,a 		
 MOV     DPTR,#FLOOR	/*move the floors look up table into DPTR*/   
 LOOP2:  MOV     A,P1	    /*Get the status of request switches by reading P1*/
   ORL     A,#0F0H	    /*Since the No.of Floors are 4*/
   MOV     R2,A		/*it checks for only lower nibble*/
    INC     A
     JZ      LOOP2	    /*Any request?No,loop back.*/
    LOOP3:  MOV     A,R2		/* Find the Floor number using Look up table*/
    RRC     A
     MOV     R2,A
     JNC     DECIDE
     INC     DPTR
     SJMP    LOOP3
      DECIDE: acall   DELAY	    /*Get the floor number*/
       CLR     A
       MOVC    A,@A+DPTR
       CJNE    A,1,L1	    /*Compare with current floor number*/
        SJMP    RESET	    /*if equal reset the request*/ 
        L1:     JC      DOWN	    /*other wise if lower elevator goes down*/
        UP:     INC     R1	    	/* if higher Elevator goes up*/
        MOV     A,R1
        ORL     A,#0F0H	    /*Elev moving Direction indicaton*/ 
        mov P0,a	    /*on LEDs*/
        SJMP    DECIDE
        DOWN:   DEC     R1	        /*Elevator goes down*/
        MOV     A,R1
        ORL     A,#0F0H	    /*Direction indication */
        mov		P0,a	    /*on LEDs*/
        SJMP    DECIDE
        RESET:  MOV     A,#05H      /*Turn off Corresponding request*/ 
        ADD     A,dpl	    /*indication LED */
        MOV     dpl,A		/*Get the corresponding code from look up table */
        CLR     A	   		/*to turn off the LED*/
        MOVC    A,@A+DPTR
        mov P0,a
        SJMP    LOOP1
       DELAY:  mov   r3,#0ffh
       decr:  mov	r4,#0ffh
        djnz   r4,$
         djnz	r3,decr
         ret        
			 /*Look up table for corresponding floors*/
        FLOOR:  DB      00H,03H,06H,09H,00H,0E0H,0D3H,0B6H,79H
      END

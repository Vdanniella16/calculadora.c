# calculadora.c
Calculadora con keypad y display de 7 segmentos
/*
 * parcial2
 *
 *  Created on: Micro 2024
 *      Author: valeriealvarez
 */


#include <stdint.h>
#include "stm32l053xx.h"

#if !defined(__SOFT_FP__) && defined(__ARM_FP)
  #warning "FPU is not initialized, but the project is compiling for an FPU. Please initialize the FPU before use."
#endif

#define ESTADO_ESPERANDO_PRIMER_DIGITO 0
#define ESTADO_ESPERANDO_SEGUNDO_DIGITO 1
#define ESTADO_REALIZANDO_OPERACION 2

void delayMs(int n);
void key_pressed();
void init();
void calculador();
void keyboard_config();
void print_bcd_7_segment_decoder_CC(uint8_t);

uint8_t digito1 = 0; 	 // guardo el primer valor
uint8_t digito2 = 0;   	// primer número
uint8_t operacion = 0; // segundo número nombrado.

uint8_t val = 0;

int main(void){

	init();

	while(1){
		keyboard_config();		
		key_pressed();			
		calculador();
	}

}


void init(){
// registros de reloj PUERTOS A, B Y B
	GPIOB->MODER &=~(1<<17);	
	GPIOB->MODER &=~(1<<5);		
	GPIOB->MODER &=~(1<<7);		
	GPIOB->MODER &=~(1<<9);		
	GPIOB->MODER &=~(1<<13);	
	GPIOB->MODER &=~(1<<15);	
	GPIOB->MODER &=~(1<<19);	
	RCC->IOPENR |=0x01<<0;		
	RCC->IOPENR |=0x01<<1;		
	RCC->IOPENR |=0x01<<2;		

	GPIOA->MODER &=~(1<<1);		//DISPLAY 0

}
void keyboard_config(){

	GPIOC->MODER &=~(1<<10);	
	GPIOC->MODER &=~(1<<11);
	GPIOC->PUPDR |=(1<<10);		    
	GPIOC->MODER &=~(1<<12);	
	GPIOC->MODER &=~(1<<13);
	// PULL UP DE KEYPAD (R Y C)
	GPIOC->PUPDR |=(1<<12);		   
	GPIOC->MODER &=~(1<<16);	
	GPIOC->MODER &=~(1<<17);
	GPIOC->PUPDR |=(1<<16);		    
	GPIOC->MODER &=~(1<<18);	
	GPIOC->MODER &=~(1<<19);
	GPIOC->PUPDR |=(1<<18);		   

}

void key_pressed(){
	if(!(GPIOC->IDR & 0x20)){		
		print_bcd_7_segment_decoder_CC(0x02);
		GPIOA->ODR |=(1<<1);
		delayMs(5);

	}else if(!(GPIOC->IDR & 0x40)){		
		print_bcd_7_segment_decoder_CC(0x05);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
	}else if(!(GPIOC->IDR & 0x100)){	
		print_bcd_7_segment_decoder_CC(0x08);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
	}else if(!(GPIOC->IDR & 0x200)){	
		print_bcd_7_segment_decoder_CC(0x00);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
	}
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){	
		print_bcd_7_segment_decoder_CC(0x01);	
			
	}else if(!(GPIOC->IDR & 0x40)){

		print_bcd_7_segment_decoder_CC(0x04);	
		GPIOA->ODR |=(1<<1);					
		delayMs(5);								
	}else if(!(GPIOC->IDR & 0x100)){	
		print_bcd_7_segment_decoder_CC(0x07);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
	}else if(!(GPIOC->IDR & 0x200)){	
		
		if(estado == ESTADO_ESPERANDO_PRIMER_DIGITO){ 
			digito1 = val;  // Almacena el primer dígito
			estado = ESTADO_ESPERANDO_SEGUNDO_DIGITO;  
	}
	else if(estado == ESTADO_ESPERANDO_SEGUNDO_DIGITO){ 
			digito2 = val;  
			estado = ESTADO_REALIZANDO_OPERACION;  
	}

				delayMs(5);
	}

	if(!(GPIOC->IDR & 0x20)){			
		if(estado == ESTADO_REALIZANDO_OPERACION){ 
			operacion = digito1 + digito2;
			print_bcd_7_segment_decoder_CC(operacion);	// REGRESA A SU ESTADO INICIAL 
		estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
		        }

		delayMs(5);

	}else if(!(GPIOC->IDR & 0x40)){		
		if(estado == ESTADO_REALIZANDO_OPERACION){ 
		operacion = digito2 - digito1;
		print_bcd_7_segment_decoder_CC(operacion);
		estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
		
		delayMs(5);

	}else if(!(GPIOC->IDR & 0x100)){	

		if(estado == ESTADO_REALIZANDO_OPERACION){ 
		operacion = digito2 * digito1;
		print_bcd_7_segment_decoder_CC(operacion);
		estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
					        }


	}
}

void print_bcd_7_segment_decoder_CC(uint8_t val){


digito1 = val; 

	if(estado == ESTADO_ESPERANDO_PRIMER_DIGITO){ 
			            digito1 = val;  
			            estado = ESTADO_ESPERANDO_SEGUNDO_DIGITO;  
			        }
			        else if(estado == ESTADO_ESPERANDO_SEGUNDO_DIGITO){ // Si estamos esperando el segundo dígito
			            digito2 = val;  // Almacena el segundo dígito
			            estado = ESTADO_REALIZANDO_OPERACION;  // Cambia al estado de realizar operación
			        }


	if(val==0x00){						//NUMERO 0 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
	}else if(val==0x01){				//NUMERO 1 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x02){				//ENCIENDE EL NUMERO 2 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x03){				//NUMERO 3 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x04){				//NUMERO 4 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x05){				//NUEMRO 5 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x06){				//NUEMRO 6 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x07){				//NUEMRO 7 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x08){				//NUEMRO 8 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x09){				//NUMERO 9 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x0A){				//ENCIENDE LA LETRA A
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x0B){				//ENCIENDE LA LETRA B
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x0C){				//ENCIENDE LA LETRA C
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
	}else if(val==0x0D){				//ENCIENDE LA LETRA D
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}
}

void delayMs(int n){	
	int i;
	for(; n > 0; n--)
		for(i = 0; i < 240; i++);
}

void calculador(){
	if (estado == 'A')
		operacion = digito1 + digito2;
	if(estado == 'B')
		operacion = digito1 - digito2;
	if (estado =='C')
		operacion = digito1 * digito2;
}

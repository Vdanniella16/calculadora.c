/*
 * parcial2
 *
 *  Created on: Micro 2024
 *      Author: valeriealvarez
 */


#include <stdint.h>
#include "stm32l053xx.h"

#if !defined(_SOFT_FP) && defined(_ARM_FP)
  #warning "FPU is not initialized, but the project is compiling for an FPU. Please initialize the FPU before use."
#endif

#define FUNCION_OBTENIENDO_DIGITO0 0
#define FUNCION_OBTENIENDO_DIGITO1 1
#define FUNCION_OBTENIENDO_DIGITO2 2

void demoraMs(int n);
void tecla_presionada();
void inicializar();
void calculadora();
void configuracion_teclado();
void imprimir_bcd_decodificador_7_segmentos(uint8_t);

uint8_t primer_digito = 0;    // Guarda el primer valor
uint8_t segundo_digito = 0;   // Primer número
uint8_t operacion = 0;        // Segundo número nombrado.

uint8_t val = 0;

int main(void){

    inicializar();

    while(1){
        configuracion_teclado();        
        tecla_presionada();          
        calculadora();
    }

}


void inicializar(){
// Registros de reloj PUERTOS A, B Y B
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

    GPIOA->MODER &=~(1<<1);        // DISPLAY 0

}
void configuracion_teclado(){

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

void tecla_presionada(){
    if(!(GPIOC->IDR & 0x20)){        
        imprimir_bcd_decodificador_7_segmentos(0x02);
        GPIOA->ODR |=(1<<1);
        demoraMs(5);

    }else if(!(GPIOC->IDR & 0x40)){        
        imprimir_bcd_decodificador_7_segmentos(0x05);
        GPIOA->ODR |=(1<<1);
        demoraMs(5);
    }else if(!(GPIOC->IDR & 0x100)){    
        imprimir_bcd_decodificador_7_segmentos(0x08);
        GPIOA->ODR |=(1<<1);
        demoraMs(5);
    }else if(!(GPIOC->IDR & 0x200)){    
        if(funcion == FUNCION_OBTENIENDO_DIGITO0){ 
            primer_digito = val;  // Almacena el primer dígito
            funcion = FUNCION_OBTENIENDO_DIGITO1;  
    }
    else if(funcion == FUNCION_OBTENIENDO_DIGITO1){ 
            segundo_digito = val;  
            funcion = FUNCION_OBTENIENDO_DIGITO2;  
    }

                demoraMs(5);
    }

    if(!(GPIOC->IDR & 0x20)){            
        if(funcion == FUNCION_OBTENIENDO_DIGITO2){ 
            operacion = primer_digito + segundo_digito;
            imprimir_bcd_decodificador_7_segmentos(operacion);    // REGRESA A SU ESTADO INICIAL 
        funcion = FUNCION_OBTENIENDO_DIGITO0;
                }

        demoraMs(5);

    }else if(!(GPIOC->IDR & 0x40)){        
        if(funcion == FUNCION_OBTENIENDO_DIGITO2){ 
        operacion = segundo_digito - primer_digito;
        imprimir_bcd_decodificador_7_segmentos(operacion);
        funcion = FUNCION_OBTENIENDO_DIGITO0;
        
        demoraMs(5);

    }else if(!(GPIOC->IDR & 0x100)){    

        if(funcion == FUNCION_OBTENIENDO_DIGITO2){ 
        operacion = segundo_digito * primer_digito;
        imprimir_bcd_decodificador_7_segmentos(operacion);
        funcion = FUNCION_OBTENIENDO_DIGITO0;
                        }


    }
}

void imprimir_bcd_decodificador_7_segmentos(uint8_t val){


primer_digito = val; 

    if(funcion == FUNCION_OBTENIENDO_DIGITO0){ 
                        primer_digito = val;  
                        funcion = FUNCION_OBTENIENDO_DIGITO1;  
                    }
                    else if(funcion == FUNCION_OBTENIENDO_DIGITO1){ // Si estamos esperando el segundo dígito
                        segundo_digito = val;  // Almacena el segundo dígito
                        funcion = FUNCION_OBTENIENDO_DIGITO2;  // Cambia al estado de realizar operación
                    }


    if(val==0x00){                        // NUMERO 0 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
    }else if(val==0x01){                // NUMERO 1 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
    }else if(val==0x02){                // ENCIENDE EL NUMERO 2 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
    }else if(val==0x03){                // NUMERO 3 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
    }else if(val==0x04){                // NUMERO 4 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
    }else if(val==0x05){                // NUMERO 5 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
    }else if(val==0x06){                // NUMERO 6 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
    }else if(val==0x07){                // NUMERO 7 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
    }else if(val==0x08){                // NUMERO 8 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
    }else if(val==0x09){                // NUMERO 9 EN EL DISPLAY
        GPIOB->BSRR=(0x3DC<<16);    // RESETA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
    }else if(val==0x0A){                // ENCIENDE LA LETRA A
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
    }else if(val==0x0B){                // ENCIENDE LA LETRA B
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
    }else if(val==0x0C){                // ENCIENDE LA LETRA C
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<8;    // SEGMENTO A
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<7;    // SEGMENTO F
    }else if(val==0x0D){                // ENCIENDE LA LETRA D
        GPIOB->BSRR=(0x3DC<<16);    // RESETEA TODOS LOS BITS A 0
        GPIOB->ODR |=0x01<<2;    // SEGMENTO B
        GPIOB->ODR |=0x01<<3;    // SEGMENTO C
        GPIOB->ODR |=0x01<<4;    // SEGMENTO D
        GPIOB->ODR |=0x01<<6;    // SEGMENTO E
        GPIOB->ODR |=0x01<<9;    // SEGMENTO G
    }
}

void demoraMs(int n){    
    int i;
    for(; n > 0; n--)
        for(i = 0; i < 240; i++);
}

void calculadora(){
    if (funcion == 'A')
        operacion = primer_digito + segundo_digito;
    if(funcion == 'B')
        operacion = primer_digito - segundo_digito;
    if (funcion =='C')
        operacion = primer_digito * segundo_digito;
}

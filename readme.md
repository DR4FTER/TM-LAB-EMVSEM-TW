# Tristan Wąsik, EM - Projekt Technika Mikroprocesorowa
RGB Mixer 
# Opis działania projektu
Moim projektem jest Mixer RGB. Jego działanie opiera się na potencjometrze - nim użytkownik wybiera poziom jasności danego koloru. W celu zmiany koloru należy wcisnąć przycisk. Sercem projektu jest AtMega32A-PU 16Mhz. Zasilanie od programatora bądź złącza jack 2.1/5.5mm (5-12VDC). Sygnalizacja zasilania odbywa się poprzez diodę LED. Zastosowano 3 porty PWM (2 porty 8-bitowe oraz 1 port 10-bitowy). Kolejne porty to port ADC służący rejestrowaniu statusu potencjometra oraz port IO dla przycisku. 
# Schemat w programie Eagle

# Płytka w programie Eagle 
![img](./hardware/scr1.jpg)

# Kod programu
```cpp
#include <avr/io.h>
#include <util/delay.h>

#define PWM0 (1<<PB3)		//nadanie nazw dla portow
#define PWM1B (1<<PD4)
#define PWM1A (1<<PD5)
#define PWM2 (1<<PD7)
#define button (1<<PC2)

static inline void initADC0(void) {	//ADC
ADMUX |= (1 << REFS0);
ADCSRA |= (1 << ADPS1) | (1 << ADPS0);
ADCSRA |= (1 << ADEN);
}

void initPWM0(unsigned int value) { //8bit
TCCR0 = 0b01101101;
OCR0 = value;
}

void initPWM2(unsigned char value) { //8bit
TCCR2 = 0b01101011;
OCR2 = value;
}
void initPWM1(unsigned int value) { //10bit
TCCR1B = 0b01101011;
TCCR1A = 0b01101011;
OCR1B = value;
OCR1A = value;
}


int main(void){

DDRD |= PWM2;		//inicjacja Wyjsc
DDRD |= PWM1B;
DDRD |= PWM1A;
DDRB |= PWM0;

PORTC |= button;	//inicjacja Wejsc

int a=0;
int ab=0;
int status=0;
initADC0();
while(1){
	ADCSRA |= (1 << ADSC); //start ADC conversion
	loop_until_bit_is_clear(ADCSRA, ADSC);
	a = ADC;
	if((a%4)==0){ //dzielenie dla PWM 8-bit
		ab=a/4;
	}

	if(!(PINC & button)){		//po przycinięciu zmiana kanału PWM
		if(status>2){
			status=0;
		}
		else{
			status++;
		}
		_delay_ms(400);
	}

	switch(status){			//wybranie na który kanał PWM mają zostać wysłane dane z ADC
	case 0:
		initPWM0(ab);
	break;
	case 1:
		initPWM1(a);
	break;
	case 2:
		initPWM2(ab);
	break;
	}


_delay_ms(50);
}

}
```
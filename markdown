# Projeto de Recuperação T3 - Miguel Laranja Monteiro
## Esquemática do Circuito - TinkerCAD

![](tinkercad.png  "Projeto Idealizado no TinkerCAD.")

 [Esquemática do Circuito](https://www.tinkercad.com/things/9FKIy5dUPci/editel?returnTo=%2Fdashboard%2Fdesigns%2Fcircuits&sharecode=4iEkKEv37yGjWVNIaTEM7jB7w_uO73MviFOFenGQmww).
## Código em Arduino
###### Código utilizado para a montagem do circuito referente ao Exercício 3 da Prática 10.


    const uint8_t sensor1_PIN        = A0;  // Sensor que mede a temperatura da serpentina
    const uint8_t sensor2_PIN        = A1;  // Sensor que mede a temperatura do reservatório
    const uint8_t bomba_PIN          = 8;   // Saída referente à Bomba
    const uint8_t LED_PIN              = 9;   // Saída LED que indica temperatura acima de 60°C


    const float tolerancia_C          = 1.0;   // diferença máxima aceitável (°C)
    const float limiteTemp_C    = 60.0;  // limite do sensor 2 (°C)
    const float histerese_C           = 2.0;   // limite de histerese
    const unsigned long intervalo_ms  = 1000;


    unsigned long ultimaLeitura = 0;
    bool temperaturaAlta = false;

    // LM35 com referência interna de 1,1 V: 1 LSB ≈ 1,074 mV ≈ 0,107 °C
    //Subrotina de leitura
    float lerTemperatura(uint8_t pino) {
    analogRead(pino);                
    int adc = analogRead(pino);        // leitura única
    return adc * 110.0 / 1024.0;       // (adc * 1,1 V / 1024) / 0,010 V/°C
    }


    void setup() {
    Serial.begin(9600);
    analogReference(INTERNAL);         // referência de 1,1 V interna (ATmega328P)

    pinMode(bomba_PIN, OUTPUT);
    pinMode(LED_PIN, OUTPUT);
    digitalWrite(bomba_PIN, LOW);
    digitalWrite(LED_PIN, LOW);
    }


    void loop() {
    unsigned long atualLeitura = millis();


    if (atualLeitura - ultimaLeitura >= intervalo_ms) {
    ultimaLeitura += intervalo_ms;   // período de 1 s sem deriva


    float t1 = lerTemperatura(sensor1_PIN);
    float t2 = lerTemperatura(sensor2_PIN);
    float diferenca = (t1 - t2);


    // Bomba: divergência entre os sensores
    bool divergente = (diferenca > tolerancia_C);


    // LED: sensor 2 acima de 60 °C (com histerese)
    if (t2 > limiteTemp_C) {
      temperaturaAlta = true;
      digitalWrite(LED_PIN, HIGH);
    } else if (t2 < limiteTemp_C - histerese_C) {
      temperaturaAlta = false;
      digitalWrite(LED_PIN, LOW);
    }


    if (diferenca > tolerancia_C){
      digitalWrite(bomba_PIN, HIGH);
    } else if (diferenca < tolerancia_C){ digitalWrite(bomba_PIN, LOW);}


    // Monitor serial com print dos principais parâmetros
    Serial.print("T1: ");  Serial.print(t1, 1);
    Serial.print(" C | T2: "); Serial.print(t2, 1);
    Serial.print(" C | Dif: "); Serial.print(diferenca, 1);
    Serial.print(divergente ? " C -> DIVERGENTE" : " C -> OK");
    if (temperaturaAlta) Serial.print(" | T2 alta");
    Serial.println();}} 

## Fluxograma - Arduino
##### Subrotina de Setup
```mermaid 
---
config:
  themeVariables:
    fontSize: 11px
  flowchart:
    rankSpacing: 18
---

graph TD
  A(Begin - Subrotina de Setup):::wide --> E[    
    const uint8_t sensor1_PIN        = A0;
    const uint8_t sensor2_PIN        = A1;
    const uint8_t bomba_PIN          = 8;
    const uint8_t LED_PIN              = 9; ]:::wide  --> 
    F[    
    const float tolerancia_C          = 1.0; 
  const float limiteTemp_C    = 60.0;  
    const float histerese_C           = 2.0;   
    const unsigned long intervalo_ms  = 1000; ]:::wide2 --> 
    G[    
    unsigned long ultimaLeitura = 0;
    bool temperaturaAlta = false;]:::wide --> 
    H(End)
  classDef wide padding-left: 80px,padding-right:200px; 
  classDef wide2 padding-left: 100px,padding-right:220px;
```

##### Subrotina de Leitura
```mermaid 
---
config:
  themeVariables:
    fontSize: 11px
  flowchart:
    rankSpacing: 18
---

graph TD
  A("float lerTemperatura(uint8_t pino) "):::wide 
  --> E["
  analogRead(pino);                
int adc = analogRead(pino);
"return adc * 110.0 / 1024.0;]:::wide --->
    H(End)
  classDef wide padding-left: 40px,padding-right:180px; 
```
##### Principal 
```mermaid 
---
config:
  themeVariables:
    fontSize: 11px
  flowchart:
    rankSpacing: 18
---
graph TD
  A("Void Setup()"):::wide --> E["Serial.begin(9600);
    analogReference(INTERNAL);"]:::wide ---> B["pinMode(bomba_PIN, OUTPUT);
    pinMode(LED_PIN, OUTPUT);"]:::wide  --> C["
    digitalWrite(bomba_PIN, LOW);
    digitalWrite(LED_PIN, LOW);"]:::wide -->
    H(End)
  classDef wide padding-left: 40px,padding-right: 160px; 
```
```mermaid 

graph TD
  A("Void Loop()"):::wide --> D{;;}:::small
  -->|1| E["unsigned long atualLeitura = millis();"]:::wide ---> B{" (atualLeitura - ultimaLeitura >= intervalo_ms)"}:::wide 
  B -->|0| B

  B -->|1| O["Serial.print(&quot T1: &quot);  Serial.print(t1, 1);
    Serial.print(&quot C | T2: &quot); Serial.print(t2, 1);
    Serial.print(&quot C | Dif: &quot); Serial.print(diferenca, 1);
    Serial.print(divergente ? &quot C -> DIVERGENTE&quot : &quot C -> OK&quot);"]:::wide5
  --> R{"(temperaturaAlta)"} 
    R -->|1| S["Serial.print(&quot | T2 alta&quot);
    Serial.println();"]:::wide5 --> B
    R -->|0| B

  B --> |1| C["ultimaLeitura += intervalo_ms;  
    float t1 = lerTemperatura(sensor1_PIN);
    float t2 = lerTemperatura(sensor2_PIN);
    float diferenca = (t1 - t2);
    bool divergente = (diferenca > tolerancia_C);"]:::wide2

    C --> M{"(diferenca > tolerancia_C)"}:::wide4
    

    M -->|1| N["digitalWrite(bomba_PIN, HIGH);"]:::wide --> B
    M -->|0| P{"(diferenca < tolerancia_C)"}

    P -->|1| Q["digitalWrite(bomba_PIN, LOW);"]:::wide -->B
     P -->|0| B

    C --> F{"(t2 > limiteTemp_C)"}:::wide3

    F -->|1| J["temperaturaAlta = true;
      digitalWrite(LED_PIN, HIGH);"]:::wide

    F -->|0| K{"(t2 < limiteTemp_C - histerese_C)"}:::wide 

    K -->|1| L["temperaturaAlta = false;
      digitalWrite(LED_PIN, LOW);"]:::wide -->B

    K-->|0| B

  classDef wide padding-left: 50px,padding-right:300px; 
  classDef wide2 padding-left: 100px,padding-right:400px;
  classDef wide3 padding-left: 10px,padding-right:150px;
  classDef wide4 padding-left: 10px,padding-right:190px;
  classDef wide5 padding-left: 200px,padding-right:450px;
```

## Código em C puro
###### Código em C puro desenvolvido a partir da tradução do código em C++ (Arduino) Exercício 3 da Prática 10.

``` 
 // Controle de bomba por diferença de temperatura - ATmega328P, C puro
// Sensor 1 (PC0/ADC0): temperatura da serpentina
// Sensor 2 (PC1/ADC1): temperatura do reservatório
// PB0: bomba | PB1: LED de temperatura acima de 60 °C

#ifndef F_CPU
#define F_CPU 16000000UL
#endif

#include <avr/io.h>
#include <avr/interrupt.h>
#include <stdint.h>
#include <stdbool.h>

#define SENSOR1_CANAL      0      // PC0 (ADC0)
#define SENSOR2_CANAL      1      // PC1 (ADC1)
#define BOMBA_PIN          PB0
#define LED_PIN            PB1

// Valores em décimos de grau Celsius
#define TOLERANCIA_X10     10     // 1,0 °C
#define LIMITE_TEMP_X10    600    // 60,0 °C
#define HISTERESE_X10      20     // 2,0 °C

#define BAUD               9600
#define UBRR_VALOR         (F_CPU / 16UL / BAUD - 1)   // 103 a 16 MHz

static volatile bool tick_1s = false;

// 16 MHz / 256 = 62500 Hz -> OCR1A = 62499
ISR(TIMER1_COMPA_vect) {
    tick_1s = true;
}

static void timer1_init(void) {
    TCCR1A = 0;
    TCCR1B = (1 << WGM12) | (1 << CS12);   // CTC, prescaler 256
    OCR1A  = 62499;
    TIMSK1 = (1 << OCIE1A);
}

static void adc_init(void) {
    // Referência interna de 1,1 V (REFS1:0 = 11), alinhado à direita
    ADMUX  = (1 << REFS1) | (1 << REFS0);
    // ADC ligado, prescaler 128 (125 kHz)
    ADCSRA = (1 << ADEN) | (1 << ADPS2) | (1 << ADPS1) | (1 << ADPS0);
    // Desliga o buffer digital em PC0 e PC1 (reduz ruído e consumo)
    DIDR0  = (1 << ADC0D) | (1 << ADC1D);
}

static uint16_t adc_ler(uint8_t canal) {
    ADMUX = (ADMUX & 0xF0) | (canal & 0x0F);
    ADCSRA |= (1 << ADSC);
    while (ADCSRA & (1 << ADSC)) { }
    return ADC;
}

// LM35 (10 mV/°C), referência 1,1 V: temp_x10 = adc * 1100 / 1024
static int16_t ler_temperatura_x10(uint8_t canal) {
    adc_ler(canal);                        // leitura descartada
    uint16_t adc = adc_ler(canal);         // leitura única
    return (int16_t)(((uint32_t)adc * 1100UL) / 1024UL);
}

static void uart_init(void) {
    UBRR0H = (uint8_t)(UBRR_VALOR >> 8);
    UBRR0L = (uint8_t)(UBRR_VALOR);
    UCSR0B = (1 << TXEN0);
    UCSR0C = (1 << UCSZ01) | (1 << UCSZ00);   // 8 bits, 1 stop, sem paridade
}

static void uart_putc(char c) {
    while (!(UCSR0A & (1 << UDRE0))) { }
    UDR0 = c;
}

static void uart_puts(const char *s) {
    while (*s) uart_putc(*s++);
}

// Imprime um valor em décimos de grau como "xx.x" (aceita negativos)
static void uart_print_x10(int16_t v) {
    char buf[6];
    uint8_t n = 0;

    if (v < 0) {
        uart_putc('-');
        v = -v;
    }

    uint16_t inteiro  = (uint16_t)v / 10;
    uint8_t  decimal  = (uint8_t)((uint16_t)v % 10);

    if (inteiro == 0) {
        buf[n++] = '0';
    } else {
        while (inteiro > 0) {
            buf[n++] = '0' + (inteiro % 10);
            inteiro /= 10;
        }
    }
    while (n > 0) uart_putc(buf[--n]);

    uart_putc('.');
    uart_putc('0' + decimal);}

int main(void) {
    bool temperaturaAlta = false;

    // PB0 (bomba) e PB1 (LED) como saída, inicialmente em nível baixo
    DDRB  |= (1 << BOMBA_PIN) | (1 << LED_PIN);
    PORTB &= ~((1 << BOMBA_PIN) | (1 << LED_PIN));

    uart_init();
    adc_init();
    timer1_init();
    sei();

    while (1) {
    if (tick_1s) {
            tick_1s = false;

            int16_t t1 = ler_temperatura_x10(SENSOR1_CANAL);
            int16_t t2 = ler_temperatura_x10(SENSOR2_CANAL);
            int16_t diferenca = t1 - t2;

            bool divergente = (diferenca > TOLERANCIA_X10);

            // LED: sensor 2 acima de 60 °C (com histerese)
            if (t2 > LIMITE_TEMP_X10) {
                temperaturaAlta = true;
                PORTB |= (1 << LED_PIN);
            } else if (t2 < (LIMITE_TEMP_X10 - HISTERESE_X10)) {
                temperaturaAlta = false;
                PORTB &= ~(1 << LED_PIN);
            }

            // Bomba: liga se diferença > tolerância, desliga se < tolerância
            if (diferenca > TOLERANCIA_X10) {
                PORTB |= (1 << BOMBA_PIN);
            } else if (diferenca < TOLERANCIA_X10) {
                PORTB &= ~(1 << BOMBA_PIN);
            }

            // Monitor serial
            uart_puts("T1: ");
            uart_print_x10(t1);
            uart_puts(" C | T2: ");
            uart_print_x10(t2);
            uart_puts(" C | Dif: ");
            uart_print_x10(diferenca);
            uart_puts(divergente ? " C -> DIVERGENTE" : " C -> OK");
            if (temperaturaAlta) uart_puts(" | T2 alta");
            uart_puts("\r\n");
        }
    }
}
```







# MSM5265-GET-DATA
Код для эмуляции контролера LCD, этот код для BMW E34
Код можно значительно улучшить используя массив всего из 20 байт, возможно это позже и сделаю))


```c++

#define CLOCK 2  // Тактирующий сигнал с приборки
#define LOAD 3  // Сигнал на отправку данных с приборки на дисплей
#define DATA PD4  // Данные


volatile uint8_t c = 0; // Счетчик для массива данных

volatile uint8_t buf[200];  // Буфер массива даных
volatile uint32_t odo = 0;  // Пробег основной
volatile uint16_t dayOdo = 0; //Пробег суточный

uint32_t odoFiltr = 0;  // Пробег основной
uint16_t dayOdoFiltr = 0; //Пробег суточный

//const uint8_t _0[] PROGMEM = {113, 115, 33, 114, 112, 34, 35}; // Последняя цифра в суточном пробеге

byte num(byte by) {             // Функция парсинга маски битов данных в цифры
  if (by == 96)       return 1;
  else if (by == 62)  return 2;
  else if (by == 124) return 3;
  else if (by == 91)  return 4;
  else if (by == 93)  return 5;
  else if (by == 125) return 6;
  else if (by == 19)  return 7;
  else if (by == 127) return 8;
  else if (by == 95)  return 9;
  else if (by == 119) return 0;
  else odo = 255 * 100000;
}


void cl() {   // Прерывание по тактирующему сигналу от приборки, считываем состояние бита записываем его в массив
  uint8_t pin = (PIND & (1 << PD4)) > 0 ? 1 : 0;
  buf[c] = pin;
  c++;
}

void  load() {   // Прерывания по загрузке данных, отсчитываем от этого сигнала 160 бит и получаем свои данные, 160 бит взято из даташита на m5265
  byte a = (buf[c - 3] << 0) | (buf[c - 2] << 1) | (buf[c - 1] << 2) | (buf[c - 83] << 3) | (buf[c - 4] << 4) | (buf[c - 81] << 5) | (buf[c - 82] << 6);
  odo = num(a) * 100000;
  a = 0;
  a = (buf[c - 85] << 0) | (buf[c - 87] << 1) | (buf[c - 5] << 2) | (buf[c - 86] << 3) | (buf[c - 84] << 4) | (buf[c - 6] << 5) | (buf[c - 7] << 6);
  odo += num(a) * 10000;
  a = 0;
  a = (buf[c - 10] << 0) | (buf[c - 9] << 1) | (buf[c - 8] << 2) | (buf[c - 90] << 3) | (buf[c - 11] << 4) | (buf[c - 88] << 5) | (buf[c - 89] << 6);
  odo += num(a) * 1000;
  a = 0;
  a = (buf[c - 92] << 0) | (buf[c - 94] << 1) | (buf[c - 12] << 2) | (buf[c - 93] << 3) | (buf[c - 91] << 4) | (buf[c - 13] << 5) | (buf[c - 14] << 6);
  odo += num(a) * 100;
  a = 0;
  a = (buf[c - 17] << 0) | (buf[c - 16] << 1) | (buf[c - 15] << 2) | (buf[c - 97] << 3) | (buf[c - 18] << 4) | (buf[c - 95] << 5) | (buf[c - 96] << 6);
  odo += num(a) * 10;
  a = 0;
  a = (buf[c - 99] << 0) | (buf[c - 101] << 1) | (buf[c - 19] << 2) | (buf[c - 100] << 3) | (buf[c - 98] << 4) | (buf[c - 20] << 5) | (buf[c - 21] << 6);
  odo += num(a);

  a = 0;
  a = (buf[c - 24] << 0) | (buf[c - 23] << 1) | (buf[c - 22] << 2) | (buf[c - 104] << 3) | (buf[c - 25] << 4) | (buf[c - 102] << 5) | (buf[c - 103] << 6);
  dayOdo = num(a) * 100;
  a = 0;
  a = (buf[c - 106] << 0) | (buf[c - 108] << 1) | (buf[c - 26] << 2) | (buf[c - 107] << 3) | (buf[c - 105] << 4) | (buf[c - 27] << 5) | (buf[c - 28] << 6);
  dayOdo += num(a) * 10;
  a = 0;
  a = (buf[c - 31] << 0) | (buf[c - 30] << 1) | (buf[c - 29] << 2) | (buf[c - 111] << 3) | (buf[c - 32] << 4) | (buf[c - 109] << 5) | (buf[c - 110] << 6);
  dayOdo += num(a);

  c = 0;
  memset(buf, 0, sizeof(buf));
}

void setup() {
  Serial.begin(115200);
  attachInterrupt(0, cl, RISING);
  attachInterrupt(1, load, RISING);
  pinMode(CLOCK, 0);
  pinMode(LOAD, 0);
  pinMode(DATA, 0);
}

void loop() {  
  if (millis() > lastMillis + 100 && odo < 0xF423F && dayOdo < 0x3E7) {
    asm("cli");
    odoFiltr = odo;  // Пробег основной
    dayOdoFiltr = dayOdo;
    Serial.print(odo);
    Serial.print("\t");
    Serial.print(dayOdo);
    Serial.println("\t");
    asm("sei");
  }
}
```

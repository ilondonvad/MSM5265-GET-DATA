### **Новая версия кода**

Устранено:
1. Пропадание пробега
2. Неправильность считывания пробега
3. Получение мусора вместо данных
4. и тд..................................................................



```c++
#define CLOCK 2  // Тактирующий сигнал с приборки
#define LOAD 3  // Сигнал на отправку данных с приборки на дисплей
#define DATA PD4  // Данные
#define LIGHT 5

volatile uint32_t buf[4];
volatile uint32_t odo = 0;  // Пробег основной
volatile uint16_t dayOdo = 0; //Пробег суточный

byte num(byte by) {             // Функция парсинга маски битов данных в цифры
  if (by == 3 || by == 6)       return 1;
  else if (by == 181 || by == 211) return 2;
  else if (by == 151 || by == 87)  return 3;
  else if (by == 71 || by == 102)  return 4;
  else if (by == 214 || by == 117) return 5;
  else if (by == 246 || by == 245)  return 6;
  else if (by == 19 || by == 7) return 7;
  else if (by == 247)  return 8;
  else if (by == 215 || by == 119) return 9;
  else if (by == 243 || by == 183) return 0;
}


ISR(INT0_vect) {
  buf[0] |= ((PIND & (1 << PD4)) >> PD4);
  buf[3] = (buf[3] << 1) | (buf[2] >> 31);
  buf[2] = (buf[2] << 1) | (buf[1] >> 31);
  buf[1] = (buf[1] << 1) | (buf[0] >> 31);
  buf[0] <<= 1;
}



ISR(INT1_vect) {
  asm("cli");
  
  byte _1 = num(((buf[0] >> 1 & 15) << 4) | (buf[2] >> 17 & 7));
  byte _2 = num((buf[0] >> 5 & 7) | ((buf[2] >> 20 & 15) << 4));
  byte _3 = num(((buf[0] >> 8 & 15) << 4) | (buf[2] >> 24 & 7));
  byte _4 = num((buf[0] >> 12 & 7) | ((buf[2] >> 27 & 15) << 4));
  byte _5 = num(((buf[0] >> 15 & 15) << 4) | (((buf[2] >> 31) | (buf[3] & 3 << 1)) & 7));
  byte _6 = num((buf[0] >> 19 & 7) | ((buf[3] >> 2 & 15) << 4));

  byte _7 = num(((buf[0] >> 22 & 15) << 4) | (buf[3] >> 6 & 7));
  byte _8 = num((buf[0] >> 26 & 7) | ((buf[3] >> 9 & 15) << 4));
  byte _9 = num(((((buf[0] >> 29)) | ((buf[1] << 3)) & 15) << 4) | (buf[3] >> 13 & 7));


  odo = (_1*100000)+(_2*10000)+(_3*1000)+(_4*100)+(_5*10)+(_6);
  dayOdo = (_7*100)+(_8*10)+(_9);

  buf[3] = 0;
  buf[2] = 0;
  buf[1] = 0;
  buf[0] = 0;
  asm("sei");
}


void setup() {
  Serial.begin(115200);
  pinMode(CLOCK, INPUT);
  pinMode(LOAD, INPUT);
  pinMode(DATA, INPUT);
  EICRA |= (1 << ISC00) | (1 << ISC01);
  EICRA |= (1 << ISC10) | (1 << ISC11);
  EIMSK |= (1 << INT0) | (1 << INT1);
  sei();
}

void loop() {
 
}

```


### Структура данных
![image](https://github.com/user-attachments/assets/67847d97-293d-4042-9bd6-8ec5d8ec6579)
Данные представляют собой массив из 4х 32 битных байта, так как контроллер дисплея представляет собой сдвиговый регистр на 160 бит, некоторые из них не нужны.

### Код страницы для анализа данных
```html
<html>
<style>
    .byte {
        display: flex;
        flex-direction: column;
        border: 1px solid gray;
        padding: 5px;
        height: 50px;
        width: 20px;
        justify-content: center;
        align-items: center;
    }

    body {
        /* display: flex; */
        /* sflex-direction: column; */
    }

    .row1,
    .row2,
    .row3,
    .row4 {
        display: flex;
    }
</style>

<body>
    <button id="connect"></button>
    <div class="row1"></div>
    <div class="row2"></div>
    <div class="row3"></div>
    <div class="row4"></div>
    <script>
        let body = document.querySelector("body");
        //let odo = ["335543", "004"]
        let odo = ["888888", "888"]
        for (let i = 0; i < 128; i++) {
            if (i < 32) document.querySelector(".row1").innerHTML += `<div class="byte" style="background-color:${checkByte(i)};">
                                                                        <span>${Math.floor(i / 32)}</span>
                                                                        <span>${i}</span>
                                                                        <span>${Math.floor(i % 32)}</span>
                                                                        </div>`
            else if (i > 31 && i < 64) document.querySelector(".row2").innerHTML += `<div class="byte" style="background-color:${checkByte(i)};">
                                                                        <span>${Math.floor(i / 32)}</span>
                                                                        <span>${i}</span>
                                                                        <span>${Math.floor(i % 32)}</span>
                                                                        </div>`
            else if (i > 63 && i < 96) document.querySelector(".row3").innerHTML += `<div class="byte" style="background-color:${checkByte(i)};">
                                                                        <span>${Math.floor(i / 32)}</span>
                                                                        <span>${i}</span>
                                                                        <span>${Math.floor(i % 32)}</span>
                                                                        </div>`
            else if (i > 95 && i < 128) document.querySelector(".row4").innerHTML += `<div class="byte" style="background-color:${checkByte(i)};">
                                                                        <span>${Math.floor(i / 32)}</span>
                                                                        <span>${i}</span>
                                                                        <span>${Math.floor(i % 32)}</span>
                                                                        </div>`

        }


        function drawByteOdo(arr, num) {

            return arr.map((value, index) => {
                if (num == 0) {
                    switch (index) {
                        case 0: return value;
                        case 1: return value;
                        case 2: return value;
                        case 3: return 255;
                        case 4: return value;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 1) {
                    switch (index) {
                        case 0: return 255;
                        case 1: return 255;
                        case 2: return 255;
                        case 3: return 255;
                        case 4: return 255;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 2) {
                    switch (index) {
                        case 0: return 255;
                        case 1: return value;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return value;
                        case 6: return 255;
                    }
                }
                else if (num == 3) {
                    switch (index) {
                        case 0: return 255;
                        case 1: return 255;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 4) {
                    switch (index) {
                        case 0: return value;
                        case 1: return 255;
                        case 2: return 255;
                        case 3: return value;
                        case 4: return 255;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 5) {
                    switch (index) {
                        case 0: return value;
                        case 1: return 255;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return 255;
                        case 6: return value;
                    }
                }
                else if (num == 6) {
                    switch (index) {
                        case 0: return value;
                        case 1: return value;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return 255;
                        case 6: return value;
                    }
                }
                else if (num == 7) {
                    switch (index) {
                        case 0: return 255;
                        case 1: return 255;
                        case 2: return value;
                        case 3: return 255;
                        case 4: return 255;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 8) {
                    switch (index) {
                        case 0: return value;
                        case 1: return value;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return value;
                        case 6: return value;
                    }
                }
                else if (num == 9) {
                    switch (index) {
                        case 0: return value;
                        case 1: return 255;
                        case 2: return value;
                        case 3: return value;
                        case 4: return value;
                        case 5: return value;
                        case 6: return value;
                    }
                }
            })
        }

        function checkByte(i) {
            //i = i + 1;
            let _1 = drawByteOdo([3, 2, 1, 83, 4, 81, 82], +odo[0][0])
            let _2 = drawByteOdo([85, 87, 5, 86, 84, 6, 7], +odo[0][1])
            let _3 = drawByteOdo([10, 9, 8, 90, 11, 88, 89], +odo[0][2])
            let _4 = drawByteOdo([92, 94, 12, 93, 91, 13, 14], +odo[0][3])
            let _5 = drawByteOdo([17, 16, 15, 97, 18, 95, 96], +odo[0][4])
            let _6 = drawByteOdo([99, 101, 19, 100, 98, 20, 21], +odo[0][5])

            let _7 = drawByteOdo([24, 23, 22, 104, 25, 102, 103], +odo[1][0])
            let _8 = drawByteOdo([106, 108, 26, 107, 105, 27, 28], +odo[1][1])
            let _9 = drawByteOdo([31, 30, 29, 111, 32, 109, 110], +odo[1][2])
            let _10 = [116, 41,113,115,33,112,34,35]


            if (_1.includes(i)) {
                return "#a90300ba";
            } else if (_2.includes(i)) {
                return "#00ff0055";
            }
            else if (_3.includes(i)) {
                return "#0000ff55";
            }
            else if (_4.includes(i)) {
                return "#ff003355";
            }
            else if (_5.includes(i)) {
                return "#04238f55";
            }
            else if (_6.includes(i)) {
                return "#6cbb00";
            }
            else if (_7.includes(i)) {
                return "rgba(1,1,1,0.8)";
            }
            else if (_8.includes(i)) {
                return "rgba(50,1,1,0.8)";
            }
            else if (_9.includes(i)) {
                return "rgba(1,50,20,0.8)";
            }
            else if (_10.includes(i)) {
                return "lightgray";
            }
        }



        let port;
        let reader;
        let output = document.getElementById('output');

        document.getElementById('connect').addEventListener('click', async () => {
            // Запрашиваем доступ к последовательному порту
            port = await navigator.serial.requestPort();
            await port.open({ baudRate: 155200 });

            // Чтение данных из порта
            reader = port.readable.getReader();
            readLoop();
        });

        async function readLoop() {
            while (true) {
                const { value, done } = await reader.read();
                if (done) {
                    reader.releaseLock();
                    break;
                }

                // Преобразуем полученные байты в hex
                const hexOutput = Array.from(new Uint8Array(value))
                    .map(byte => byte.toString(16).padStart(2, '0')) // Конвертация в hex
                    .join(' '); // Соединение в строку

                console.log(hexOutput); // Выводим в консоль
            }
        }
    </script>

</body>

</html>
```

```js
let body = document.querySelector("body");
        //let odo = ["335543", "004"]     - реальный пробег
        let odo = ["888888", "888"]        - пробег для теста
        for (let i = 0; i < 128; i++) {
...
```

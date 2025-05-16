# 📟 Projeto de Monitoramento Ambiental - Vinheria Agnello

Este projeto foi desenvolvido para monitorar **temperatura**, **umidade** e **luminosidade** em um ambiente (como uma vinheria), utilizando sensores e um display LCD para exibir as informações em tempo real. Também usa LEDs e um buzzer para alertar quando algo está fora do ideal. 🌡️💡🔔

---

## 🔧 O que esse projeto faz

- Mede a **temperatura** e a **umidade** usando o sensor **DHT11**
- Mede a **luminosidade** com um **sensor LDR**
- Exibe as informações em um **display LCD**
- Usa **LEDs coloridos** e um **buzzer** para alertar sobre o estado do ambiente:
  - 🟢 Verde = tudo certo
  - 🟡 Amarelo = atenção
  - 🔴 Vermelho + som = alerta 🔍📊📢

---

## 🧰 Componentes utilizados

- Arduino (Uno, Mega, etc.)
- Sensor DHT11 (temperatura e umidade)
- Sensor LDR (luminosidade)
- Display LCD 
- 3 LEDs (verde, amarelo, vermelho)
- 1 buzzer
- Resistores e jumpers 🧪📦🔌

---

## ⚙️ Como funciona

1. O Arduino faz **leituras múltiplas** para garantir uma média mais precisa.
2. Com base nos valores médios, ele decide:
   - Se o ambiente está **escuro**, **meia-luz** ou **claro**
   - Se a **temperatura** está dentro da faixa ideal (entre 10°C e 15°C)
   - Se a **umidade** está adequada (entre 50% e 70%)
3. Dependendo da situação, ele:
   - Mostra mensagens no LCD
   - Acende LEDs específicos
   - Aciona o buzzer se necessário 🧠⚡📈

---

## 🖼️ Exemplo de funcionamento

| Situação               | LED aceso   | Buzzer | Ação no LCD          |
| ---------------------- | ----------- | ------ | -------------------- |
| Ambiente escuro        | 🟢 Verde    | ❌      | "Ambiente escuro"    |
| Ambiente meia-luz      | 🟡 Amarelo  | ❌      | "Ambiente meia luz"  |
| Ambiente claro         | 🔴 Vermelho | ✅      | "Ambiente claro"     |
| Temperatura fora ideal | 🟡 Amarelo  | ✅      | "Temp. Alta/Baixa"   |
| Umidade fora ideal     | 🔴 Vermelho | ✅      | "Umidade Alta/Baixa" 🎛️🧾📋

---

## 🚀 Como usar

1. Conecte os componentes conforme os pinos definidos no código:
   - LDR no pino A0
   - DHT11 no pino 5
   - LEDs: verde (pino 10), amarelo (9), vermelho (8)
   - Buzzer no pino 7
2. Faça o upload do código para seu Arduino.
3. Alimente o Arduino e veja o sistema funcionando! ⚙️🔌📲

---

## 📦 Bibliotecas necessárias

Antes de compilar, instale estas bibliotecas na IDE do Arduino:

- `Wire.h` (já vem com a IDE)
- `LiquidCrystal_I2C.h`
- `DHT.h` (como a da Adafruit) 📚💾🔧

---

## 📌 Observações

- O sistema faz pausas entre as verificações para dar tempo de leitura e evitar ruídos nos sensores.
- É possível ajustar os **limites de luminosidade, temperatura e umidade** diretamente no código, conforme as necessidades do seu ambiente. 📝📉🧯

---

## ✨ Feito para o projeto da Vinheria Agnello 🍷🍇🏷️

---

## 📄 Código Utilizado

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int ldrPin = A0;
const int dhtPin = 5;
const int ledVerde = 10;
const int ledAmarelo = 9;
const int ledVermelho = 8;
const int buzzerPin = 7;

DHT dht(dhtPin, DHT22);

void setup() {
  lcd.begin(16, 2);
  lcd.backlight();

  dht.begin();

  pinMode(ledVerde, OUTPUT);
  pinMode(ledAmarelo, OUTPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  lcd.setCursor(0, 0);
  lcd.print("Vinheria Agnello");
  lcd.setCursor(0, 1);
  lcd.print("Monitoramento");
  delay(2000);
}

void loop() {
  float tempSum = 0, humSum = 0;
  int ldrSum = 0;
  const int numReadings = 5;

  for (int i = 0; i < numReadings; i++) {
    tempSum += dht.readTemperature();
    humSum += dht.readHumidity();
    ldrSum += analogRead(ldrPin);
    delay(200);
  }

  float temp = tempSum / numReadings;
  float hum = humSum / numReadings;
  int ldrValue = ldrSum / numReadings;

  lcd.clear();

  if (ldrValue < 300) {
    digitalWrite(ledVerde, HIGH);
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(ledVermelho, LOW);
    lcd.setCursor(0, 0);
    lcd.print("Ambiente escuro");
    digitalWrite(buzzerPin, LOW);
  } else if (ldrValue >= 300 && ldrValue < 700) {
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledAmarelo, HIGH);
    digitalWrite(ledVermelho, LOW);
    lcd.setCursor(0, 0);
    lcd.print("Ambiente meia luz");
    digitalWrite(buzzerPin, LOW);
  } else {
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(ledVermelho, HIGH);
    lcd.setCursor(0, 0);
    lcd.print("Ambiente claro");
    digitalWrite(buzzerPin, HIGH);
    delay(2000);
    digitalWrite(buzzerPin, LOW);
  }

  delay(2000);

  if (temp >= 10 && temp <= 15) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temperatura OK");
    lcd.setCursor(0, 1);
    lcd.print(temp);
    lcd.print(" C");
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(buzzerPin, LOW);
  } else {
    lcd.clear();
    lcd.setCursor(0, 0);
    if (temp < 10) {
      lcd.print("Temp. Baixa");
    } else {
      lcd.print("Temp. Alta");
    }
    lcd.setCursor(0, 1);
    lcd.print(temp);
    lcd.print(" C");
    digitalWrite(ledAmarelo, HIGH);
    digitalWrite(buzzerPin, HIGH);
    delay(2000);
    digitalWrite(buzzerPin, LOW);
  }

  delay(2000);

  if (hum >= 50 && hum <= 70) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Umidade OK");
    lcd.setCursor(0, 1);
    lcd.print(hum);
    lcd.print(" %");
    digitalWrite(ledVermelho, LOW);
    digitalWrite(buzzerPin, LOW);
  } else {
    lcd.clear();
    lcd.setCursor(0, 0);
    if (hum < 50) {
      lcd.print("Umidade Baixa");
    } else {
      lcd.print("Umidade Alta");
    }
    lcd.setCursor(0, 1);
    lcd.print(hum);
    lcd.print(" %");
    digitalWrite(ledVermelho, HIGH);
    digitalWrite(buzzerPin, HIGH);
    delay(2000);
    digitalWrite(buzzerPin, LOW);
  }

  delay(1000);
}
```

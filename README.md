# AlarmeEstiloACME

O código de arduino que eu escrevi não é proximo do mais eficiente, 
entretanto como eu não sou versado em C++ e as particuloaridades dele 
o resultado final funcionou, mas com alguns caviates.


#include <Wire.h>        //Biblioteca para manipulação do protocolo I2C
#include <DS3231.h>      //Biblioteca para manipulação do DS3231
#include <Servo.h>
#include <LiquidCrystal.h>

DS3231 clock;              //Criação do objeto do tipo DS3231
RTCDateTime dataehora;   //Criação do objeto do tipo RTCDateTime

Servo meuServo;

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

int portaServo = 9;
int portaButao = 8;
int portaButaoTeste = 7;
int potenciometroHora = A0;
int potenciometroMinuto = A1;


int horaDoAlarme = 6;
int minutoDoAlarme = 10;

void setup()
{
  Serial.begin(9600);     //Inicialização da comunicação serial
  clock.begin();            //Inicialização do RTC DS3231
  clock.setDateTime(__DATE__, __TIME__);   //Configurando valores iniciais 
                                         //do RTC DS3231
  clock.armAlarm1(false);
  clock.armAlarm2(false);
  clock.clearAlarm1();
  clock.clearAlarm2();

  clock.setAlarm2(0, 6, 10, DS3231_MATCH_H_M);

  meuServo.attach(portaServo);
  meuServo.write(65);

  pinMode(portaButao, INPUT);
  pinMode(portaButaoTeste, INPUT);

  lcd.begin(16,2);
  
}
void loop(){
  updateTime();
    
  if (clock.isAlarm2()){
    meuServo.write(180);
    delay(1000);
    meuServo.write(65);
  }
  if (digitalRead(portaButaoTeste)==HIGH){
    meuServo.write(180);
    delay(100);
    meuServo.write(65);
  }
  while (digitalRead(portaButao)==HIGH){
    clock.armAlarm2(false);
    clock.clearAlarm2();
    while (digitalRead(portaButao)==HIGH){
      horaDoAlarme = map(analogRead(potenciometroHora),0,1023,0,23);
      minutoDoAlarme = map(analogRead(potenciometroMinuto),0,1023,0,59);
      updateTime();
      while(digitalRead(portaButaoTeste)==HIGH){
      horaDoAlarme = map(analogRead(potenciometroHora),0,1023,0,23);
      minutoDoAlarme = map(analogRead(potenciometroMinuto),0,1023,0,59);
      clock.setDateTime(clock.getDateTime().year,clock.getDateTime().month,clock.getDateTime().day,horaDoAlarme,minutoDoAlarme,clock.getDateTime().second);
      updateTime();
        }
      }
    clock.setAlarm2(0, horaDoAlarme, minutoDoAlarme, DS3231_MATCH_H_M);
    }
}

void updateTime (){
  dataehora = clock.getDateTime(); 
  lcd.clear();
  lcd.setCursor(4, 0);
  lcd.print(dataehora.hour);    
  lcd.print(":");
  lcd.print(dataehora.minute); 
  lcd.print(":");
  lcd.print(dataehora.second);
  
  lcd.setCursor(2, 1);
  lcd.print("alarme:");
  lcd.print(horaDoAlarme);    
  lcd.print(":");
  lcd.print(minutoDoAlarme); 
  }

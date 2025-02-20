# Titre
## Semaine 1
## Semaine 2
## Semaine 3
## Semaine 4 : jeudi 20 f&vrier
- Test des 2 moteurs pour 
Une morceau de code pour controller bles 2 moteurs

```C
void setup(){
    //Pins
pinMode(9, OUTPUT);
pinMode(8, OUTPUT);
pinMode(7, OUTPUT);
pinMode(6, OUTPUT);
}
void loop(){

int speed1 = 70// gauche
int speed2 = 120;// droite

analogWrite(9,speed1);
analogWrite(6, speed2);
delay(2000);
analogWrite(9, speed1);
analogWrite(6, speed2);
delay(2000);
analogWrite(9, 0);
analogWrite(6, 0);
delay(10000);
}
```


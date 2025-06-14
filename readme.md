# Projet Rescue
## Séance 1 : jeudi 20 février

- Changement d'alimentation : le driver moteur est maintenant alimenté avec une alimentation de table et il transmet l'alimentation vers les 2 moteurs et aussi la carte arduino grâce au régulateur 5V intégré. La tension d'alimentation choisie est de 9V pour alimenter les moteurs.

- Après avoir effectué ce nouveau câblage, les 2 moteurs tournent et on peut aussi débrancher le câble de la carte arduino qui est assez court et limitait donc les déplacements du robot. Mais les 2 moteurs ne tournent pas à la même vitesse. La solution est d'augmenter dans le programme la vitesse du moteur qui tourne plus lentement jusqu'à ce que leurs vitesses s'équilibrent. Le robot peut donc maintenant se déplacer tout droit mais aussi faire des virages à droite et à gauche.

- Test des 2 moteurs :
<details>
  <summary>Morceau de code pour contrôler les 2 moteurs</summary>

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
```
</details>

Prochaine séance : 
Faire fonctionner le nouveau capteur et faire un câblage en utilisant les pins ENA et ENB du driver moteur.


## Séance 2 :

- Mise en place du capteur : le capteur fonctionne mais le robot ne réagit pas bien
- Le robot peut suivre la ligne en allant tout droit mais il réagit aléatoirement lorsqu'il doit tourner pour se remettre sur la ligne

<details>
  <summary>Programme</summary>
    
```C
#include <QTRSensors.h>
QTRSensors qtr;

// Définition des capteurs QTR-MD-04A

const uint8_t SensorCount = 4;
uint16_t sensorValues[SensorCount];

const int ENA = 9;
const int IN1 = 8;
const int IN2 = 7;
const int ENB = 10;
const int IN3 = 6;
const int IN4 = 5;

int speedMotor = 180;  // Vitesse moteur (0-255)

void setup()
{
  Serial.begin(9600);
  // Optional: wait for some input from the user, such as a button press.
  // Initialize the sensors.
  // In this example we have three sensors on pins A0 - A2.
  qtr.setTypeRC(); // or setTypeAnalog()
  qtr.setSensorPins((const uint8_t[]) {
    A1, A2, A3, A4
  }, 4);
  qtr.setEmitterPin(2);

  // Optional: change parameters like RC timeout, set an emitter control pin...
  // Then start the calibration phase and move the sensors over both reflectance
  // extremes they will encounter in your application. This calibration should
  // take about 5 seconds (250 iterations * 20 ms per iteration).
  //
  // Passing a QTRReadMode to calibrate() is optional; it should match the mode
  // you plan to use when reading the sensors.
  for (uint8_t i = 0; i < 250; i++)
  {
    qtr.calibrate();
    delay(20);
  }
  // Optional: signal that the calibration phase is now over and wait for
  // further input from the user, such as a button press.



  // Configuration des moteurs
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);


  int speedMotorGauche = 180;  // Vitesse moteur gauche (0-255)
  int speedMotorDroit = 180 + 80;  // Compensation de 80 unités



}

// Fonction pour lire la position de la ligne
int lirePositionLigne() {
  return qtr.readLineBlack(sensorValues);
}

// Fonctions de mouvement
void avancer() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerGauche() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerDroite() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void arreter() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}


void loop()
{

  uint16_t sensors[4];
  // Get calibrated sensor values returned in the sensors array, along with the
  // line position, which will range from 0 to 2000, with 1000 corresponding to
  // a position under the middle sensor.
  int16_t position = qtr.readLineBlack(sensors);
  Serial.println(position);
  // If all three sensors see very low reflectance, take some appropriate action
  // for this  situation.
  if ((sensors[0] > 750) && (sensors[1] > 750) && (sensors[2] > 750))
  {
    // Do something. Maybe this means we're at the edge of a course or about to
    // fall off a table, in which case we might want to stop moving, back up,
    // and turn around.
    return;
  }


  Serial.print("Position ligne: ");
  Serial.println(position);

  // Suivi de ligne en fonction de la position détectée
  if (position < 600) {
    tournerGauche();  // Ligne à gauche → tourner à gauche
  }
  else if (position > 2400) {
    tournerDroite();  // Ligne à droite → tourner à droite
  }
  else {
    avancer();  // Ligne au centre → avancer
  }

  delay(100);
}
```
</details>

- Prochaine séance :
  Modifier le programme pour que le robot puisse se réajuster sur la ligne lorsqu'il la quitte


## Séance 3 :

- Après modification du programme, le robot fonctionne par moment. Parfois il sait suivre une ligne droite et même des virages. Mais après avoir éteint et rallumé l'alimentation sans modifier le programme, le robot ne suis pas la ligne.
- De plus, après un test sur une ligne discontinue, le robot n'est pas capable de suivre la ligne tout droit.
- Voici le programme actuel :

<details>
  <summary>Programme</summary>
    
```C
#include <QTRSensors.h>
QTRSensors qtr;

// Définition des capteurs QTR-MD-04A

const uint8_t SensorCount = 4;
uint16_t sensorValues[SensorCount];

const int ENA = 9;
const int IN1 = 8;
const int IN2 = 7;
const int ENB = 10;
const int IN3 = 6;
const int IN4 = 5;

int speedMotor = 80;  // Vitesse moteur (0-255)

void setup()
{
  Serial.begin(9600);
  // Optional: wait for some input from the user, such as a button press.
  // Initialize the sensors.
  // In this example we have three sensors on pins A0 - A2.
  qtr.setTypeRC(); // or setTypeAnalog()
  qtr.setSensorPins((const uint8_t[]) {
    A1, A2, A3, A4
  }, 4);
  qtr.setEmitterPin(2);

  // Optional: change parameters like RC timeout, set an emitter control pin...
  // Then start the calibration phase and move the sensors over both reflectance
  // extremes they will encounter in your application. This calibration should
  // take about 5 seconds (250 iterations * 20 ms per iteration).
  //
  // Passing a QTRReadMode to calibrate() is optional; it should match the mode
  // you plan to use when reading the sensors.
  for (uint8_t i = 0; i < 250; i++)
  {
    qtr.calibrate();
    delay(20);
  }
  // Optional: signal that the calibration phase is now over and wait for
  // further input from the user, such as a button press.



  // Configuration des moteurs
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);


  int speedMotorGauche = 80;  // Vitesse moteur gauche (0-255)
  int speedMotorDroit = 80;  // Compensation de 80 unités
  analogWrite(ENA, speedMotor);
  analogWrite(ENB, speedMotor);


}

// Fonction pour lire la position de la ligne
int lirePositionLigne() {
  return qtr.readLineBlack(sensorValues);
}

// Fonctions de mouvement
void avancer() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerGauche() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerDroite() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void arreter() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}


void loop()
{

  uint16_t sensors[4];
  // Get calibrated sensor values returned in the sensors array, along with the
  // line position, which will range from 0 to 2000, with 1000 corresponding to
  // a position under the middle sensor.
  int16_t position = qtr.readLineBlack(sensors);
  Serial.println(position);
  // If all three sensors see very low reflectance, take some appropriate action
  // for this  situation.
  if ((sensors[0] > 750) && (sensors[1] > 750) && (sensors[2] > 750))
  {
    // Do something. Maybe this means we're at the edge of a course or about to
    // fall off a table, in which case we might want to stop moving, back up,
    // and turn around.
    return;
  }


  Serial.print("Position ligne: ");
  Serial.println(position);

  // Suivi de ligne en fonction de la position détectée
  if (position < 600) {
    tournerGauche();  // Ligne à droite → tourner à droite
  }
  else if (position > 2400) {
    tournerDroite();  // Ligne à gauche → tourner à gauche
  }
  else {
    avancer();  // Ligne au centre → avancer
  }

  delay(100);
}
```

</details>

- Prochaine séance : faire en sorte que le robot puisse toujours suivre la ligne (au moins la ligne droite), peut-être en changeant le placement du capteur, l'éclairage, le programme...

## BILAN 3 avril

- Après quelques modifications du programme, le robot est capable de suivre des lignes droites et des courbes mais pas des virages trop serrés.

  Voici le programme actuel :

<details>
  <summary>Programme</summary>
  
```C
#include <QTRSensors.h>
QTRSensors qtr;

// Définition des capteurs QTR-MD-04A

const uint8_t SensorCount = 4;
uint16_t sensorValues[SensorCount];

const int ENA = 9;
const int IN1 = 8;
const int IN2 = 7;
const int ENB = 10;
const int IN3 = 6;
const int IN4 = 5;

int speedMotor = 80;  // Vitesse moteur (0-255)

void setup()
{
  Serial.begin(9600);
  // Optional: wait for some input from the user, such as a button press.
  // Initialize the sensors.
  // In this example we have three sensors on pins A0 - A2.
  qtr.setTypeRC(); // or setTypeAnalog()
  qtr.setSensorPins((const uint8_t[]) {
    A1, A2, A3, A4
  }, 4);
  qtr.setEmitterPin(2);

  // Optional: change parameters like RC timeout, set an emitter control pin...
  // Then start the calibration phase and move the sensors over both reflectance
  // extremes they will encounter in your application. This calibration should
  // take about 5 seconds (250 iterations * 20 ms per iteration).
  //
  // Passing a QTRReadMode to calibrate() is optional; it should match the mode
  // you plan to use when reading the sensors.
  for (uint8_t i = 0; i < 250; i++)
  {
    qtr.calibrate();
    delay(20);
  }
  // Optional: signal that the calibration phase is now over and wait for
  // further input from the user, such as a button press.



  // Configuration des moteurs
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);


  int speedMotorGauche = 80;  // Vitesse moteur gauche (0-255)
  int speedMotorDroit = 80;  // Compensation de 80 unités
  analogWrite(ENA, speedMotor);
  analogWrite(ENB, speedMotor);


}

// Fonction pour lire la position de la ligne
int lirePositionLigne() {
  return qtr.readLineBlack(sensorValues);
}

// Fonctions de mouvement
void avancer() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerGauche() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerDroite() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void arreter() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}


void loop()
{

  uint16_t sensors[4];
  // Get calibrated sensor values returned in the sensors array, along with the
  // line position, which will range from 0 to 2000, with 1000 corresponding to
  // a position under the middle sensor.
  int16_t position = qtr.readLineBlack(sensors);
  Serial.println(position);
  // If all three sensors see very low reflectance, take some appropriate action
  // for this  situation.
  if ((sensors[0] > 750) && (sensors[1] > 750) && (sensors[2] > 750))
  {
    // Do something. Maybe this means we're at the edge of a course or about to
    // fall off a table, in which case we might want to stop moving, back up,
    // and turn around.
    return;
  }


  Serial.print("Position ligne: ");
  Serial.println(position);

  // Suivi de ligne en fonction de la position détectée
  if (position < 600) {
    tournerDroite();
      // Ligne à droite → tourner à droite
  }
  else if (position > 1500) {
    tournerGauche();  // Ligne à gauche → tourner à gauche
  }
  else {
    avancer();  // Ligne au centre → avancer
  }

  delay(100);
}
```

</details>

Prochaine séance : 
- Ajouter un nouveau capteur
- Utiliser une carte arduino mega pour ajouter le nouveau capteur
- -> modifications du programme et du câblage

## Bilan 15 mai

- Réorganisation des câbles du robot : tout est maintenant organisé, les câbles sont moins encombrants et ne se baladent plus partout. Les cartes sont désormais fixées sur le robot.
- Modifications à venir :  
modifications du programme : ajouter le mode perdu  
-> si le robot tourne à droite ou à gauche alors lancer le timer  
si timer > durée alors basculement vers le mode perdu (la durée sera définie après avoir effectué des tests)  
L'objectif est dans un premier temps de détecter le mode perdu (dans quels cas il s'active) car il doit s'activer que dans certains cas selon notre demande.  
Puis dans un second temps de définir les actions du mode perdu, par exemple : s'arrêter, tourner sur lui même.  
De plus, une fois la ligne retrouvée, le robot va devoir rebasculer dans le mode par défaut (suivi de ligne).

## Bilan 22 mai

- écriture du programme en machine à états
- tests et résolution des problèmes du programme
- le robot fonctionne avec le programme machine à états en ligne droite et courbe mais pas dans les virages en angle droit ou alors prend beaucoup de temps
- Vidéos : https://drive.google.com/drive/folders/1nuJETroWs8m9KIOfO9hq21NwZyQXgBga?usp=sharing
<details>
  <summary>Programme</summary>
  
```C
#include <QTRSensors.h>
QTRSensors qtr;

// Définition des capteurs QTR-MD-04A

const uint8_t SensorCount = 4;
uint16_t sensorValues[SensorCount];

const int ENA = 9;
const int IN1 = 8;
const int IN2 = 7;
const int ENB = 10;
const int IN3 = 6;
const int IN4 = 5;

int speedMotor = 80;  // Vitesse moteur (0-255)

// définition des états

int centre = 0;
int LigneDroite = 0;
int LigneGauche = 0;

void setup() {

  pinMode(13, OUTPUT);

  Serial.begin(9600);
  // Optional: wait for some input from the user, such as a button press.
  // Initialize the sensors.
  // In this example we have three sensors on pins A0 - A2.
  qtr.setTypeRC();  // or setTypeAnalog()
  qtr.setSensorPins((const uint8_t[]) {
    A1, A2, A3, A4
  },
  4);
  qtr.setEmitterPin(2);

  // Optional: change parameters like RC timeout, set an emitter control pin...
  // Then start the calibration phase and move the sensors over both reflectance
  // extremes they will encounter in your application. This calibration should
  // take about 5 seconds (250 iterations * 20 ms per iteration).
  //
  // Passing a QTRReadMode to calibrate() is optional; it should match the mode
  // you plan to use when reading the sensors.
  for (uint8_t i = 0; i < 250; i++) {
    qtr.calibrate();
    delay(20);
  }
  // Optional: signal that the calibration phase is now over and wait for
  // further input from the user, such as a button press.



  // Configuration des moteurs
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);


  int speedMotorGauche = 80;  // Vitesse moteur gauche (0-255)
  int speedMotorDroit = 80;   // Compensation de 80 unités
  analogWrite(ENA, speedMotor);
  analogWrite(ENB, speedMotor);
}

// Fonction pour lire la position de la ligne
int lirePositionLigne() {
  return qtr.readLineBlack(sensorValues);
}

// Fonctions de mouvement
void avancer() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerGauche() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void tournerDroite() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void arreter() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}


void loop() {

  digitalWrite(13, LOW);

  uint16_t sensors[4];
  // Get calibrated sensor values returned in the sensors array, along with the
  // line position, which will range from 0 to 2000, with 1000 corresponding to
  // a position under the middle sensor.
  int16_t position = qtr.readLineBlack(sensors);
  //Serial.println(position);
  // If all three sensors see very low reflectance, take some appropriate action
  // for this  situation.
  if ((sensors[0] > 750) && (sensors[1] > 750) && (sensors[2] > 750)) {
    // Do something. Maybe this means we're at the edge of a course or about to
    // fall off a table, in which case we might want to stop moving, back up,
    // and turn around.
    return;
  }


  Serial.print("Position ligne: ");
  Serial.println(position);


  // Définition des changements d'état en fonction des conditions
  if (position < 600) {
    LigneDroite = 1;  // Changement d'état
  } else {
    LigneDroite = 0;
  }

  if (position > 1500) {
    LigneGauche = 1;  // Changement d'état
  } else {
    LigneGauche = 0;
  }

  if (position > 600 && position < 1500) {
    centre = 1;  // Changement d'état
  } else {
    centre = 0;
  }

  // Définition des actions en fonction des états

  if (LigneDroite == 1) {
    tournerDroite();  // Ligne à droite → tourner à droite
    LigneGauche = 0;
    centre = 0;
    Serial.println("TourneDroite");


  }

  if (LigneGauche == 1) {
    tournerGauche();  // Ligne à gauche → tourner à gauche
    LigneDroite = 0;
    centre = 0;
    Serial.println("TourneGauche");

  }

  if (centre == 1) {
    digitalWrite(13, HIGH);
    avancer();  // Ligne au centre → avancer
    LigneGauche = 0;
    LigneDroite = 0;
    Serial.println("ToutDroit");

  }



  delay(100);
}

```

</details>


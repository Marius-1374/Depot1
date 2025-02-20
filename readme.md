# Projet Rescue
## Séance 1 : jeudi 20 février

- Changement d'alimentation : le driver moteur est maintenant alimenté avec une alimentation de table et il transmet l'alimentation vers les 2 moteurs et aussi la carte arduino grâce au régulateur 5V intégré. La tension d'alimentation choisie est de 9V pour alimenter les moteurs.

- Après avoir effectué ce nouveau câblage, les 2 moteurs tournent et on peut aussi débrancher le câble de la carte arduino qui est assez court et limitait donc les déplacements du robot. Mais les 2 moteurs ne tournent pas à la même vitesse. La solution est d'augmenter dans le programme la vitesse du moteur qui tourne plus lentement jusqu'à ce que leurs vitesses s'équilibrent. Le robot peut donc maintenant se déplacer tout droit mais aussi faire des virages à droite et à gauche.

- Test des 2 moteurs :
Morceau de code pour controller les 2 moteurs :

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

Prochaine séance : 
Faire fonctionner le nouveau capteur et faire un câblage en utilisant les pins ENA et ENB du driver moteur.

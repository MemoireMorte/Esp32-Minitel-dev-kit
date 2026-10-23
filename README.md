# ESP32 Minitel dev kit

Carte à base d'**ESP32-WROOM-32** qui se branche sur la prise péri-informatique (DIN 5 broches) d'un Minitel et l'utilise comme terminal d'entrée/sortie de l'ESP32 : le clavier et l'écran du Minitel deviennent la console d'un microcontrôleur disposant du Wifi et du Bluetooth.

La carte peut être alimentée directement par le Minitel ou par une alimentation externe (5-9V).
La carte v2 peut-etre alimenté en usb.

Elle est compatible, côté logiciel, avec les exemples du projet [iodeo/Minitel-ESP32](https://github.com/iodeo/Minitel-ESP32) (même UART, mêmes GPIO 16/17).

## Code

- [MemoireMorte/EspTel](https://github.com/MemoireMorte/EspTel) - client SSH - WS - Telnet
- [MemoireMorte/minitracker](https://github.com/MemoireMorte/minitracker) - séquenceur 6 pistes
- [MemoireMorte/3615-Home-Assistant](https://github.com/MemoireMorte/3615-Home-Assistant) - dashboard Home Assistant

## Vue d'ensemble

```
                 +--------------------------------------+
 Minitel DIN 5   |  PowerSource     AMS1117-3.3         |
 (J5) 1 RX  <----|-- Q1 2N2222 <---- GPIO17 (TX2)       |
      2 GND -----|-- GND                                |
      3 TX  -----|-----------------> GPIO16 (RX2)       |
      4 PT   n.c.|                    ESP32-WROOM-32    |
      5 +V  -----|-- cavalier                           |
                 |                   GPIO1/3 (UART0) ---|--> J3 programmation (FTDI) 1:GND
 micro-USB ------|-- cavalier        GPIO0  --- PROG    |                            2:NC
 (alim. seule)   |                   EN     --- RESET   |                            3:3.3V
                 +--------------------------------------+                            4:RX
                                                                                     5:TX
```

## Pinout

### GPIO de l'ESP32 utilisés

| GPIO | Fonction | Détail |
|---|---|---|
| GPIO16 (RX2) | Réception depuis le Minitel | Relié à la broche 3 de la DIN (TX Minitel), tirage 56 kΩ au 3,3 V |
| GPIO17 (TX2) | Émission vers le Minitel | Via Q1 (2N2222 en base commune, sortie collecteur ouvert) vers la broche 1 de la DIN (RX Minitel) |
| GPIO1 (TXD0) | UART0 TX | Connecteur de programmation J3 |
| GPIO3 (RXD0) | UART0 RX | Connecteur de programmation J3 |
| GPIO0 | Bouton **PROG** | Tirage 10 kΩ au 3,3 V + 100 nF, à la masse quand le bouton est appuyé |
| EN | Bouton **RESET** | Tirage 10 kΩ au 3,3 V + 100 nF |
| GPIO13 | LED utilisateur (v2 seulement) | Active à l'état haut, résistance 220 Ω |

La seconde LED est le témoin d'alimentation 3,3 V. Les autres GPIO du module ne sont pas câblés sur la v1, et sont reliés aux pins de chaque coté sur la v2.

### Connecteur Minitel (J5, 5 broches au bord de la carte)

La numérotation du connecteur suit celle de la prise DIN du Minitel : broche *n* de la carte = broche *n* de la DIN.

| Broche | Signal DIN Minitel | Sur la carte |
|---|---|---|
| 1 | RX (entrée du Minitel) | Collecteur de Q1, piloté par GPIO17 |
| 2 | Masse | GND |
| 3 | TX (sortie du Minitel) | GPIO16 |
| 4 | PT (périphérique prêt) | Non connecté |
| 5 | Alimentation (~8,5 V, 1 A max) | Cavalier `PowerSource`, côté Minitel |

Rappel : sur une DIN 5 broches à 180°, les broches ne sont pas dans l'ordre numérique. Vue de face, l'ordre physique est **1 – 4 – 2 – 5 – 3** (la 2 au centre). Vérifier au multimètre avant de souder le cordon.

### Connecteur de programmation (J3, 5 broches)

Sérigraphie : `TX RX 3.3 NC GND`. Les noms sont donnés du point de vue de l'ESP32.

| Broche J3 | Sérigraphie | Signal ESP32 |
|---|---|---|
| 5 | TX | GPIO1 / TXD0 |
| 4 | RX | GPIO3 / RXD0 |
| 3 | 3.3 | Rail +3,3 V |
| 2 | NC | Non connecté |
| 1 | GND | Masse (pastille carrée) |

### Switch d'alimentation (`PowerSource`, 3 broches)

## Branchement au Minitel

1. Réaliser un cordon DIN 5 broches mâle vers le connecteur J5, fil à fil (1→1, 2→2, 3→3, 5→5 ; la 4 est inutile) ou utilisez un cable MIDI.
2. Placer le switch `Power Source` côté **Minitel**.
3. Brancher la DIN à l'arrière du Minitel, puis allumer le Minitel. La LED d'alimentation de la carte doit s'allumer.

Points d'attention :

- **Alimentation par le Minitel** : la broche 5 fournit environ 8,5 V. Le régulateur dissipe donc la différence (≈ 5 V × le courant consommé) et chauffe lorsque le Wifi est actif. C'est normal, mais éviter de l'enfermer sans aération.
- Tous les Minitel ne fournissent pas d'alimentation sur la broche 5 (certains Minitel 1 anciens notamment). Dans ce cas, alimenter par alimentation externe, switch côté **Ext**.

## Programmation avec un module FTDI

La carte V1 n'a ni convertisseur USB-série ni circuit d'auto-reset : on programme par l'UART0 avec un adaptateur externe (FTDI FT232, CP2102, CH340…) et on passe en mode bootloader à la main avec les boutons.
La carte V2 dispose d'une UART CH340 reliée directement au port USB et ne nécessite pas de programmeur externe.

### Câblage

**L'adaptateur doit être réglé en 3,3 V** (cavalier ou interrupteur sur le module). Des niveaux 5 V sur RX endommagent l'ESP32.

| J3 (carte) | Module FTDI |
|---|---|
| TX | RX |
| RX | TX |
| GND | GND |
| 3.3 | Ne pas brancher (voir ci-dessous) |

Alimenter la carte par le micro-USB ou par le Minitel pendant la programmation. La sortie 3,3 V d'un FT232 ne fournit qu'environ 50 mA, ce qui est insuffisant pour l'ESP32 (pointes à plusieurs centaines de mA). La broche `3.3` de J3 ne doit être utilisée que si l'adaptateur possède un vrai régulateur 3,3 V, et dans ce cas sans autre source d'alimentation branchée.

Il n'est pas nécessaire de débrancher le Minitel pour programmer : il est sur l'UART2, indépendant de l'UART0.

### Passage en mode bootloader

1. Maintenir **PROG** appuyé.
2. Appuyer brièvement sur **RESET**.
3. Relâcher **PROG**.

L'ESP32 attend alors le téléversement. Une fois celui-ci terminé, appuyer sur **RESET** pour lancer le programme.

### Arduino IDE

1. Installer le support des cartes ESP32 (gestionnaire de cartes, paquet *esp32* d'Espressif).
2. Choisir la carte **ESP32 Dev Module** et le port COM de l'adaptateur FTDI.
3. Installer la bibliothèque [Minitel1B_Hard](https://github.com/eserandour/Minitel1B_Hard) d'Éric Sérandour.
4. Mettre la carte en mode bootloader (ci-dessus), puis téléverser.

Si le téléversement échoue à la connexion (`Connecting.....`), refaire la séquence PROG/RESET, ou baisser la vitesse de téléversement à 115200 bauds.

Le moniteur série (115200 bauds en général) reste utilisable sur le FTDI pour le débogage, pendant que le Minitel sert de terminal.

## Utilisation logicielle

Le Minitel est sur l'**UART2** de l'ESP32 (`Serial2`, RX = GPIO16, TX = GPIO17), en **1200 bauds, 7 bits, parité paire, 1 stop (7E1)** à l'allumage.

Exemple minimal :

```cpp
#include <Minitel1B_Hard.h>

#define MINITEL_PORT Serial2   // RX = GPIO16, TX = GPIO17
#define LED_PIN      13

Minitel minitel(MINITEL_PORT);

void setup() {
  Serial.begin(115200);                        // debug sur J3 (FTDI)
  pinMode(LED_PIN, OUTPUT);

  minitel.changeSpeed(minitel.searchSpeed());  // détecte la vitesse courante du Minitel
  minitel.newScreen();
  minitel.println("Bonjour depuis l'ESP32 !");
  digitalWrite(LED_PIN, HIGH);
}

void loop() {
  unsigned long touche = minitel.getKeyCode();
  if (touche != 0) {
    Serial.printf("Touche : 0x%lX\n", touche);
  }
}
```

### Avec les exemples iodeo

Les croquis du dossier `arduino/` de [iodeo/Minitel-ESP32](https://github.com/iodeo/Minitel-ESP32) fonctionnent sur cette carte tant que le port Minitel est `Serial2` sur les GPIO 16/17. Deux différences matérielles à garder en tête par rapport à la carte iodeo :

- pas d'USB-série intégré : téléversement par FTDI et séquence PROG/RESET manuelle ;
- la LED utilisateur est sur GPIO13.

### Vitesse de liaison

Le Minitel démarre à 1200 bauds. Les Minitel 1B et suivants acceptent 4800 bauds, les Minitel 2 et suivants 9600 bauds ; `minitel.changeSpeed(4800)` bascule le Minitel et l'UART. Le réglage est perdu à l'extinction du Minitel, d'où l'appel à `searchSpeed()` au démarrage.


## Dépannage

| Symptôme | Piste |
|---|---|
| LED d'alimentation éteinte | Position du switch `PowerSource` ; présence de tension sur la broche 5 de la DIN |
| Téléversement impossible | Séquence PROG/RESET, TX/RX à croiser, masse commune, adaptateur en 3,3 V |
| Caractères incohérents sur le Minitel | Vitesse ou format (7E1) incorrect ; appeler `searchSpeed()` au démarrage |
| Rien ne s'affiche, mais le clavier est reçu (ou l'inverse) | Broches 1 et 3 de la DIN inversées dans le cordon |
| Redémarrages à l'activation du Wifi | Alimentation insuffisante (FTDI seul ?) |

## Crédits

- [iodeo/Minitel-ESP32](https://github.com/iodeo/Minitel-ESP32) — projet d'origine et exemples logiciels
- [eserandour/Minitel1B_Hard](https://github.com/eserandour/Minitel1B_Hard) — bibliothèque Minitel pour Arduino

# Processadors_Digitals_Practica_6
# Practica 6

En aquesta pràctica treballarem un altre tipus de bus; el SPI. Aquest bus és de tipus “master-slave” que resulta bastant útil per a connectar perifèrics a un processador. En aquesta pràctica treballarem amb un lector de targetes SD i RFID.

## Exercici 1

En quest primer exercici treballarem amb un lector de targetes SD en el que llegirem i escriurem els continguts d’un arxiu .txt emmagatzemat dins la targeta SD.

### Funció `setup()`

Aquesta funció serà la que contindrà tot el codi per a aquest exercissi. Aqui utilitzarem algunes funcions de les llibreries `<SPI.h>` i `<SD.h>` que ens permetràn realitzar la pràctica més fàcilment.
Començarem comprobant el funcionament correcte de la targeta SD; ja que si aquesta (per exemple) no estigués ben connectada, no tindria sentit seguir amb el programa. Això ho aconseguim amb la funció `SD.begin( )` ; indicant dins el parèntesi el pin ****CS**** al que hem connectat el lector de targetes SD. En el nostre cas, aquest pin és el ************GPIO 5************; per tant escriurem la funció `SD.begin(5)` .
Un cop comprovada la connexió de la SD, obrirem un arxiu de text emmagatzemat dins aquesta i llegirem els continguts d’aquest arxiu treient-los per pantalla.

```cpp
File myFile;

void setup()
{
	Serial.begin(115200);
	Serial.print("Iniciando SD ...");
	if (!SD.begin(5)) { //indicamos el pin de CS
	  Serial.println("No se pudo inicializar");
	  return;
	}
	Serial.println("Inicializacion exitosa");
	myFile = SD.open("/archivo.txt");//abrimos el archivo
	if (myFile) {
	  Serial.println("/archivo.txt");
	  while (myFile.available()) {
	    Serial.write(myFile.read());
  }
  myFile.close(); //cerramos el archivo
	} else {
	  Serial.println("Error al abrir el archivo");
	}
}
```

També vam probar un altre codi més complert amb funcions per escriure, llegir arxius de la targeta SD, crear i eliminar arxius, etc. El codi amb aquestes funcions es troba a l’arxiu ****************************Practica_6.1.1****************************.

## Exercici 2

Aquest exercici consistirà en connectar un lector de RFID a la placa ESP32 i crear un programa que llegeixi el codi de cualsevol targeta RFID que apropem al sensor i ens mostri el seu codi per pantalla.

### Funció `setup()`

En aquesta funció inicialitzarem el bus SPI i el sensor RFID utilitzant funcions de les llibreries `<SPI.h>` i `<MFRC522.h>` .

```cpp
#define RST_PIN 27 //Pin 9 para el reset del RC522
#define SS_PIN 5 //Pin 10 para el SS (SDA) del RC522
MFRC522 mfrc522(SS_PIN, RST_PIN); //Creamos el objeto para el RC522

void setup() {
  Serial.begin(115200); //Iniciamos la comunicación serial
  SPI.begin(); //Iniciamos el Bus SPI
  mfrc522.PCD_Init(); // Iniciamos el MFRC522
  Serial.println("Lectura del UID");
}
```

Previament a la funció `setup()` definirem els pins per al reset (********RST********) i per al ****CS**** (que en aquest exemple anomenem SS).

### Funció `loop()`

Aquesta funció ens detectarà la presència (o no) d’una targeta RFID propera al sensor i treurà per pantalla el codi de la targeta.

```cpp
void loop() {
  // Revisamos si hay nuevas tarjetas presentes
  if ( mfrc522.PICC_IsNewCardPresent())
  {
    //Seleccionamos una tarjeta
    if ( mfrc522.PICC_ReadCardSerial())
    {
      // Enviamos serialemente su UID
      Serial.print("Card UID:");
      for (byte i = 0; i < mfrc522.uid.size; i++) {
        Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
        Serial.print(mfrc522.uid.uidByte[i], HEX);
      }
    Serial.println();
    // Terminamos la lectura de la tarjeta actual
    mfrc522.PICC_HaltA();
    }
  }
}
```

Els dos `Serial.print()` dins del `for()` son els que converteixen a hexadecimal el codi de la targeta RFID i el truen per pantalla.
Posant l’escriptura del codi dins el primer `if()` ens permet detectar una targeta un sol cop quan l’apropem al sensor; evitant detectar-la com diverses targetes mentre questa estigui a prop del sensor. Fins que no allunyem del sensor la targeta, el programa no fa cap lectura nova.

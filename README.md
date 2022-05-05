# Sw4IoT
Trabajo en grupo de Software para Internet de las Cosas, Máster en Ingeniería Informática UC3M.

# Introducción
Este repositorio Git es material complementario del trabajo en grupo de la asignatura Software para Internet de las Cosas (Sw4IoT) del Máster en Ingeniería Informática de la Universidad Carlos III de Madrid. En él, se detallan principalmente aspectos de *hardware* y del *software* analizado, junto con los estudios y modificaciones realizados.

El trabajo consiste en analizar la arquitectura IoT del [sensor de calidad de aire DIY de AirGradient](https://www.airgradient.com/diy/), desde el *hardware* hasta como mostrar los datos obtenidos en una interfaz de usuario.

# Lista de la compra
*Do it Yourself* implica también *Buy it Yourself*, sin las piezas por separado dificilmente se podrá contruir el sensor. AirGradient proporciona gratuitamente 3 PCBs de su sensor (pagando gastos de envío) para quien lo solicite en la página web. Es un precio muy económico y la mejor forma de obtenerlos, salvo que te lo dé tu profesor.

En cuanto a los sensores, los enlaces proporcionados por AirGradient son los más baratos que hay en AliExpress sumando gastos de envío, esto puede variar en función de ofertas y cupones, pero mi consejo es no ahorrarse unos pocos euros por comprar en un vendedor de dudosa reputación. Si tienes reparo a comprar por AliExpress, va a ser muy complicado encontrar los componentes que necesitas y si lo haces será con un sobreprecio bastante elevado.

A mayores de dichos componentes vas a necesitar:
*    Un soldador.
*    Estaño.
*    *Flux*.
*    Tiras de pines macho y hembra, si bien algunas de las piezas que se compran por AliExpress vienen con ellos, algunas no lo traen, así que es necesario comprar más por separado, son baratas y también nos sirven de repuesto en caso de desgracia. Recomendable aprovechar el pedido en AliExpress para pedirlas [por ahí](https://es.aliexpress.com/item/4000873858801.html?spm=a2g0o.productlist.0.0.4f8d6f71De7uCH&algo_pvid=b76c128a-a59a-4447-81d1-1932ac6c2e54&algo_exp_id=b76c128a-a59a-4447-81d1-1932ac6c2e54-7&pdp_ext_f=%7B%22sku_id%22%3A%2210000010058190554%22%7D&pdp_pi=-1%3B2.24%3B-1%3B-1%40salePrice%3BEUR%3Bsearch-mainSearch).

# Modificaciones planteadas sobre este *hardware*
AirGradient propone y ha diseñado su PCB para ser usado con una placa Wemos D1 mini, basada en el ESP8266. Es un microcontrolador muy solvente y que para la fucionalidad planteada por la propia empresa es suficiente. Sin embargo, si buscamos expandir las posibilidades de nuestro equipo es necesario utilizar un microcontrolador más potente, algunos de estas ampliaciones pueden ser:
*    Emplear cifrado extremo a extremo. El ESP8266 tiene potencia limitada, si se realizan algunos cálculos localmente que consuman CPU sumado a sus [limitadas capacidades criptográficas](https://www.reddit.com/r/esp8266/comments/5n6gqr/whats_the_current_state_of_ssltls_on_the_esp8266/) el resultado no es el deseado.
*    Ampliar la funcionalidad del dispositivos. Si bien es cierto que el dispositivo cumple a la perfección su labor, ser un sensor de calidad del aire, eso no significa que no pueda ser ampliado. Mucha gente es reacia al IoT porque "son más trastos en casa", es por ello que sería interesante explorar integrar funcionalidad de dispositivos que suelen estas en las casa de la gente: un reloj, una estación metereológica... De esta manera el dispositivo ya no sería "un trasto más", sino que quedaría integrado dentro de algo que ya se tenía previamente.

Todos estas ampliaciones requieren de un microcontrolador más potente, el elegido en este caso fue el ESP32-WROOM-32D en su forma de [kit de desarrollo](https://es.aliexpress.com/item/32959541446.html?spm=a2g0o.productlist.0.0.399e74dfsqCDZ2&algo_pvid=8b461c01-9148-431b-b46c-cd6da843b5f3&algo_exp_id=8b461c01-9148-431b-b46c-cd6da843b5f3-3&pdp_ext_f=%7B%22sku_id%22%3A%2267085222105%22%7D&pdp_pi=-1%3B4.36%3B-1%3B-1%40salePrice%3BEUR%3Bsearch-mainSearch), es económico, potente como para implementar lo anteriormente visto y, sobre todo, parecido al ESP8266.

Sin embargo, el PCB diseñado por AirGradient no es compatible, se pueden emplear *jumper wires* para conectar un ESP32 pero el resultado no será tan limpio como uno desearía ni lo más seguro o práctico. Es por ello que se ha modificado el esquemático para que cualquiera pueda encargar un PCB adaptado al ESP32.

<img title="PCB ESP32 mod" alt="PCB DIY de AirGradient modificado para el ESP32" src="https://raw.githubusercontent.com/Juanbanpar/Sw4IoT/main/PCBv2_mod.png">

Como se puede apreciar se ha modificado sobre la segunda versión del PCB, AirGradient está [trabajando en una revisión](https://forum.airgradient.com/t/new-airgradient-diy-kit-version-3-feedback-and-discussion/146) con cambios significativos que pueden facilitar la adaptación de un ESP32 sin modificar el factor de forma o emplear incluso un ESP32 mini. En este trabajo en grupo hemos modificado y trabajado sobre la versión v2 al ser la más reciente en su momento, a futuro es posible que continueos el trabajo con el nuevo PCB en función de nuestro interés personal.

# Modificaciones en el *software*

## Cifrado con TLS


## MQTT
Un protocolo alternativo a HTTP, más orientado a IoT, ideal para este tipo de dispositivos es [MQTT](https://mqtt.org/) este ofrece una serie de ventajas a la hora de contar con múltiples dispositivos gracias a su modelo publicador/subscriptor.
Para implementarlo se puede emplear la [librería de Arduino](https://docs.arduino.cc/tutorials/uno-wifi-rev2/uno-wifi-r2-mqtt-device-to-device), la modificación del código sería substituir las líneas dedicadas a la conexión HTTP por:

'''
const char broker[] = "test.mosquitto.org";
int port = 1883;
const char topic[] = "air";

...

void setup() {
  if (!mqttClient.connect(broker, port)) {
      Serial.print("MQTT connection failed! Error code = ");
      Serial.println(mqttClient.connectError());
      while (1);
  }
}

...

Serial.print("Sending message to topic: ");
Serial.println(topic);
Serial.println(Rvalue);
'''

Si se quisiera usar cifrado con tokens JWT no sería recomendable emplear código Ardunio debido a la limitada disponibilidad de [liberías y documentación](https://github.com/chrismoorhouse/ArduinoJWT). De igual manera, el ESP8266 no sería suficientemente potente para ejecutarlo, sería necesario recurrir al ESP32.

## Portar el código a MicroPython
Portar el código a MicroPython sería interesante pues permitiría ampliar la funcionalidad de forma más simple y empleando librerías mucho más potentes de alto nivel. También sería más fácil usar distintos microcontroladores sin apenas modificar el *software* del dispositivo, permitiendo mantenerse al día con los grances avances que se realizan en este ámbito.

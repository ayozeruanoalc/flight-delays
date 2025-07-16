# FlightDelays

[![My Skills](https://go-skill-icons.vercel.app/api/icons?i=java,idea,maven,junit,python,vscode,anaconda,scikitlearn,pandas,api,sqlite,uml,github)](https://go-skill-icons.vercel.app/api/)

## 📚 Tabla de Contenidos
- [📝 Descripción del proyecto y propuesta de valor](#-descripción-del-proyecto-y-propuesta-de-valor)
- [🧩 Tecnologías utilizadas](#-tecnologías-utilizadas)
- [🎯 Justificación de APIs y estructura del Datamart](#-justificación-de-la-elección-de-apis-y-estructura-del-datamart)
- [⚙️ Configuración](#%EF%B8%8F-configuraci%C3%B3n)
- [Modos de ejecución](#tutorial-de-ejecución-con-ejemplos)
- [Uso de la UI](#tutorial-de-uso-de-la-ui)
- [Arquitectura](#arquitecturas-del-sistema-y-aplicación)
- [Principios y patrones de diseño](#principios-y-patrones-de-diseño-aplicados-en-cada-módulo)


## 📝 Descripción del proyecto y propuesta de valor

**FlightDelays** es una aplicación desarrollada en **Java** que permite registrar, procesar y correlacionar datos sobre **retrasos de vuelos** con las **condiciones meteorológicas** asociadas a los aeropuertos de origen y destino.

El sistema se basa principalmente en **ActiveMQ** como núcleo de su arquitectura, utilizando esta plataforma de mensajería para desacoplar procesos y **gestionar eventos de forma asíncrona**.
Adicionalmente, cuenta con un mecanismo opcional de **persistencia de datos** mediante **SQLite**, que puede ser utilizado según las necesidades del entorno o configuración.

El valor que se aporta específicamente al usuario se trata de una **UI**, a la que es posible hacerle consultas sobre el rendimiento de determinados **predictores**, que tomen las **condiciones climáticas de aeropuertos** e intenten **explicar los retrasos de vuelos** mediante las mismas.

De esta manera, si algún modelo alcanzase un **valor de error lo suficientemente pequeño** al haberse entrenado con una **cantidad de registros considerable**, querrá decir que sería capaz de hacer **predicciones sobre retrasos con un grado de precisión notable**. El usuario será capaz de:
- Elegir el aeropuerto.
- Si se considera de **llegada o salida** (tal vez el clima afecte más en el retraso de un aterrizaje que en el de un despegue)
- Escoger **cuál de los modelos disponibles** se entrenará.

## 🧩 Tecnologías utilizadas

| Categoría              | Tecnologías                                                                 |
|------------------------|------------------------------------------------------------------------------|
| Lenguajes              | Java 21, Python 3.11.9                                                       |
| IDEs y entornos        | IntelliJ IDEA, VSCode, Anaconda                                              |
| Librerías de análisis  | Pandas, Scikit-learn                                                         |
| Testing                | JUnit                                                                        |
| Base de datos          | SQLite                                                                       |
| APIs externas          | AviationStack, OpenWeatherMap                                                |
| Arquitectura           | Apache ActiveMQ, arquitectura basada en eventos                             |
| Diseño/modelado | UML                                                                          |
| Control de versiones   | GitHub  

## 🎯 Justificación de la elección de APIs y estructura del Datamart 

### 🌐 APIs seleccionadas

**✈️ AviationStack API** → https://aviationstack.com/ 

Elegida por su **amplio alcance global** y su capacidad para proporcionar información detallada sobre:
- Vuelos
- Aeropuertos
- Aerolíneas
- Estados de vuelos

Esto la convierte en una fuente fundamental para analizar retrasos y correlacionarlos con condiciones externas.

**🌦️ OpenWeatherMap History API** → https://openweathermap.org/history

Integrada para acceder a **datos meteorológicos históricos**, permitiendo:
- Obtener condiciones climáticas precisas en ubicaciones específicas
- Correlacionar efectivamente **clima y retrasos** de vuelos

---

### 🗃️ Estructura del Datamart

La arquitectura del Datamart se organiza en **tres secciones principales**, cada una con una función y propósito claro:

**1️⃣ Particiones de eventos en tiempo real**

- **Formato**: Archivos `.csv`
- **Cantidad**: 2 archivos (uno por tópico: `Flights` y `Weather`)
- **Contenido**:
    - Campo de **marca temporal** para gestionar asincronía
    - Campo con el **evento crudo en formato JSON** proveniente del broker
- **Objetivo**: Almacenar información entrante en tiempo real para su posterior procesamiento

**2️⃣ Partición limpia (matching aplicado)**

- **Formato**: Archivo `.csv`
- **Contenido**:
    - Eventos de vuelos y clima **emparejados por cercanía temporal**
    - Recogidos en tiempo real o provenientes de históricos
- **Objetivo**: Dataset limpio y estructurado para análisis en Python

**3️⃣ Partición procesada**

- **Formato**: Archivo `.csv`
- **Contenido**:
    - Resultado del análisis realizado sobre la partición limpia
    - Información procesada y lista para su consulta mediante la UI
- **Objetivo**: Brindar datos útiles al usuario final

---

### 🧠 Ventajas del diseño del Datamart

#### ✅ Modularidad
Cada etapa del flujo (captura, transformación, análisis) está separada, facilitando:
- Mantenimiento
- Trazabilidad de errores

#### ✅ Escalabilidad
Componentes como el matching pueden evolucionar sin impactar el resto del sistema.

#### ✅ Flexibilidad
Permite sustituir herramientas (por ejemplo, cambiar Python por otra tecnología de análisis) sin rediseñar el sistema completo.

#### ✅ Robustez
El uso de archivos intermedios `.csv` como **checkpoints persistentes** mejora:
- Recuperación ante fallos
- Reprocesamiento ante errores inesperados

## ⚙️ Configuración

### 1. Requisitos Previos

- **Instalar ActiveMQ** en tu equipo.
- Tener instalado **Python** (versión **3.11.9** o superior). Asegúrate de que esté definido como **variable de entorno** del sistema.
- **Maven** debe poder ejecutarse en IntelliJ:
  - Si da error, **reinstalar Maven**, agregarlo a las variables del sistema y ejecutar en la terminal de IntelliJ el comando:
    
    ```bash
    mvn clean install
    ```

---

### 2. Clonar el Proyecto

- Clonar el repositorio en IntelliJ usando la opción "**Get from Version Control**" e ingresando el enlace del repositorio en **Repository URL**.

---

### 3. Preparar los Módulos

#### 📦 AviationStackFeeder

- Ir al archivo `main` del módulo.
- Proporcionar los siguientes **argumentos** (cada uno en una línea separada), en este orden:

    - Ruta **absoluta** de la base de datos.
    - URL de conexión TCP del broker de ActiveMQ.  
      _Ejemplo:_ `tcp://localhost:12345`
    - Cuatro códigos **IATA** de aeropuertos (uno por línea).<br>
      _Ejemplos:_ `MAD` `AMS` `JFK` `ZRH`
    - Cantidad indefinida de **apiKeys de AviationStack** (una por línea).

#### 📦 Event-Store-Builder

- Ir al archivo `main` del módulo.
- Argumentos esperados (uno por línea), en este orden:
    - URL de conexión TCP del broker de ActiveMQ.
    - Tópicos del broker (deben ser **obligatoriamente**):
    
      ```
      Flights
      Weather
      ```

#### 📦 Flight-Delay-Estimator

- Ir al archivo `main` del módulo.
- Argumentos esperados en orden (uno por línea):
    - Ruta **relativa** del histórico de vuelos, debe ser:  
     `eventstore/Flights/AviationStackFeeder`
    - Ruta **relativa** del histórico de climas, debe ser:  
     `eventstore/Weather/OpenWeatherMapFeeder`
    - Ruta **absoluta** del CSV donde se guardarán los datos matcheados.
    - URL de conexión TCP del broker de ActiveMQ.
    - Tópicos del broker (**deben ser obligatoriamente**):
  
      ```
      Flights
      Weather
      ```
    - Rutas relativas de los CSVs crudos (**deben ser obligatoriamente**):
      
      ```
      flight-delay-estimator/src/main/resources/datamart-partition-for-raw-flights.csv  
      flight-delay-estimator/src/main/resources/datamart-partition-for-raw-weather.csv
      ```
  - Ruta **absoluta** para guardar los datos procesados.

#### 📦 OpenWeatherMapFeeder

- Ir al archivo `main` del módulo.
- Argumentos esperados en orden (uno por línea):

  - Ruta **absoluta** de la base de datos.
  - URL de conexión TCP del broker de ActiveMQ.
  - Ruta **relativa** del archivo CSV de IATAs, debe ser:  
     `openweathermap-feeder/src/main/resources/iata-icao.csv`
  - Una **apikey PREMIUM** (Plan Estudiante, Professional o superior) de OpenWeatherMap.
  - Cuatro códigos **IATA** de aeropuertos (deben coincidir con los usados en el otro feeder).
    
---

### 4. Preparación para Tests (Opcional)

#### AviationStackFeeder

- Crear la carpeta:  
  `test/resources`
- Crear los archivos:
  - `Apikeys.txt` → Varias claves (separadas por espacios), la **primera debe ser válida**.
  - `ApiKeysFake.txt` → Varias claves (separadas por espacios), la **primera debe ser inválida**.

#### OpenWeatherMapFeeder

- Crear la carpeta:  
  `test/resources`
- Descargar el archivo `iata-icao.csv` y **moverlo fuera del proyecto**.
- Crear el archivo:
  - `apikey.txt` → Contiene dos valores separados por un espacio:  
    1. Una apikey **válida**.  
    2. La **ruta absoluta** al archivo `iata-icao.csv`.

## Tutorial de ejecución con ejemplos

Modos de ejecución:

- **Uso del entorno de mensajería e invocación de la UI:**

    Es el modo principal de ejecución. Se realiza una conexión a un broker de mensajería (en nuestro caso ActiveMQ); para que ambos feeders puedan enviar información en formato de eventos, que consisten en mensajes inmutables que perduran a lo largo del tiempo. De esta forma, AviationStackFeeder recoge la información de vuelos activos; y al día siguiente, OpenWeatherMapFeeder suministrará los valores climáticos de los aeropuertos elegidos por el usuario en la configuración de la aplicación. La frecuencia de actualización es de 1 día para ambas APIs (por defecto, a las 10:00 se actualiza el AviationStackFeeder y a las 11:00 el OpenWeatherMapFeeder).

    Todos estos eventos son almacenados en un EventStore para llevar un historial de mensajes recibidos.

    A su vez, el Datamart concentra toda esa información que pueda ser relevante para la propuesta de valor. Tiene la capacidad de, cargar el histórico de mensajes almacenados en el EventStore; y de recibir eventos en tiempo real, para hacer un posterior procesamiento de los eventos recibidos.

    En último lugar, el usuario podrá interactuar con la UI. (Ejemplos mostrados más abajo ↓)

    - Encender el broker de mensajería.
    - Ejecutar el main de Event-Store-Builder (para el almacenamiento de eventos).
    - Ejecutar el main de Flight-Delay-Estimator (para la carga de históricos y recepción de eventos en tiempo real, junto a la ejecución de la UI).
    - Ejecutar el main de AviationStackFeeder (para el envio automático de información):
        
        ```FlightController controller = new FlightController(new AviationStackProvider(new AviationStackProcessor(Arrays.copyOfRange(args,6,args.length)),new FlightJSONParser(), Arrays.copyOfRange(args,2,6)), new FlightEventStore(args[1],new FlightEventSerializer(),new FlightEventMapper()), new TaskScheduler());```

        ```controller.execute();```

    - Ejecutar el main de OpenWeatherMapFeeder:

        ```WeatherController controller = new WeatherController(new OpenWeatherMapProvider(new OpenWeatherMapProcessor(args[3]),new WeatherJSONParser(), Arrays.copyOfRange(args,4,args.length)), new WeatherEventStore(args[1],new WeatherEventMapper(),new WeatherEventSerializer()), new TaskScheduler(), new AirportToCoordinates(args[2]), new UnixUtils());```

        ```controller.execute();```

- **Guardado en SQLite:**

    Almacena en una base de datos de SQLite la información proveniente de las APIs (sin utilizar el Datamart, ni del EventStore, ni de la UI; simplemente escribe en la database).


    - Ejecutar el main de AviationStackFeeder:

        ```FlightController controller = new FlightController(new AviationStackProvider(new AviationStackProcessor(Arrays.copyOfRange(args,6,args.length)),new FlightJSONParser(), Arrays.copyOfRange(args,2,6)), new FlightSQLStore(args[0],new SQLConnection(),new SQLModifierFlights(), new FlightModelMapper()), new TaskScheduler());```

        ```controller.execute();```

    - Ejecutar el main de OpenWeatherMapFeeder:

        ```WeatherController controller = new WeatherController(new OpenWeatherMapProvider(new OpenWeatherMapProcessor(args[3]),new WeatherJSONParser(), Arrays.copyOfRange(args,4,args.length)), new WeatherSQLStore(args[0],new SQLModifierWeather(), new SQLConnection(), new WeatherResultMapper()), new TaskScheduler(), new AirportToCoordinates(args[2]), new UnixUtils());```

        ```controller.execute();```


### 📌 Ejemplos de uso

#### AviationStackFeeder:
- **Envío de mensajes al broker**

    Al ejecutar el ``main`` en modo **ActiveMQ**, el feeder enviará mensajes al broker en la hora programada:

    <img src="https://github.com/user-attachments/assets/0d853e87-d90d-4194-beaa-0ce11703bbc4" width="700">

- **Guardado de información en SQLite**

    Si se ejecuta el ``main`` en modo **SQLite**, los datos se almacenarán en la base de datos local:
      
    <img src="https://github.com/user-attachments/assets/d35901fc-ba87-4657-a2d0-fe37215a8a88" width="700">

- **Errores posibles de la API**

    La API de AviationStack puede fallar ocasionalmente por problemas internos. Si esto ocurre, lo más recomendable es **esperar al día siguiente y volver a intentar**:

    <img src="https://github.com/user-attachments/assets/c74e2079-84b4-41ef-905a-8df83070c06c" width="700">

#### OpenWeatherMapFeeder:
- **Envio de mensajes al broker**:

  <img src="https://github.com/user-attachments/assets/9dae77d9-9d24-4b02-b7da-142be5133b32" width="700">
    
- **Guardado de información en SQLite**

  <img src="https://github.com/user-attachments/assets/6e146a50-9df4-4d0a-93b9-579e9d5184d3" width="700">

#### EventStoreBuilder:

  <img src="https://github.com/user-attachments/assets/58c21017-5707-4b22-8d90-dfd4b84eee99" width="700">

## Tutorial de uso de la UI

El usuario puede interactuar con la CLI de la siguiente forma:

- Introduce el IATA del aeropuerto 
- Introduce si ese aeropuerto se toma como de salida o de llegada
- Elige el modelo predictivo del que quieras ver el rendimiento (disponibles: ```LinearRegression``` ```KNNRegressor``` son los modelos que mejor se adaptan teóricamente)
- Elige entre realizar otra consulta (```s```) o cerrar la interfaz (```n```), después de esta consulta; en caso de querer realizar otra consulta, actualiza el Datamart con los eventos a tiempo real. <br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://github.com/user-attachments/assets/2a0b9e9c-67f8-4e9d-848b-3588bed3f22a" width="650">

El Datamart comprobará con procesos programados periódicamente cada cinco minutos para verificar la incorporación de datos en tiempo real. Adicionalmente, cada treinta minutos se activará un procedimiento de conciliación y emparejamiento de datos entre las distintas fuentes de información, con el objetivo de garantizar su integridad y coherencia.

El usuario será notificado de la ejecución de estos procesos mediante mensajes de estado generados por el sistema, los cuales reflejan el progreso y los resultados de las tareas programadas.

### Arquitecturas del sistema y aplicación


![Sistema](https://github.com/user-attachments/assets/456f992f-c6a0-4869-b70a-77a686a54f0e)


[Diagrama de clases de AviationStackFeeder](https://github.com/user-attachments/assets/d71b49f5-468f-4fb4-9aa9-22921dc00ebd)

[Diagrama de clases de EventStoreBuilder](https://github.com/user-attachments/assets/11ea1e2a-01e7-4d2b-9e4d-e9ba79775fdc)

[Diagrama de clases de OpenWeatherMapFeeder](https://github.com/user-attachments/assets/1b0cb9fa-b874-4466-8498-6f8f7168eedc)

[Diagrama de clases de FlightDelayEstimator](https://github.com/user-attachments/assets/4aafacd8-d96b-4c0f-8f43-dc89a0b3a1b3)

### Principios y patrones de diseño aplicados en cada módulo

En los feeders, la arquitectura implementada sigue un diseño modular de tipo hexagonal, lo que permite una clara separación entre el núcleo de la aplicación y sus interfaces externas, como bases de datos, APIs o interfaces de usuario. Esto facilita el desacoplamiento y mejora la flexibilidad del sistema. Cada módulo está diseñado conforme al Single Responsibility Principle, SRP, asegurando que cada componente tenga un propósito bien definido. Esto mejora la mantenibilidad, facilita las pruebas y permite realizar cambios sin afectar otras partes del sistema. Además, se aplica el Open/Closed Principle (OCP), permitiendo que los módulos puedan ser extendidos con nuevas funcionalidades sin necesidad de modificar el código existente, favoreciendo así la escalabilidad y el mantenimiento del sistema. 

Ejemplos de OCP:

```
public interface FlightStore {
    public void saveFlights (FlightResponse flightResponse);
}
```
```
public interface FlightProvider {
    FlightResponse flightProvider(String airportType, String airportIata);
    String[] getPreferredAirports();
}
```

Estas interfaces permiten que se puedan añadir nuevas tecnologías al código si surgiese la necesidad; y no haría falta modificar el resto del código. Por ejemplo, una implementación de FlightStore para guardar datos en Oracle o MySQL. Esta dinámica es idéntica en el otro feeder. Asimismo, se podría introducir otra tecnología de recolección de datos que no sea mediante APIs.

En el EventStoreBuilder, se sigue también una arquitectura hexagonal; aunque no se aprecia al consistir de un número de clases muy pequeño.

En último lugar, el módulo de la unidad de negocio (FlightDelayEstimator) sigue una estructura similar a la hexagonal. Se refleja el OCP en la siguiente interfaz:

```
public interface ProcessInvoker {
    public void executeExternalProcess() throws IOException, InterruptedException;
}
```

El OpenClosedPrinciple permitiría introducir un proceso de análisis de datos diferente si se desease en el futuro, sin cambiar demasiado código (por ejemplo, un proceso en R u otro lenguaje).

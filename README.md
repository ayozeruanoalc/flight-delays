# FlightDelays

[![My Skills](https://go-skill-icons.vercel.app/api/icons?i=java,idea,maven,junit,python,vscode,anaconda,scikitlearn,pandas,api,sqlite,uml,github)](https://go-skill-icons.vercel.app/api/)


### Descripción del proyecto y propuesta de valor

FlightDelays es una aplicación desarrollada en Java que permite registrar, procesar y correlacionar datos sobre retrasos de vuelos con las condiciones meteorológicas asociadas a los aeropuertos de origen y destino. El sistema se basa principalmente en ActiveMQ como núcleo de su arquitectura, utilizando esta plataforma de mensajería para desacoplar procesos y gestionar eventos de forma asíncrona. Adicionalmente, cuenta con un mecanismo opcional de persistencia de datos mediante SQLite, que puede ser utilizado según las necesidades del entorno o configuración.

Tecnologías utilizadas:
- Lenguaje de programación: Java 21
- Lenguaje auxiliar: Python 3.11.9
- Gestor de dependencias: Apache Maven
- Base de datos: SQLite
- Mensajería asíncrona: Apache ActiveMQ
- IDE de desarrollo: IntelliJ IDEA

El valor que se aporta específicamente al usuario se trata de una UI, a la que es posible hacerle consultas sobre el rendimiento de determinados clasificadores
(predictores), que tomen las condiciones climáticas de aeropuertos; e intenten explicar los retrasos de vuelos mediante las mismas. De esta manera, si algún modelo alcanzase un valor de error lo suficientemente pequeño al haberse entrenado con una cantidad de registros considerable, querrá decir que sería capaz de hacer predicciones sobre retrasos con un grado de precisión considerable. El usuario será capaz de elegir el aeropuerto y si se considera de llegada o salida (tal vez el clima afecte más en el retraso de un aterrizaje que en el de un despegue), y cuál de los modelos disponibles se entrenará.

### Justificación de la elección de APIs y estructura del Datamart 

La API utilizada para recabar información sobre el tráfico aéreo es AviationStackAPI, seleccionada por su amplio alcance de datos a nivel global. Además, permite obtener información detallada sobre vuelos, aeropuertos, aerolíneas y estados de vuelos, lo cual es fundamental para el análisis de retrasos y su correlación con condiciones externas.

Para los datos meteorológicos, se integró la API de OpenWeatherMap, que proporciona datos históricos y en tiempo real sobre el clima en ubicaciones específicas, permitiendo así una correlación efectiva entre el estado del tiempo y los retrasos en vuelos. 

**La estructura de datamart planteada es la siguiente:** 
- Particiones para eventos recibidos en tiempo real: 2 archivos CSV, que se encargan de almacenar la información que envíe el broker en tiempo real (1 archivo por tópico implementado; en este caso, Flights y Weather). Cuentan con un campo de marca temporal, para gestionar la asincronía y realizar matching de forma óptima; y otro campo donde se guardan los eventos provenientes del broker en formato crudo (como json).
- Partición limpia (con matching aplicado): 1 archivo CSV, que representa la información valiosa a la que posteriormente se le hará análisis (via Python); también se le puede ver como el resultado de emparejar eventos de vuelos y climas que sean compatibles (con esto se refiere a que tengan registros temporales muy cercanos en el tiempo), ya sea su fuente un histórico o las particiones para eventos en tiempo real.
- Partición procesada: 1 archivo CSV, que es el resultado del análisis de Python efectuado a la partición limpia. Contiene toda la información que le pudiese ser útil al usuario; que podrá ser accesible mediante la UI.

La estructura del Datamart planteada ofrece una serie de ventajas clave que justifican su diseño. En primer lugar, destaca por su modularidad, ya que cada etapa del flujo de datos —captura, transformación y análisis— está claramente separada, lo que facilita tanto el mantenimiento como la identificación y corrección de errores. Además, la arquitectura es escalable, permitiendo que componentes específicos, como el proceso de emparejamiento de eventos (matching), puedan evolucionar o ampliarse sin impactar el funcionamiento del resto del sistema. Esta separación de responsabilidades también aporta una alta flexibilidad, ya que permite modificar o reemplazar herramientas o tecnologías (por ejemplo, sustituir el motor de análisis en Python por otro) sin necesidad de rediseñar la solución completa. Por último, se logra una mayor robustez frente a fallos gracias al almacenamiento intermedio en archivos CSV, que actúan como puntos de control persistentes, permitiendo la recuperación o reprocesamiento de los datos ante interrupciones o errores inesperados. Esta combinación de características hace que la arquitectura sea sólida, mantenible y adaptable a distintos contextos y requerimientos.

### Configuración

1. Instalar el ActiveMQ en tu equipo.
2. Es necesario tener instalado Python (v3.11.9 o superiores). También debe estar definido como variable de entorno del sistema.
3. Maven debe poder ejecutarse en IntelliJ. Si diese error, reinstalar Maven, agregar a variables del sistema y ejecutar en IntelliJ ```mvn clean install```. 
5. Clonar el proyecto de Github en IntelliJ con la opción de **Repository URL**, pegando el link del repositorio.
6. Preparar los módulos para su funcionamiento: 
    - Ir al main de AviationStackFeeder:
        - **Argumentos en orden (salto de línea para separarlos):** 
            - Ruta absoluta de database.
            - Enlace URL de conexión TCP del broker de ActiveMQ. (Ejemplo: ```tcp://localhost:12345```)
            - Cuatro códigos IATA de aeropuertos, con los que trabajará el feeder. (Ejemplos: ```MAD``` ```AMS``` ```JFK``` ```ZRH```)*
            - Número indefinido, elegido por el consumidor, de apiKeys de AviationStack.*
    - Ir al main de Event-Store-Builder:
        - **Argumentos en orden (salto de línea para separarlos):** 
            - Enlace URL de conexión TCP del broker de ActiveMQ.
            - Tópicos del broker de mensajería de ActiveMQ. (Obligatoriamente deben ser: ```Flights``` y ```Weather```)*
    - Ir al main de Flight-Delay-Estimator:
        - **Argumentos en orden (salto de línea para separarlos):**
            - Ruta relativa del histórico de vuelos (obligatoriamente: ```eventstore/Flights/AviationStackFeeder```).
            - Ruta relativa del histórico de climas (obligatoriamente: ```eventstore/Weather/OpenWeatherMapFeeder```).
            - Ruta absoluta del CSV para guardar los archivos matcheados.
            - Enlace URL de conexión TCP del broker de ActiveMQ.
            - Tópicos del broker de mensajería de ActiveMQ (obligatoriamente deben ser: ```Flights``` y ```Weather```)*
            - Rute relativa de CSVs crudos. (tienen que ser obligatoriamente: ```flight-delay-estimator/src/main/resources/datamart-partition-for-raw-flights.csv``` y ```flight-delay-estimator/src/main/resources/datamart-partition-for-raw-weather.csv```)*
            - Ruta absoluta para guardar datos procesados.
    - Ir al main de OpenWeatherMapFeeder:
        - **Argumentos en orden (salto de línea para separarlos):** 
            - Ruta absoluta de database.
            - Enlace URL de conexión TCP del broker de ActiveMQ.
            - Ruta relativa del archivo CSV de IATAs e ICAOs. (Debe ser obligatoriamente: ```openweathermap-feeder/src/main/resources/iata-icao.csv```)
            - Una apikey **PREMIUM** (Plan Estudiante, Professional o superior), proporcionada por OpenWeatherMap.
            - Cuatro códigos IATA de aeropuertos, con los que operará el feeder. (Tienen que ser los mismos que se le pasen al otro feeder)*
    
    _* Separar con salto de línea_

Si tiene pensado hacer test tanto de AviationStackFeeder como de OpenWeatherMapFeeder, hacer lo siguiente:
- AviationStackFeeder:
   - Crear la carpeta ```test/resources```.
   - Crear ```Apikeys.txt``` con varias claves, separadas por espacios, la primera debe ser válida.
   - Crear ```ApiKeysFake.txt``` con varias claves, separadas por espacios, la primera no debe ser válida.
- OpenWeatherMapFeeder:
   - Crear la carpeta ```test/resources```.
   - Descargar el ```iata-icao.csv``` y moverlo fuera del proyecto.
   - Crear ```apikey.txt``` con apikey válida, y separado con un espacio, la ruta absoluta al CSV del ```iata-icao.csv```.


### Tutorial de ejecución con ejemplos

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


### Ejemplos de uso

- AviationStackFeeder:
    - **Envio de mensajes al broker** (habiendo ejecutado el main en modo ActiveMQ, pasará lo siguiente a la hora programada):
  
        <img src="https://github.com/user-attachments/assets/0d853e87-d90d-4194-beaa-0ce11703bbc4" width="700">

    - **Guardado de información en SQLite** (habiendo ejecutado el main en modo SQLite, ocurrirá lo siguiente):
      
      <img src="https://github.com/user-attachments/assets/d35901fc-ba87-4657-a2d0-fe37215a8a88" width="700">

    Esta API podría dar algún error al ejecutar debido a errores internos dentro de ella, si eso ocurre lo más recomendado es esperar al día siguiente:

    <img src="https://github.com/user-attachments/assets/c74e2079-84b4-41ef-905a-8df83070c06c" width="700">

- OpenWeatherMapFeeder:
    - **Envio de mensajes al broker**:
        
      <img src="https://github.com/user-attachments/assets/9dae77d9-9d24-4b02-b7da-142be5133b32" width="700">
    
    - **Guardado de información en SQLite**

      <img src="https://github.com/user-attachments/assets/6e146a50-9df4-4d0a-93b9-579e9d5184d3" width="700">

- EventStoreBuilder:

  <img src="https://github.com/user-attachments/assets/58c21017-5707-4b22-8d90-dfd4b84eee99" width="700">

### Tutorial de uso de la UI

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

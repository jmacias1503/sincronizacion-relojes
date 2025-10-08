# Sincronización de Relojes
## Objetivo
Implementar y comparar los algoritmos de Cristian, Berkeley y NTP simulado localmente, usando placas ESP32 como nodos clientes y tu computadora como servidor de tiempo. No se permite usar servidores NTP en línea, toda la comunicación debe ser local.
### Requerimientos
- Al menos 1 ESP32 actuando como nodos sincronizados con la computadora.
- Una computadora que funja como servidor de tiempo, con scripts en Python.
- Comunicación vía Wifi Mesh (painlessMesh) usando Json.
- Mostrar resultados por Serial Monitor de cada ESP32.
### Parte 1 - Algoritmo de Cristian
1. Tu computadora actuará como servidor UDP de tiempo.
    1. El script en Python debe enviar su hora actual (timestamp) al recibir el mensaje "TIME".
2. Cada ESP32 enviará una solicitud al servidor y calculará:
$$T_{client} = T_{server} + \frac{(T_1 + T_0)}{2}$$
    donde:
    - $T_{client}$: Hora sincronizada del cliente (el resultado del algoritmo).
    - $T_{server}$: Hora devuelta por el servidor de tiempo cuando respondió la petición.
    - $T_0$: Tiempo en el cliente cuando se envió la petición al servidor.
    - $T_1$: Tiempo en el cliente cuando se recibió la respuesta del servidor.
3. Mostar en el monitor:
    1. Hora antes de sincronizar
    2. Hora recibida del servidor
    3. Error: $error \in [-\frac{(T_1 - T_0)}{2}, + \frac{(T_1 - T_0)}{2}]$
    4. Hora ajustada estimada
### Parte 2 - Algoritmo de Berkeley
1. Uno de los ESP32 será el coordinador (maestro).
2. Solicitud de tiempos. El maestro envía una solicitud a todos los demás nodos para conocer su hora actual. Cada nodo responde con su tiempo local $T_i$ , obtenido mediante `millis()` o `getNodeTime()`.
3. Cálculo de diferencias. Al recibir las respuestas, el maestro calcula su propio tiempo local $T_m$ y determina la diferencia de cada nodo respecto al suyo:
$$\Delta_i = T_i - T_m$$
4. Filtrado de valores atípicos: Antes de promediar, se eliminan diferencias $\Delta_i$ que excedan un umbral definido (por ejemplo, $|\Delta_i| > 2\sigma$) para evitar que valores erróneos afecten la sincronización.
5. Cálculo del promedio de diferencias. Se calcula la diferencia promedio considerando únicamente los valores válidos:
$$\Delta = \frac{1}{n} \sum_{i = 1}^{n} \Delta_i$$
6. Ajuste del reloj maestro. El coordinador actualiza su propio reloj aplicando el promedio calculado:
$$T'_m = T_m + \Delta$$
7. Cálculo de los ajustes para cada nodo: El maestro determina cuánto debe ajustar cada cliente su reloj:
$$ajuste_i = \Delta - \Delta_i$$
Este valor se envía a cada nodo
8. Ajuste local de los clientes. Cada ESP32 actualiza su tiempo local aplicando el ajuste recibido:
$$T'_i = T_i + ajuste_i$$
Este lo probaremos en el laboratorio
### Parte 3 - Algoritmo NTP
1. El cliente envía una solicitud (Request):
    El nodo cliente (ESP32) envía un paquete NTP al servidor, incluyendo su tiempo local en el instante de envío:
    $T_1$ = hora en que el cliente envia la solicitud

2. El servidor recibe la solicitud:
3. Al llegar el mensaje, el servidor anota su hora local de recepción:
    $T_2$ = hora en que el servidor recibe la solicitud
4. El servidor responde:
    Antes de enviar la respuesta, el servidor marca el tiempo de envío:
    $T_3$ = hora en que el servidor envía la respuesta
    La respuesta incluye los valores $T_1, T_2, T_3$
5. El cliente recibe la respuesta:
6. Cuando el mensaje llega de vuelta, el cliente anota:
    $T_4$ = hora en que el cliente recibe la respuesta
7. Cálculo del retardo y el desfase:
    Con estos cuatro tiempos, el cliente puede calcular:
8. Retardo total de ida y vuelta (Round-Trip Delay):
    $$\delta = (T_4 - T_1) - (T_3 - T_2)$$
9. Desfase de reloj (Clock Offset):
    $$\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$$
10. Ajuste del reloj local del cliente:
    Finalmente, el cliente corrige su reloj con base en el desfase calculado:
    Tiempo de cliente ajustado = $T_4 + \theta$

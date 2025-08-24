
### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW
## Ejercicio Introducción al paralelismo - Hilos - Caso BlackListSearch

## Integrantes
- Ricardo Andrés Ayala Garzón [lRicardol](https://github.com/lRicardol)
- Santiago Amaya Zapata [SantiagoAmaya21](https://github.com/SantiagoAmaya21)

### Dependencias:
####   Lecturas:
*  [Threads in Java](http://beginnersbook.com/2013/03/java-threads/)  (Hasta 'Ending Threads')
*  [Threads vs Processes]( http://cs-fundamentals.com/tech-interview/java/differences-between-thread-and-process-in-java.php)

### Descripción
  Este ejercicio contiene una introducción a la programación con hilos en Java, además de la aplicación a un caso concreto.
  

**Parte I - Introducción a Hilos en Java**

1. De acuerdo con lo revisado en las lecturas, complete las clases CountThread, para que las mismas definan el ciclo de vida de un hilo que imprima por pantalla los números entre A y B.
2. Complete el método __main__ de la clase CountMainThreads para que:
	1. Cree 3 hilos de tipo CountThread, asignándole al primero el intervalo [0..99], al segundo [99..199], y al tercero [200..299].
	2. Inicie los tres hilos con 'start()'.
   
![Captura de pantalla 2025-08-23 145733.png](img/Captura%20de%20pantalla%202025-08-23%20145733.png)

3. Ejecute y revise la salida por pantalla.

![Captura de pantalla 2025-08-23 150141.png](img/Captura%20de%20pantalla%202025-08-23%20150141.png)
4. Cambie el incio con 'start()' por 'run()'. Cómo cambia la salida?, por qué?.

Cuando se inician un hilo con start(), cada uno de los prosesos se ejecuta simultáneamente en diferentes subprocesos de la JVM. Esto significa que el sistema operativo determina el orden de ejecución y, por lo tanto, la salida de la consola aparece mixta, con números de diferentes hilos intercalados.

En cambio, si se llama directamente al método run(), no se crean subprocesos adicionales: el código de cada objeto CountThread se ejecuta como un método normal en el subproceso principal. En este caso, la salida es secuencial (primero de 0 a 99, luego de 100 a 199 y finalmente de 200 a 299), ya que no hay concurrencia real.

**Parte II - Ejercicio Black List Search**


Para un software de vigilancia automática de seguridad informática se está desarrollando un componente encargado de validar las direcciones IP en varios miles de listas negras (de host maliciosos) conocidas, y reportar aquellas que existan en al menos cinco de dichas listas. 

Dicho componente está diseñado de acuerdo con el siguiente diagrama, donde:

- HostBlackListsDataSourceFacade es una clase que ofrece una 'fachada' para realizar consultas en cualquiera de las N listas negras registradas (método 'isInBlacklistServer'), y que permite también hacer un reporte a una base de datos local de cuando una dirección IP se considera peligrosa. Esta clase NO ES MODIFICABLE, pero se sabe que es 'Thread-Safe'.

- HostBlackListsValidator es una clase que ofrece el método 'checkHost', el cual, a través de la clase 'HostBlackListDataSourceFacade', valida en cada una de las listas negras un host determinado. En dicho método está considerada la política de que al encontrarse un HOST en al menos cinco listas negras, el mismo será registrado como 'no confiable', o como 'confiable' en caso contrario. Adicionalmente, retornará la lista de los números de las 'listas negras' en donde se encontró registrado el HOST.

![](img/Model.png)

Al usarse el módulo, la evidencia de que se hizo el registro como 'confiable' o 'no confiable' se dá por lo mensajes de LOGs:

INFO: HOST 205.24.34.55 Reported as trustworthy

INFO: HOST 205.24.34.55 Reported as NOT trustworthy


Al programa de prueba provisto (Main), le toma sólo algunos segundos análizar y reportar la dirección provista (200.24.34.55), ya que la misma está registrada más de cinco veces en los primeros servidores, por lo que no requiere recorrerlos todos. Sin embargo, hacer la búsqueda en casos donde NO hay reportes, o donde los mismos están dispersos en las miles de listas negras, toma bastante tiempo.

Éste, como cualquier método de búsqueda, puede verse como un problema [vergonzosamente paralelo](https://en.wikipedia.org/wiki/Embarrassingly_parallel), ya que no existen dependencias entre una partición del problema y otra.

Para 'refactorizar' este código, y hacer que explote la capacidad multi-núcleo de la CPU del equipo, realice lo siguiente:

1. Cree una clase de tipo Thread que represente el ciclo de vida de un hilo que haga la búsqueda de un segmento del conjunto de servidores disponibles. Agregue a dicha clase un método que permita 'preguntarle' a las instancias del mismo (los hilos) cuantas ocurrencias de servidores maliciosos ha encontrado o encontró.

2. Agregue al método 'checkHost' un parámetro entero N, correspondiente al número de hilos entre los que se va a realizar la búsqueda (recuerde tener en cuenta si N es par o impar!). Modifique el código de este método para que divida el espacio de búsqueda entre las N partes indicadas, y paralelice la búsqueda a través de N hilos. Haga que dicha función espere hasta que los N hilos terminen de resolver su respectivo sub-problema, agregue las ocurrencias encontradas por cada hilo a la lista que retorna el método, y entonces calcule (sumando el total de ocurrencuas encontradas por cada hilo) si el número de ocurrencias es mayor o igual a _BLACK_LIST_ALARM_COUNT_. Si se da este caso, al final se DEBE reportar el host como confiable o no confiable, y mostrar el listado con los números de las listas negras respectivas. Para lograr este comportamiento de 'espera' revise el método [join](https://docs.oracle.com/javase/tutorial/essential/concurrency/join.html) del API de concurrencia de Java. Tenga también en cuenta:

	* Dentro del método checkHost Se debe mantener el LOG que informa, antes de retornar el resultado, el número de listas negras revisadas VS. el número de listas negras total (línea 60). Se debe garantizar que dicha información sea verídica bajo el nuevo esquema de procesamiento en paralelo planteado.

	* Se sabe que el HOST 202.24.34.55 está reportado en listas negras de una forma más dispersa, y que el host 212.24.24.55 NO está en ninguna lista negra.


**Parte II.I Para discutir la próxima clase (NO para implementar aún)**

La estrategia de paralelismo antes implementada es ineficiente en ciertos casos, pues la búsqueda se sigue realizando aún cuando los N hilos (en su conjunto) ya hayan encontrado el número mínimo de ocurrencias requeridas para reportar al servidor como malicioso. Cómo se podría modificar la implementación para minimizar el número de consultas en estos casos?, qué elemento nuevo traería esto al problema?

Para optimizar, se podría introducir un contador global de coincidencias y un mecanismo de detención temprana que interrumpa a los hilos cuando el host ya puede considerarse malicioso. Esto traería un nuevo elemento de coordinación entre hilos, ya que sería necesario gestionar sincronización y comunicación para compartir el estado global de la búsqueda y evitar condiciones de carrera.

**Parte III - Evaluación de Desempeño**

A partir de lo anterior, implemente la siguiente secuencia de experimentos para realizar las validación de direcciones IP dispersas (por ejemplo 202.24.34.55), tomando los tiempos de ejecución de los mismos (asegúrese de hacerlos en la misma máquina):

1. Un solo hilo.
![hilo 1.png](img/hilo%201.png)
2. Tantos hilos como núcleos de procesamiento (haga que el programa determine esto haciendo uso del [API Runtime](https://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html)).
![Hilos = Nucleos.png](img/Hilos%20%3D%20Nucleos.png)
3. Tantos hilos como el doble de núcleos de procesamiento.
![Hilos = Núcleos x2.png](img/Hilos%20%3D%20N%C3%BAcleos%20x2.png)
4. 50 hilos.
![50 hilos.png](img/50%20hilos.png)
5. 100 hilos.
![100 hilos.png](img/100%20hilos.png)

Al iniciar el programa ejecute el monitor jVisualVM, y a medida que corran las pruebas, revise y anote el consumo de CPU y de memoria en cada caso. ![](img/jvisualvm.png)

Con lo anterior, y con los tiempos de ejecución dados, haga una gráfica de tiempo de solución vs. número de hilos. Analice y plantee hipótesis con su compañero para las siguientes preguntas (puede tener en cuenta lo reportado por jVisualVM):

![Grafica T vs N.png](img/Grafica%20T%20vs%20N.png)

Analizando la gráfica y viendo los tiempos que tardan las diferentes cantidades de hilos, podemos afirmar que a medida que se aumentan los hilos en intervalos de 1 hasta 20, el tiempo de ejecución baja drásticamente en comparación con valores mayores a 20 o 30, donde el tiempo si bien sigue bajando, su pendiente es mucho menor por lo que el cambio suele ser muy pequeño.

**Parte IV - Ejercicio Black List Search**

1. Según la [ley de Amdahls](https://www.pugetsystems.com/labs/articles/Estimating-CPU-Performance-using-Amdahls-Law-619/#WhatisAmdahlsLaw?):

	![](img/ahmdahls.png), donde _S(n)_ es el mejoramiento teórico del desempeño, _P_ la fracción paralelizable del algoritmo, y _n_ el número de hilos, a mayor _n_, mayor debería ser dicha mejora. Por qué el mejor desempeño no se logra con los 500 hilos?, cómo se compara este desempeño cuando se usan 200?. 

Aunque la fórmula sugiere que a mayor número de hilos mayor será S(n), no quiere decir que sea lo mejor agregar más y más hilos, ya que la función tiene límite, cuando n crece, se va acercando a 1/(1-p), dependiendo de que valor tenga p, el comportamiento de la función cuando n = 500, es similar a valores menores, incluso mucho menores que 500, entonces colocar 500 hilos en este ejercicio resulta exagerado teniendo en cuenta de que 200 hilos ofrecen un desempeño similar, además de que con 500 hilos o 200 se compite por los mismos recursos de la CPU.

2. Cómo se comporta la solución usando tantos hilos de procesamiento como núcleos comparado con el resultado de usar el doble de éste?.

Es mejor usar el doble de núcleos, ya que el tiempo se redujo en casi la mitad.

3. De acuerdo con lo anterior, si para este problema en lugar de 100 hilos en una sola CPU se pudiera usar 1 hilo en cada una de 100 máquinas hipotéticas, la ley de Amdahls se aplicaría mejor?. Si en lugar de esto se usaran c hilos en 100/c máquinas distribuidas (siendo c es el número de núcleos de dichas máquinas), se mejoraría?. Explique su respuesta.

Escenario distribuido con 100 máquinas: Una ventaja de este escenario es que todos los hilos no estarán compitiendo por una única CPU, pero aparece un nuevo factor que no es tenido en cuenta en la ley de Amdalhs, temas de red, latencia y sincronización entrarían al problema.

Escenario con c hilos en 100/c máquinas (con c núcleos cada una): La idea es balancear hilos y núcleos: cada máquina puede ejecutar c hilos de manera natural (un hilo por núcleo). Así se mantiene un uso eficiente de los recursos y se reduce la sobrecarga de administrar más hilos de los que núcleos disponibles. En este caso, el desempeño sería similar o incluso mejor que tener 100 máquinas de un solo núcleo, porque se aprovecha mejor la capacidad multinúcleo local antes de escalar a la red.
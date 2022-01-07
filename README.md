PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/pluginfile.php/3145524/mod_assign/introattachment/0/spk_8mu.tgz?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
  
  **El programa sox nos permite generar una señal con el formato adecuado a partir de otro formato. En nuestro programa el formato de entrada, identificado con el comando '-t' es raw (crudo). '-e' conforma la codificación en un formato 'signed' con '-b' dieciséis bits.**
  
  **El comando de SPTK x2x nos permite convertir datos de una entrada estandard a otro tipo de datos diferente. En nuestra línea de comando, convertimos short (2 bytes) a float (4 bytes) con la orden '+sf'.**
  
  **Frame extrae una trama de una secuencia de datos. de longitud '-l' y período '-p'.**
  
  **Window enventana una señal. La longitud de datos de entrada es la de la trama '-l' y el resultado del enventanado tiene una longitud '-L'.**
  
  **LPC calcula los coeficientes de predicción lineal de la trama de longitud '-l' y el orden de los coeficientes lo determinamos con '-m'.** 
  

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
    **Para obtener el fichero fmatrix a partir de los ficheros de salida de sptk el número de columnas se calcula a partir del orden del predictor lineal. La obtención del número de filas depende de la longitud de la señal, longitud y desplazamiento de la ventana, y la cadena de comandos que se ejecutan para obtener la parametrización. Para simplificar la tarea, convertimos la señal parametrizada a texto usando X2X +fa y contamos el número de líneas con el comando wc -l de UNIX.**

  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
    **Es conveniente porque de esta manera podemos guardar los coeficientes que creamos necesarios de los ficheros de un locutor en concreto en un fichero. Como se verá a continuación con los coeficientes 2 y 3.**

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 | $LPC -l 240 -m $lpc_order |
	$LPCC -m $lpc_order -M $cepstrum_order > $base.lpcc
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
	$MFCC -l 240 -m $mfcc_order -s $sampling_frequency -n $channel_order > $base.mfcc
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  

  
  
  ![](captures/coef2_3_lp.png) ![](captures/coef2_3_mfcc.png) ![](captures/coef2_3_lpcc.png)
  

  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
  **Usando un código de python hemos podido obtener las gráficas que relacionan los coeficientes 2 y 3 de cada una de las parametrizaciones.**
  
  **Usamos el siguiente comando para obtener los datos**
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  fmatrix_show work/lp/BLOCK01/SES017/*.lp | egrep '^\[' | cut -f2,3 > lp_2_3.txt
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  **Cambiamos 'lp' por la parametrización que queremos analizar (MFCC o LPCC).**
  ```python
  
  f = open('lp_2_3.txt', "r")
  leer_lineas = f.read()
  fil=[]
  col=[]
  x=""
  y=""
  tab = False
  for i in leer_lineas:
    if i != '\t' and tab == False:
      x = x+i
    
    elif i == '\t':
      fil.append(float(x))
      tab = True

    if tab == True and i != '\t' and i!='\n':
      y = y + i
    elif i == '\n':
      col.append(float(y))
      x=""
      y=""
      tab = False
    
  plt.scatter(fil,col,s = 0.1)
  plt.title('Dependencia coeficientes 2 y 3. Param = lp')
  plt.xlabel('Coef 2')
  plt.ylabel('Coef 3')
  plt.show()
 
  ```
  
  
  + ¿Cuál de ellas le parece que contiene más información?

**La información de cada una está relacionada con la correlación entre componentes, ya que la información que nos proporciona un componente es mayor cuanto menor sea la correlación o, mejor dicho, cuanto mayor sea la intercorrelación. Observando las 3 gráficas generadas con los coeficientes 2 y 3 de cada una de las parametrizaciones, vemos que aquella en la que están más dispersos es en la de los coeficientes MFCC, por lo que se puede derivar a una mayor incorrelación entre ellos, lo que supone en una cantidad de información mayor.**

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        |    LP    |   LPCC   |     MFCC   |
  |------------------------|:--------:|:--------:|:----------:|
  | &rho;<sub>x</sub>[2,3] | -0.87228 | 0.769145 | -0.146628  |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.

**Los resultados son los esperados puesto que un valor cercano a +-1 del coeficiente Pearson implica una alta correlación mientras que un valor cercano al 0 implica una mayor incorrelación. Observando los valores obtenidos, vemos como la parametrización MFCC es la que nos genera un coeficiente de Pearson más cercano a 0, por lo que la información que nos proporciona cada componente es mayor que en las otras 2 parametrizaciones.**
  
  **Mediante otro script de python obtenemos el valor del parámetro de pearson a partir de los datos que constituyen los gráficos.**
  
  |    De nuestros datos   |    LP    |   LPCC   |     MFCC   |
  |------------------------|:--------:|:--------:|:----------:|
  | &rho;<sub>x</sub>[2,3] | -0.18018 |  0.76197 |  -0.13334  |
  
  **Observamos que los valores obtenidos con LPCC y MFCC son muy parecidos a los calculados con Pearson mientras que en el caso de LP difiere bastante.**
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?
  
  **Para la parametrización MFCC, el orden de los coeficientes es suficiente que sea 13 ya que a partir de ahí la mejora no es significativa y se usan entre 24 y 40 bandas de frecuencia. Para los coeficientes LPCC, es común usar a partir de 12 coeficientes.**

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  ![Gráfica con 2 primeros coeficientes de MFCC](https://github.com/xpons99/P4/blob/garcia-pons/captures/GMM_2coeff_MFCC_SES008.PNG)
  
  **Podemos observar como, en este caso, la población modelada sí presenta un carácter multimodal aunque con el GMM podemos modelarlo correctamente.**
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
  ![Gráfica de comparación modelos y poblaciones de 2 locutores distintos](https://github.com/xpons99/P4/blob/garcia-pons/captures/Modelos_locutores_4.PNG)
  
  **Observando estas imágenes se puede destacar la gran adaptación que tiene el modelo de cada locutor a sus propios datos, por lo que en estos casos el uso del modelado GMM nos permite una buena diferenciación entre las señales de uno y otro.**

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  ![Error LP](https://github.com/xpons99/P4/blob/garcia-pons/captures/lp18T0.0001N200m60i1.PNG)
  
  ![Error LPCC](https://github.com/xpons99/P4/blob/garcia-pons/captures/lpcc1525T0.0001N200m60i1.PNG)
  
  ![Error MFCC](https://github.com/xpons99/P4/blob/garcia-pons/captures/error_rate.png)
  
  |               |    LP    |   LPCC   |    MFCC   |
  |---------------|:--------:|:--------:|:---------:|
  | Tasa de error |  7.01%   |   3.31%  |   0.76%   |

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
  
  **Nuestro mejor sistema de verificación del locutor lo hemos obtenido usando MFCC y presentaba las características siguientes:**
  
  ![Verif_err MFCC](https://github.com/xpons99/P4/blob/garcia-pons/captures/verif_err.png)
  
  |                  |   MFCC   |
  |------------------|:--------:|
  |   Umbral óptimo  |  0.0413  | 
  | # Falsas Alarmas |  7/1000  |
  |    # Perdidas    |  16/250  |
  |       Score      |   12,7   |
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.

---
layout: single
title: Photorealistic style transfer
read_time: false

---

<a title="Hermosillo, sonora" href="https://ricardoamata.github.io/images/output_1.png">
    <img src="https://ricardoamata.github.io/images/output_1.png" alt="Hermosillo, sonora">
</a>



## Introducción

En este blog se explicara en que consiste la transferencia de estilo y como conseguir
que esta sea fotorrealista, se mostrara de que manera se puede usar una red neuronal
pre entrenada en reconocimiento de imágenes para alcanzar este resultado, así como
las mejores estrategias a seguir, ya que el proceso no es una receta de cocina la 
cual seguir al pie de la letra y depende mucho de las entradas. Al final mostraremos
algunos métodos de post procesamiento y algunas formas de mejorar los resultados obtenidos
en base a nuevas ideas propuestas recientemente.

## Red neuronal

Una red neuronal se puede ver como una colección de nodos conectados entre si a los que se
les llama neuronas, cada neurona recibe una serie de señales que son multiplicadas cada una
por un peso y en base a eso emite una señal de salida, así que una neurona solamente calcula
una combinación lineal de las señales de entrada y al resultado se le suma un sesgo, por lo
tanto no son mas que aproximadores lineales. Las señales están representadas por números reales
y la salida de la red puede ser un escalar o un vector dependiendo del numero de neuronas 
de salida.

La red mas simple es la red hacia adelante la cual divide sus neuronas en capas, todas las
neuronas de una capa envía su salida a cada neurona de la capa siguiente, estas usan lo que 
les enviaron como entradas para calcular otra salida y el proceso se repite hasta la ultima 
capa. El objetivo de la red neuronal es encontrar los pesos de cada neurona que minimicen el
error de la salida, para lo cual se usa el método de descenso de backpropagation. Existe el 
inconveniente de que las neuronas solo emiten resultados lineales y por lo tanto no importa 
cuantas capas tenga una red si la salida de cada neurona es lineal es como si solo se tuviera 
una capa, por lo que a la salida de las neuronas se les aplica una función que se conoce como 
función de activación cuyo único objetivo es eliminar la linealidad en la salida de las 
neuronas. Para saber mas entra a:
[Deep Learning: Feedforward Neural Network](https://towardsdatascience.com/deep-learning-feedforward-neural-network-26a6705dbdc7).

<a title="Chrislb [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:MultiLayerNeuralNetwork_english.png#/media/File:MultiLayerNeuralNetworkBigger_english.png">
    <img width="512" alt="MultiLayerNeuralNetwork english" src="https://upload.wikimedia.org/wikipedia/commons/c/c2/MultiLayerNeuralNetworkBigger_english.png">
</a>

## Redes convolucionales

En las redes convolucionales no todas las neuronas de una capa se conectan con las de la
capa siguiente, en cambio se usan capas convolucionales que son capas que tienen un conjunto
de filtros a los que se les llama kernel, que no son mas que matrices, los cuales se aplican 
a toda la imagen por regiones que tengan la misma dimensión que el kernel, para calcular el
resultado se cada elemento del kernel con cada elemento de la región deseada y luego se suma
todo (como un producto punto vectorial).

El kernel inicia en la esquina superior izquierda de la imagen y termina en la esquina inferior
derecha recorriendo la imagen de izquierda a derecha y de arriba a abajo, cada resultado de 
aplicar el kernel en una cierta región de la imagen se guarda en otra matriz a la que se le 
conoce como mapa de características.

<a title="Aphex34 [CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Typical_cnn.png">
    <img width="512" alt="Typical cnn" src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/63/Typical_cnn.png/512px-Typical_cnn.png">
</a>

Cada capa de la red es un conjunto de filtros que se aplican a la salida de la capa anterior, así hasta
que se llega a una capa densamente conectada que hace la clasificación con las características que
los filtros lograron obtener. En el caso de las redes para reconocer imágenes el resultado es un 
vector cuya dimensión es el numero de clases diferentes a reconocer. Para conocer mas entra a: [A Comprehensive Guide to Convolutional Neural Networks](https://towardsdatascience.com/a-comprehensive-guide-to-convolutional-neural-networks-the-eli5-way-3bd2b1164a53).


## Transferencia de estilo

La transferencia de estilo consisten en tomar una imagen a la que llamaremos de referencia y otra
a la que llamaremos de contenido y modificar la imagen de contenido tratando de que se parezca
a la de referencia sin que se pierda el contenido original.

¿Cómo podemos entonces usar una red neuronal entrenada en reconocimiento de imágenes para transferir
estilo? ¿No se supone que una red que clasifica imágenes solo nos dice a que clase pertenece dicha
imagen? Es verdad que la red solo sirve para clasificación, pero lo que a nosotros nos interesa es
poder separar el estilo del contenido y poder establecer un criterio de como medir el error en cuanto
a lo parecidas que son dos imágenes en contenido y otro para medir lo parecido que son en estilo, así
podemos iniciar con una imagen aleatoria del tamaño de la imagen de contenido y medir que tanto se
parece en estilo al imagen de referencia y que tanto se parece el contenido a la imagen de contenido y
después minimizar ese error.

Las redes entrenadas para el reconocimiento de imágenes guardan en los mapas de características
información valiosa acerca del contenido de la imagen, en las primeras capas se guardan estructuras
simples que formal los pixeles, como bordes, líneas, esquinas etc. cada capa es mas abstracta que
la anterior y por lo tanto podemos encontrar figuras mas abstractas y que tienen mas que ver con
el contenido de la imagen, por lo que para saber que tan parecidas son dos imágenes en contenido
hace falta comparar los mapas de características de estas dos imágenes al pasar por la red neuronal.

Para calcular el error del estilo usaremos la correlación entre las activaciones de las neuronas
de la red, que se obtiene calculando la matriz de Gram de los mapas de características de las dos
imágenes en cada capa y obteniendo su distancia.

El error total lo calculamos como el error de contenido mas el del estilo, después usaremos
este error con el método de backpropagation para actualizar la imagen inicial. Para conocer los
detalles matemáticos detrás de este método puedes revisar el [articulo original de Leon A. Gatys](https://arxiv.org/abs/1508.06576)

## Fotorrealismo

El método anterior tiene buenos resultados para trasferir el estilo artístico, ya que si bien
no se pierde el contenido original de la imagen al realizar el proceso si se trata de una fotografía
se puede perder el realismo porque el método puede distorsionar la foto y hacer que parezca una 
pintura o que no simplemente luzca editada con algún programa de edición de fotos como se puede
ver en el ejemplo de abajo.

| Contenido | Referencia | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="contenido" href="https://ricardoamata.github.io/images/in7.png"><img src="https://ricardoamata.github.io/images/in7.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/tar1.png" alt="referencia"><img src="https://ricardoamata.github.io/images/tar1.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/bad_styled.png"><img src="https://ricardoamata.github.io/images/bad_styled.png" alt="resultado"></a> |

Principalmente se pueden encontrar dos problemas: uno es la distorsión de la imagen y el otro
es la transferencia de estilo sin sentido. El primer problema es fácil de entender, pero el segundo
pede sonar un poco extraño, lo de la trasferencia de estilo sin sentido se refiere a que el estilo
de una sección de la imagen de referencia se transfiera a una región de la imagen de contenido que
no debería (Ej. que el estilo de un edificio se transfiera al cielo).

Para resolver el primer problema partimos de la idea de que la imagen de entrada ya es una imagen
fotorrealista y solo hace falta agregar un termino extra a la función de error que
penaliza las distorsiones en la imagen, para lo que usaremos una función que nos mida la afinidad
de los colores de los pixeles de la imagen de salida en relación a los de la imagen de entrada. Justo
esto es lo que se consigue al calcular la matriz de Matting Laplacian de una imagen que representa
en una escala de grises mate la combinación local afín de los canales RGB de una imagen [R],
entonces podemos usar esta matriz para calcular que tan distorsionada queda la imagen de salida
y a este termino le llamaremos factor de fotorrealismo.

El calcular la matriz de Gram sobre toda la imagen de estilo y toda la imagen de salida para medir
la diferencia que hay entre los estilos causa que no se realice una transferencia tomando en cuenta
el contexto de cada imagen. Para resolver este problema es necesario agregar una mascara de 
segmentación a las imágenes de estilo y contenido, en donde se divide la imagen por secciones
y así limitar la transferencia de estilo solo con secciones que tengan que ver, por ejemplo
que si en la imagen de contenido hay un edificio, a este solo se le traspase el estilo de otro
edificio en la imagen de referencia.

| Imagen | Segmentación |
|:------------------------------:|:------------------------------:|
| <a title="NoImageYet" href="https://ricardoamata.github.io/images/NoImage.png"><img src="https://ricardoamata.github.io/images/NoImage.png" alt="NoImageYet"></a> | <a title="NoImageYet" href="https://ricardoamata.github.io/images/NoImage.png"><img src="https://ricardoamata.github.io/images/NoImage.png" alt="contenido"></a> |

Con esto ya tenemos un nuevo método de transferencia de estilo con el cual ya no sufriremos por
deformaciones en la imagen de salida o que el estilo se transfiera sin tener en cuenta el contexto
de la imagen. Si quieres conocer todos los detalles matemáticos detrás del método revisa el articulo [Deep photo style transfer](https://arxiv.org/abs/1703.07511).

Ejemplo usando el método descrito para transferencia fotorrealista con las imágenes antes mostradas:


| Contenido | Referencia | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="contenido" href="https://ricardoamata.github.io/images/in7.png"><img src="https://ricardoamata.github.io/images/in7.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/tar1.png"><img src="https://ricardoamata.github.io/images/tar1.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/good_styled.png"><img src="https://ricardoamata.github.io/images/good_styled.png" alt="resultado"></a> |

## El peso del estilo

Nuestra función de error esta compuesta por tres términos, los cuales son: la diferencia
de contenido entre la imagen de salida y la de contenido, la diferencia del estilo entre la
imagen de salida y la de referencia y el factor de fotorrealismo, si queremos que
a la imagen de salida no le importen mucho las distorsiones o que lo que sea mas importante sea
el estilo, entonces podríamos multiplicar el factor de estilo por un numero para incrementarlo
y que así tenga mas peso en la función y sea mas relevante a la hora de minimizar. Abajo podemos
ver un ejemplo con diferentes pesos para el estilo con una misma imagen de contenido y referencia
y el peso del fotorrealismo sin variar.

| Contenido | Referencia |
|:--------------------------:|:------------------------------:|
| <a title="Contetnido" href="https://ricardoamata.github.io/images/in_lcc.jpg"><img src="https://ricardoamata.github.io/images/in_lcc.jpg" alt="Contenido"></a> | <a title="Referencia" href="https://ricardoamata.github.io/images/style_pasillo.jpg"><img src="https://ricardoamata.github.io/images/style_pasillo.jpg" alt="Referencia"></a> |

Resultados:

| 100 | 500 |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o0.png"><img src="https://ricardoamata.github.io/images/pasillo_o0.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o1.png"><img src="https://ricardoamata.github.io/images/pasillo_o1.png" alt="Resultado"></a> |
                                                                          
| 1000 | 5000 |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o2.png"><img src="https://ricardoamata.github.io/images/pasillo_o2.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o3.png"><img src="https://ricardoamata.github.io/images/pasillo_o3.png" alt="Resultado"></a> |

| 10000 |
|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o4.png"><img src="https://ricardoamata.github.io/images/pasillo_o4.png" alt="Resultado"></a> |

## El peso del fotorrealismo

Si por el contrario queremos una imagen que sea mas realista podemos aumentar el peso por el que
multiplicamos el factor fotorrealista ejemplo:

| Contenido | Referencia |
|:--------------------------:|:------------------------------:|
| <a title="Contetnido" href="https://ricardoamata.github.io/images/in8.jpg"><img src="https://ricardoamata.github.io/images/in8.jpg" alt="Contenido"></a> | <a title="Referencia" href="https://ricardoamata.github.io/images/style8.png"><img src="https://ricardoamata.github.io/images/style8.png" alt="Referencia"></a> |
                                                                
| 100 | 1000 |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls0.png"><img src="https://ricardoamata.github.io/images/output_igls0.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls1.png"><img src="https://ricardoamata.github.io/images/output_igls1.png" alt="Resultado"></a> |
                                                                          
| 10<sup>4</sup> | 10<sup>5</sup> |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls2.png"><img src="https://ricardoamata.github.io/images/output_igls2.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls3.png"><img src="https://ricardoamata.github.io/images/output_igls3.png" alt="Resultado"></a> |

                                                                           
| 10<sup>6</sup> | 10<sup>7</sup> |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls4.png"><img src="https://ricardoamata.github.io/images/output_igls4.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/output_igls5.png"><img src="https://ricardoamata.github.io/images/output_igls5.png" alt="Resultado"></a> |

## Como influye la segmentación

Como ya se dijo, se usa una mascara de segmentación para poder transferir el estilo
de la imagen de manera "inteligente", por lo el resultado final dependerá mucho de la
segmentación que se utilice.

A continuación se muestra un ejemplo usando dos segmentaciones distintas con una imagen
de contenido antes mostrada.

| Contenido | Referencia |
|:--------------------------:|:------------------------------:|
| <a title="Contetnido" href="https://ricardoamata.github.io/images/in_lcc.jpg"><img src="https://ricardoamata.github.io/images/in_lcc.jpg" alt="Contenido"></a> | <a title="Referencia" href="https://ricardoamata.github.io/images/style_pasillo.jpg"><img src="https://ricardoamata.github.io/images/style_pasillo.jpg" alt="Referencia"></a> |
       
| seg contenido | seg estilo | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="Segmentacion del contenido" href="https://ricardoamata.github.io/images/in_lcc_seg.jpg"><img src="https://ricardoamata.github.io/images/in_lcc_seg.jpg" alt="Segmentacion del contenido"></a> | <a title="Segmentacion del estilo" href="https://ricardoamata.github.io/images/style_lcc_seg.jpg"><img src="https://ricardoamata.github.io/images/style_lcc_seg.jpg" alt="Segmentacion del estilo"></a> | <a title="resultado" href="https://ricardoamata.github.io/images/pasillo_o2.png"><img src="https://ricardoamata.github.io/images/pasillo_o2.png" alt="resultado"></a> |
                                                                                       
| seg contenido | seg estilo | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="Segmentacion del contenido" href="https://ricardoamata.github.io/images/in_lcc_seg2.jpg"><img src="https://ricardoamata.github.io/images/in_lcc_seg2.jpg" alt="Segmentacion del contenido"></a> | <a title="Segmentacion del estilo" href="https://ricardoamata.github.io/images/style_lcc_seg2.jpg"><img src="https://ricardoamata.github.io/images/style_lcc_seg2.jpg" alt="Segmentacion del estilo"></a> | <a title="resultado" href="https://ricardoamata.github.io/images/output_seg.png"><img src="https://ricardoamata.github.io/images/output_seg.png" alt="resultado"></a> |



## Como mejorar el resultado obtenido

Para mejorar el resultado final es posible aplicar un smoothing a la imagen, por ejemplo
se podría pasar un filtro bilineal para eliminar los cambios bruscos de color sin perder
claridad de la imagen, también es bueno experimentar con diferentes valores del estilo y
el fotorrealismo, ya que para una imagen podrían dar buenos resultados pero para otra no.

## Conclusiones

La transferencia de estilo fotorrealista es un resultado posible gracias a las redes
neuronales para el reconocimiento de imágenes, sin embargo es un método costoso
computacionalmente hablando, todas las pruebas que mostramos fueron hechas en usando
una tarjeta grafica nvidia tesla, el tiempo varia dependiendo de los tamaños de las
imágenes de entrada. Si quieres el la implementación usando tensorflow el repositorio
de github es [este](https://github.com/ricardoamata/deep-photo-styletransfer-tf) que
es un fork del repositorio de Louie Yang, es recomendable tener una buena tarjeta grafica
para ejecutar este programa, o correrlo desde alguna hpc.

En este blog nos basamos en el articulo de [Fujun Luan](https://arxiv.org/abs/1703.07511)
donde si bien el algoritmo que propone es eficiente con el tiempo ha surgido nuevos
artículos que tratan de mejorar el método de Fujun y arreglar ciertos fallos que se puede
encontrar en las imágenes finales.
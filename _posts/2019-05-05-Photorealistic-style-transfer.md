---
layout: single
title: Photorealistic style transfer
read_time: false

---

<a title="Hermosillo, sonora" href="https://ricardoamata.github.io/images/output_1.png">
    <img src="https://ricardoamata.github.io/images/output_1.png" alt="Hermosillo, sonora">
</a>


# *****
# Nota importante: No se ha revisado la ortografía del post.
# *****



## Introducción

En este blog se explicara en que consiste la transferencia de estilo y como conseguir
que esta sea fotorealista, se mostrara de que manera se puede usar una red neuronal
pre entrenada en reconociemiento de imagenes para alcanzar este resultado, asi como
las mejores estrategias a seguir, ya que el proceso no es una receta de cocina la 
cual seguir al pie de la letra y depende mucho de las entradas. Al final mostraremos
algunos metodos de post procesamiento y algunas formas de mejorar los resultados obtenidos
en base a nuevas ideas propuestas recientemente.

## Red neuronal

Una red neuronal se puede ver como una coleccion de nodos conectados entre si a los que se
les llama neuronas, cada neurona recive una serie de señales que son multiplicadas cada una
por un peso y en base a eso emite una señal de salida, asi que una neurona solamente calcula
una convinacion lineal de las señales de entrada y al resultado se le suma un sesgo, por lo
tanto no son mas que aproximadores lineales. Las señales estan representadas por numeros reales
y la salida de la red puede ser un escalar o un vector dependiendo del numero de neuronas 
de salida.

La red mas simple es la red hacia adelante la cual divide sus neuronas en capas, todas las
neuronas de una capa envia su salida a cada neurona de la capa siguiente, estas usan lo que 
les enviaron como entradas para calcular otra salida y el proceso se repite hasta la ultima 
capa. El objetivo de la red neuronal es encontrar los pesos de cada neurona que minimicen el
error de la salida, para lo cual se usa el metodo de desenso de backpropagation. Existe el 
inconveninete de que las neuronas solo emiten resultados lineles y por lo tanto no importa 
cuantas capas tenga una red si la salida de cada neurona es lineal es como si solo se tuviera 
una capa, por lo que a la salida de las neuronas se les aplica una funcion que se conoce como 
funcion de activacion cuyo unico objetivo es eliminar la linealidad en la salida de las 
neuronas. Para saber mas entra a:
[Deep Learning: Feedforward Neural Network](https://towardsdatascience.com/deep-learning-feedforward-neural-network-26a6705dbdc7).

<a title="Chrislb [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:MultiLayerNeuralNetwork_english.png#/media/File:MultiLayerNeuralNetworkBigger_english.png">
    <img width="512" alt="MultiLayerNeuralNetwork english" src="https://upload.wikimedia.org/wikipedia/commons/c/c2/MultiLayerNeuralNetworkBigger_english.png">
</a>

## Redes convolucionales

En las redes convolucionales no todoas las neuronas de una capa se conectan con las de la
capa siguiente, en cambio se usan capas convolucionales que son capas que tienen un conjunto
de filtros a los que se les llama kernel, que no son mas que matrices, los cuales se aplican 
a toda la imagen por regiones que tengan la misma dimension que el kerne, para calcular el
resultado se cada elemento del kernel con cada elemento de la region deseada y luego se suma
todo (como un producto punto vectorial).

El kernel inica en la esquina superior izquierda de la imagen y termina en la esquina inferior derecha
recoriendo la imagen de izquierda a derecha y de arriba a abajo, cada resultado de aplicar el kernel
en una cierta region de la imagen se guarda en otra matriz a la que se le conoce como mapa de caracteristicas.

<a title="Aphex34 [CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Typical_cnn.png">
    <img width="512" alt="Typical cnn" src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/63/Typical_cnn.png/512px-Typical_cnn.png">
</a>

Cada capa de la red es un conjunto de filtros que se aplican a la salida de la capa anterior, asi hasta
que se llega a una capa densamente conectada que hace la clasificacion con las caracteristicas que
los filtros lograron obtener. En el caso de las redes para reconocer imagenes el resultado es un 
vector cuya dimension es el numero de clases diferentes a reconocer. Para conocer mas entra a: [A Comprehensive Guide to Convolutional Neural Networks](https://towardsdatascience.com/a-comprehensive-guide-to-convolutional-neural-networks-the-eli5-way-3bd2b1164a53).


## Transferencia de estilo

La transferencia de estilo consisten en tomar una imagen a la que llamaremos de referencia y otra
a la que llamaremos de contenido y modificar la imagen de contenido tratando de que se paresca
a la de referencia sin que se pierda el contenido original.

¿Como podemos entonces usar una red neronal entrenada en reconocimieto de imagenes para transferir
estilo? ¿No see supono que una red que clasifica imagenes solo nos dice a que clase pertenece dicha
imagen? Es verdad que la red solo sirve para clasificacion, pero lo que a nosotros nos interesa es
poder separar el estilo del contenido y poder establecer un criterio de como medir el error en cuanto
a lo parecidas que son dos imagenes en contenido y otro para medir lo parecido que son en estilo, asi
podemos iniciar con una imagen aleatoria del tamaño de la imagen de contenido y medir que tanto se
parece en estilo al imagen de referencia y que tanto se parece el contenido a la imagen de contenido y
despues minimizar ese error.

Las redes entrenadas para el reconocimiento de imagenes guardan en los mapas de caracteristicas
informacion valiosa acerca del contenido de la imagen, en las primeras capas se guardan estructuras
simples que formal los pixeles, como bordes, lineas, esquinas etc. cada capa es mas abstracta que
la anterior y por lo tanto podemos encontrar figuras mas abstractas y que tienen mas que ver con
el contenido de la imagen, por lo que para saber que tan parecidas son dos imagenes en contenido
hace falta comparar los mapas de caracteristicas de estas dos imagenes al pasar por la red neuronal.

Para calcular el error del estilo usaremos la correlacion entre las activaciones de las neuronas
de la red, que se obtien calculando la matriz de Gram de los mapas de caracteristicas de las dos
imagenes en cada capa y obteninedo su distancia.

El error total lo calculamos como el error de contenido mas el del estilo, despues usaremos
este error con el metodo de backpropagation para actualizar la imagen inical. Para conocer los
detalles matematicos detras de este metodo puedes revisar el [articulo original de Leon A. Gatys](https://arxiv.org/abs/1508.06576)

## Fotorealismo

El metodo anterior tiene buenos resultados para trasferir el estilo artistico, ya que si bien
no se pierde el contenido original de la imagen al realizar el proceso si se trata de una fotografia
se puede perder el realismo porque el metodo puede distorsionar la foto y hacer que paresca una 
pintura o que no simplemente lusca editada con algun programa de edicion de fotos como se puede
ver en el ejemplo de abajo.

| Contenido | Referencia | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="contenido" href="https://ricardoamata.github.io/images/in7.png"><img src="https://ricardoamata.github.io/images/in7.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/tar1.png" alt="referencia"><img src="https://ricardoamata.github.io/images/tar1.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/bad_styled.png"><img src="https://ricardoamata.github.io/images/bad_styled.png" alt="resultado"></a> |

Principalmente se pueden encontrar dos problemas: uno es la distorcion de la imagen y el otro
es la transferencia de estilo sin sentido. El primer problema es facil de entender, pero el segundo
pede sonar un poco extraño, lo de la trasferencia de estilo sin sentido se refiere a que el estilo
de una seccion de la imagen de referencia se transfiera a una region de la imagen de contenido que
no deberia (ej. que el estilo de un edificio se transfiera al cielo).

Para resolver el primer porblema partimos de la idea de que la imagen de entrada ya es una imagen
fotorealista y solo hace falta agregar un termino extra a la funcion de error que
penaliza las distorciones en la imagen, para lo que usaremos una funcion que nos mida la affinidad
de los colores de los pixeles de la imagen de salida en relacion a los de la imagen de entrada. Justo
esto es lo que se consigue al acalcular la matriz de Matting Laplacian de una imagen que representa
en una escala de grisces mate la combinacion local afín de los canales RGB de una imagen [R],
entonces podemos usar esta matriz para calcular que tan distorcionada queda la imagen de salida
y a este termino le llamaremos factor de fotorrealismo.

El calcular la matriz de Gram sobre toda la imagen de estilo y toda la imagen de salida para medir
la diferencia que hay entre los estilos causa que no se realize una ransferencia tomando en cuenta
el contexto de cada imagen. Para resolver este problema es necesario agregar una mascara de 
segmentacion a las imagenes de estilo y contenido, en donde se divide la imagen por secciones
y asi limitar la transferencia de estilo solo con secciones que tengan que ver, por ejemplo
que si en la imagen de contenido hay un edificio, a este solo se le traspase el estilo de otro
edificio en la imagen de referencia.

| Imagen | Segmentación |
|:------------------------------:|:------------------------------:|
| <a title="NoImageYet" href="https://ricardoamata.github.io/images/NoImage.png"><img src="https://ricardoamata.github.io/images/NoImage.png" alt="NoImageYet"></a> | <a title="NoImageYet" href="https://ricardoamata.github.io/images/NoImage.png"><img src="https://ricardoamata.github.io/images/NoImage.png" alt="contenido"></a> |

** Futura descripcion de las imagenes de arriba **

Con esto ya tenemos un nuevo metodo de transferencia de estilo con el cual ya no sufiremos por
deformaciones en la imagen de salida o que el estilo se transfiera sin tener en cuenta el contexto
de la imagen. Si quieres conocer todos los detalles matematicos detras del metodo revisa el articulo [Deep photo style transfer](https://arxiv.org/abs/1703.07511).

Ejemplo usando el metodo descrito para transferencia fotorealista con las imagenes antes mostradas:


| Contenido | Referencia | Resultado |
|:--------------------:|:--------------------:|:--------------------:|
| <a title="contenido" href="https://ricardoamata.github.io/images/in7.png"><img src="https://ricardoamata.github.io/images/in7.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/tar1.png" alt="referencia"><img src="https://ricardoamata.github.io/images/tar1.png" alt="contenido"></a> | <a title="contenido" href="https://ricardoamata.github.io/images/good_styled.png"><img src="https://ricardoamata.github.io/images/good_styled.png" alt="resultado"></a> |

## El peso del fotorrealismo

Nuestra funcion de error esta compuesta por tres terminos, los cuales son: la diferencia
de contenido entre la imagen de salida y la de contenido, la diferencia del estilo entre la
imagen de salida y la de referencia y el factor de fotorrealismo, si queremos que
a la imagen de salida no le importen mucho las distorciones o que lo que sea mas importante sea
el estilo, entonces podriamos multiplicar el factor de estilo por un numero para incrementarlo
y que asi tenga mas peso en la funcion y sea mas relevante a la hora de minimizar. Abajo podemos
ver un ejemplo con diferentes pesos para el estio con una misma imagen de contenido y referencia
y el peso del fotorealismo sin variar.

| Contenido | Referencia |
|:--------------------------:|:------------------------------:|
| <a title="Contetnido" href="https://ricardoamata.github.io/images/in_lcc.jpg"><img src="https://ricardoamata.github.io/images/in_lcc.jpg" alt="Contenido"></a> | <a title="Referencia" href="https://ricardoamata.github.io/images/style_pasillo.jpg"><img src="https://ricardoamata.github.io/images/style_pasillo.jpg" alt="Referencia"></a> |

Resultados:

| 100 | 500 |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o1.png"><img src="https://ricardoamata.github.io/images/pasillo_o1.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o2.png"><img src="https://ricardoamata.github.io/images/pasillo_o2.png" alt="Resultado"></a> |
| 1000 | 5000 |
|:------------------------------:|:------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o3.png"><img src="https://ricardoamata.github.io/images/pasillo_o3.png" alt="Resultado"></a> | <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o4.png"><img src="https://ricardoamata.github.io/images/pasillo_o4.png" alt="Resultado"></a> |

| 10000 |
|:-------------------------------------------------------:|
| <a title="Resultado" href="https://ricardoamata.github.io/images/pasillo_o5.png"><img src="https://ricardoamata.github.io/images/pasillo_o5.png" alt="Resultado"></a> |

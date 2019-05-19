---
layout: single
title: Photorealistic style transfer
read_time: false

---

<a title="Hermosillo, sonora" href="https://ricardoamata.github.io/images/output_1.png">
    <img src="https://ricardoamata.github.io/images/output_1.png" alt="Hermosillo, sonora">
</a>

## Introdución

En este blog se explicara en que consiste la transferencia de estilo y como conseguir
que esta sea fotorealista, se mostrara de que manera se puede usar una red neuronal
pre entrenada en reconociemiento de imagenes para alcanzar este resultado, asi como
las mejores estrategias a seguir, ya que el proceso no es una receta de cocina la 
cual seguir al pie de la letra y depende mucho de las entradas. Al final mostraremos
algunos metodos de post procesamiento y algunas formas de mejorar los resultados obtenidos
en base a nuevas ideas propuestas recientemente.

## Red neuronal

Antes de empezar a entender el metodo de transferencia de estilo primero hay que 
conocer que son las redes neuronales y en particular las redes convolucionales, que
son la parte principal de todo el proceso.

Podemos imaginar a una red neuronal como una caja negra a la cual se le introducen
un conjunto de datos y esta emite un resultado, por lo que podriamos decir que se 
trata de una funcion que es desconocida, la cual fue aproximada por la red usando
algun metodo de aprendizaje automatico.

La red mas simple es la red hacia adelante la cual es en si un conjuto de neuronas 
divididas en capas donde todas las neuronas de una capa se conectan con todas las 
neuronas de la capa siguiente. Una neurona no es mas que un regresor lineal que toma
los valores proporcionados por las neuronas de la capa anterior, multiplica cada valor
por un peso y luego los suma y al resultado le aplica pasa por una funcion y lo que
se optiene se le conoce como activacion de la neurona, la cual se manda a las neuronas
de la capa siguiente y el proceso se repite hasta la ultima capa donde se obtiene una
salida. Si quieres saber mas puedes revisar este blog:
[Deep Learning: Feedforward Neural Network](https://towardsdatascience.com/deep-learning-feedforward-neural-network-26a6705dbdc7).

<a title="Chrislb [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:MultiLayerNeuralNetwork_english.png#/media/File:MultiLayerNeuralNetworkBigger_english.png">
    <img width="512" alt="MultiLayerNeuralNetwork english" src="https://upload.wikimedia.org/wikipedia/commons/c/c2/MultiLayerNeuralNetworkBigger_english.png">
</a>

## Redes convolucionales

En el caso de las redes convolucionales no todas las neuronas de una capa se conectan a las de la siguiente
al menos no para las neuronas convolucionales, las cuales estan inspiradas por como el cerebro reconoce
imagenes, cuando el cerebro percibe un estimulo visual solo ciertas neuronas se activan con ciertas regiones
de la imagen captada, para lograr este objetivo se utilizan filtros convolucionales, los cuales reducen el
tamaño de la imagen de entrada, por cada capa que se aplica se reduce mas y se conservan las caracteristicas
representativas de la imagen.

Una imagen no es mas que una matriz de numeros que van de 0 a 255 en tres canales, un filtro o kerne
tambien es una matriz que se aplica a una cierta region de la imagen tomando una submatriz que tiene
las mismas dimensiones del kernel y luego se aplica un producto punto entre las dos como si fueran 
vectores.

El kernel inica en la esquina superior izquierda de la imagen y termina en la esquina inferior derecha
recoriendo la imagen de izquierda a derecha y de arriba a abajo, cada resultado de aplicar el kernel
en una cierta region de la imagen se guarda en otra matriz a la que se le conoce como mapa de caracteristicas.

<a title="Aphex34 [CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Typical_cnn.png">
    <img width="512" alt="Typical cnn" src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/63/Typical_cnn.png/512px-Typical_cnn.png">
</a>

Cada capa de la red es un conjunto de filtros que se aplican a la salida de la capa anterior, asi hasta
que se llega a una capa densamente conectada que hace la clasificacion con las caracteristicas que
los filtros lograron obtener. Para conocer mas ve a: [A Comprehensive Guide to Convolutional Neural Networks](https://towardsdatascience.com/a-comprehensive-guide-to-convolutional-neural-networks-the-eli5-way-3bd2b1164a53).


## Transferencia de estilo





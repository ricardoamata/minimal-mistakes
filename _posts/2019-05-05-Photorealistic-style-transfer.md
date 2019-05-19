---
layout: single
title: Photorealistic style transfer
read_time: false

---

![Ciudad de Hermosillo](/images/output_1.png)

## Introdución

En este blog se explicara en que consiste la transferencia de estilo y como conseguir
que esta sea fotorealista, se mostrara de que manera se puede usar una red neuronal
pre entrenada en reconociemiento de imagenes para alcanzar este resultado, asi como
las mejores estrategias a seguir, ya que el proceso no es una receta de cocina la 
cual seguir al pie de la letra y depende mucho de las entradas. Al final mostraremos
algunos metodos de post procesamiento y algunas formas de mejorar los resultados obtenidos
en base a nuevas ideas propuestas recientemente.

## Redes convolucionales

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
salida. Si quieres saber mas puedes revisar el blog
[Deep Learning: Feedforward Neural Network](https://towardsdatascience.com/deep-learning-feedforward-neural-network-26a6705dbdc7).

## Convolucion

La convolucion es...



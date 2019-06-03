---
layout: single
title: Tradución automatica usando atención
read_time: false

---

En este blog se mostrara en que consiste el modelo encoder-decoder, que es la
capa de atención. Se dará una pequeña introducción a los temas necesarios y por
ultimo explicaremos como es posible dicho modelo para traducción entre lenguajes,
así como mostrar algunos resultados obtenidos.

## Redes recurrentes

Las redes neuronales recurrentes se han usado en los últimos años para lograr buenos
resultados en el área de traducción automática, es por eso que en este blog empezaremos
dando una breve introducción sobre este tipo de redes.

Las redes convolucionales son un tipo de red inspiradas en la capacidad de memoria que
tienen las neuronas del cerebro, con la finalidad de ser efectivas a la hora de analizar
datos que cambian con el tiempo. En estas redes las salidas de una capa no solo se conectan
con la siguiente, sino que se vuelven a conectar a la misma capa, de ahí el nombre de 
redes recurrentes.

<a title="François Deloche [CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Recurrent_neural_network_unfold.svg"><img width="600" alt="Recurrent neural network unfold" src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b5/Recurrent_neural_network_unfold.svg/1024px-Recurrent_neural_network_unfold.svg.png"></a>

La imagen de arriba muestra una red que tiene una sola capa con una entrada x<SUB>t</SUB> que 
es la entrada en el tiempo t, y produce una salida O<SUB>t</SUB>, luego esa misma salida 
vuelve a entrar a la capa junto con la entrada x<SUB>t+1</SUB>. Si desenrollamos una capa en el 
tiempo es como si tuviéramos una red con muchas capas y una salida por capa. Cada capa se le llama
unidad, existen diferentes tipos de unidades.

## Unidad LSTM

Cada capa consta de una unidad [LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory) 
(Long Short Term Memory) la cual esta diseñada para tratar uno de los problemas de las redes
recurrentes, que es el [desvanecimiento del gradiente](https://en.wikipedia.org/wiki/Vanishing_gradient_problem).

<a title="Guillaume Chevalier [CC BY 4.0 (https://creativecommons.org/licenses/by/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:The_LSTM_cell.png"><img width="600" alt="The LSTM cell" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/The_LSTM_cell.png/1024px-The_LSTM_cell.png"></a>

Una unidad LSTM esta compuesta por una celda y tres reguladores, conocidos como puertas que son:
la puerta de entrada (**input gate**), la puerta de salida (**output gate**), y la puerta de olvido
(**forget gate**). Cada puerta es una capa de una red neuronal normal, con una función de activación
logística.

La primer puerta por la que pasa la entrada una ves es concatenada con la salida anterior
es la puerta de olvido y después de haber obtenido una salida multiplica su resultado con el estado 
interno anterior de la celda C<SUB>t-1</SUB>. Esta puerta decide que datos de entrada son importantes 
y que datos no, emite valores entre 0 y 1 de modo  que 0 significa que un dato no merece ser recordado 
y 1 significa que el dato tiene que ser recordado exactamente.

Luego es turno de la puerta de entrada, en la cual se multiplica el resultado por la salida de otra 
capa, pero con función de activación tanh, luego se suma el resultado con el estado anterior 
C<SUB>t-1</SUB> para obtener nuestro nuevo estad interno C<SUB>t</SUB>.

Por ultimo es turno de la capa de salida filtrar aquellos valores de nuestro estado que no sean
necesarios, para ello calculamos el valor de C<SUB>t-1</SUB> al pasar por la función tanh y lo
multiplicamos por la salida de la puerta de salida.

# Unidad GRU

las unidades GRU (Gated Recurrent Unit) son una variante de las LSTM que solo usa dos puertas, 
la **update gate** y la **reset gate**, además de que solamente tiene un estado interno, la 
primera se encarga de decidir que tanta información de
los pasos anteriores requiere ser preservada, y la otra que tanta información es necesaria olvidar.
Para la implementación se usaron GRU por que son mas fáciles de entrenar.

<a title="Jeblad [CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Gated_Recurrent_Unit,_base_type.svg"><img width="512" alt="Gated Recurrent Unit, base type" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Gated_Recurrent_Unit%2C_base_type.svg/512px-Gated_Recurrent_Unit%2C_base_type.svg.png"></a>

## Sequence to sequence

En el modelo sequence to sequence se utilizan dos redes recurrentes, una de ellas se llama encoder
y es en donde entra nuestra secuencia de palabras, dicha red se entrena para producir un vector de
tamaño fijo que represente nuestra secuencia de entrada, ese vector es desconocido y es enviado
a la segunda red, que a la que se le conoce como decoder, la cual se encarga de producir una secuencia
de salida en base al vector que se le proporciono.

En el caso de la traducción automática tendríamos un texto de entrada en nuestro lenguaje fuente
y el encoder se encargaría de transformarlo en una representación abstracta, luego le pasaría es
representación al decoder el cual lo transformaría en otro texto que sea una traducción del texto
original. La red es entrenada de modo que se escoja la salida que sea mas probable de acuerdo a la
entrada.

[![encoder-decoder](https://cdn-images-1.medium.com/max/800/1*1JcHGUU7rFgtXC_mydUA_Q.jpeg)](https://cdn-images-1.medium.com/max/800/1*1JcHGUU7rFgtXC_mydUA_Q.jpeg)


La red solo acepta entradas numéricas, por lo que para poder introducir secuencias de caracteres
primero debemos convertirlas a entradas validas, para ello usamos una capa de embedding la cual
convierte un numero en un vector denso, esta campa se puede entrenar a parte o junto con la red
y sirve para representar tu entrada en forma vectorial, de manera que si quieres que cada elemento
de la secuencia sean palabras entonces los vectores de salida nos indicaran que tan relacionadas están
dos palabras dependiendo de la distancia. También puedes usar caracteres puros como entrada, pero es
probable que existan ocasiones en las que no obtengas palabras reales.

## Capa de atención

Uno de los problemas principales del modelo encoder decoder surge por tratar de codificar una
secuencia de un tamaño indefinido en un vector de tamaño constante, es posible que para secuencias
de una gran longitud se pierda información que es importante a la hora de hacer la codificación,
lo que repercute directamente en el resultado final.

La idea base del mecanismo de atención es brindar al decoder acceso a los demás estados del encoder
y no solo al estado final,
esto es inspirado en la forma en la que un ser humano procesaría la información de entra para
generar la salida, no memorizaría toda la entrada, en cambio solo memorizaría lo mas importante
y cuando necesite información extra ir a la fuente de información original.

La capa de atención es una capa densa cuyas entradas son todos los estados emitidos por
el encoder, tiene una activación softmax y a su salida es el vector de pesos por los que se
multiplicaran todos los estados emitidos por el encoder antes de ser enviados al decoder,
de modo que el decoder preste mas atención a los estados que sean mas relevantes (de ahí
el nombre de atención). La capa de atención estima las probabilidades de que una salida del
encoder sea significativa para el decoder en el estado actual, por esta razón si un estado es
muy relevante se espera que se multiplique por un peso mas grande que el de un estado que es
poco relevante.

Una vez multiplicado cada estado por su peso de atención, se suman y se envían al encoder
como el vector codificado de la entrada, por lo que se calcula un vector de contexto para
cada palabra que va a predecir el decoder, si lo graficamos nos daría algo como esto:

![Attention image](/images/attention.png)

## Implementación

La implementación simplemente sigue el tutorial de tensorflow [nmt_with_attention](https://github.com/tensorflow/tensorflow/blob/r1.13/tensorflow/contrib/eager/python/examples/nmt_with_attention/nmt_with_attention.ipynb), el repositorio de gihub de nuestra implementación es [NMT_using_attention](https://github.com/ricardoamata/NMT_using_attention)

los datos se obtuvieron de [http://www.manythings.org/anki/](http://www.manythings.org/anki/), proporciona
datos de frases en dos idiomas, cada línea tiene una frase en un idioma y luego la misma frase en otro
idioma separadas por una tabulación. El modelo se entreno para hacer traducciones de español a ingles.

Ejemplo de algunas traducciones de español a ingles en base al numero de datos de entrenamiento:

30000 datos aprendizaje:

* hace mucho frio aqui - It's too cold here

* Ya no hay comida - There's no food anymore

* Mi casa es de color azul - My house is from the blue

* Solo sé que no sé nada - I know i know what they don't know anything

* Mañana no trabajo - I'm not working tomorrow

60000 datos aprendizaje:

* hace mucho frio aqui - It's too cold here

* Ya no hay comida - There's no more left

* Mi casa es de color azul - My house is of blue eyes

* Solo sé que no sé nada - I simply know nothing

* Mañana no trabajo - I'm not work today

90000 datos aprendizaje:

* hace mucho frio aqui - It's too cold here

* Ya no hay comida - There's no food left

* Mi casa es de color azul - My house is full of japanese

* Solo sé que no sé nada - All i don't know it takes nothing 

* Mañana no trabajo - I'm not working tomorrow

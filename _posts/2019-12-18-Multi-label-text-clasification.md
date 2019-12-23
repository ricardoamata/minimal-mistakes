---
layout: single
title: Clasificación multi label y regresión usando textos
read_time: false

---

En este blog explicare como entrenar un modelo [svm](https://towardsdatascience.com/https-medium-com-pupalerushikesh-svm-f4b42800e989) para predecir a qué géneros pertenecen una película a partir de un texto que sirva como una descripción de la trama. También se entrena un modelo que usando esta misma descripción en texto prediga el año de salida de la película.

[Libreta con el código implementado](https://github.com/ricardoamata/Proyecto-ML/blob/master/Clasificaci%C3%B3n_de_pel%C3%ADculas_usando_las_tramas.ipynb)

### Descripción de los datos

Los datos son sacados de kaggle y se pueden descargar de aqui: [Wiki Movie Plots](https://www.kaggle.com/jrobischon/wikipedia-movie-plots), la base de datos contiene descripciones para 34,886 películas de todo el mundo. las columnas son las siguientes:

- Release Year - Año en que se estreno la película
- Title - Titulo de la película
- Origin/Ethnicity - Origen de la pelicula (e.g. America, Bollywood, Tamil, etc)
- Director - Director(s)
- Cast - Actores y actrices principales
- Genre - Genero(s) de la película
- Wiki Page - URL de la pagina de Wikipedia de la cual la descripción de la trama fue tomada
- Plot - descripción larga de la trama de la película (ADVERTENCIA: puede contener espoilers)

## Problemas específicos de este dataset que hay que solucionar

Los textos no están limpios: hay que remover todos caracteres raros como los puntos, comas, entre otros y reemplazar las contracciones que tienen los textos, ademas hay que realizar otras tareas de pln.

Los géneros no están en el formato correcto: en muchos casos nos encontramos con géneros que son iguales pero están mal escritos o escritos de otra forma, todos estos tenemos que cambiarlos por un único nombre, además tenemos que remover todos los géneros que sean demasiado raros y en algunos casos cambiarlos por otros más adecuados.

## El problema de tener múltiples clases de salida

El problema de clasificar un dato tal que pueda pertenecer a múltiples clases tiene varias soluciones, una de ellas sería transformar el conjunto de clases en su conjunto potencia, donde cada subconjunto forme una nueva clase, el problema de esto es que el número de nuevas clases sería demasiado grande si antes tenias n clases ahora tendrás 2<sup>n</sup> nuevas clases, lo que dificulta el entrenamiento, para contrarrestar esto se podría utilizar solo únicamente aquellas clases que aparecen en el conjunto de entrenamiento, esto claro aumentaría nuestro sesgo, ya que si un dato pertenece a un subconjunto de clases que no está presente en el conjunto de entrenamiento no lo podremos clasificar correctamente.

Otra alternativa y la cual usaremos en este blog es entrenar un clasificador binario por clase, de manera que cada clasificador estime la probabilidad de un dato de pertenecer a cada clase independientemente del resto de clasificadores, la ventaja es que si tenemos n clases solo necesitamos n clasificadores para poder mapear un cualquier dato a todo el conjunto potencia de las clases, lo malo es que el conjunto vacío también está incluido, esto es que puede que todos los clasificadores tengan una salida falsa y por lo tanto existan datos que no pertenezcan a ninguna clase, esto es problema solo si sabemos de entrada que todos los datos tienen que pertenecer a mínimo una clase

## La codificación de las entradas

Para poder meter nuestros textos a un algoritmo de aprendizaje es necesario buscar una representación matemática de estos, una forma seria bag of words, donde se convierte un texto a un vector en el cual  cada posición representa una palabra, pudiendo haber un uno o un cero dependiendo de si esa palabra se encuentra en nuestro texto. 

Un ejemplo sería considerando el siguiente vocabulario: {“hoy”, “día“, “yo”, “es”, “mañana”, “gran”, “vida”}

la representación vectorial de cada palabra es:

|Palabra | vector |
|:-------:|:---------:|
| hoy | 1,0,0,0,0,0,0,0 |
| día | 0,1,0,0,0,0,0,0 |
| yo | 0,0,1,0,0,0,0,0 |
| es | 0,0,0,1,0,0,0,0 |
| mañana | 0,0,0,0,1,0,0,0 |
| gran | 0,0,0,0,0,1,0,0 |
| vida | 0,0,0,0,0,0,1,0 |
| un | 0,0,0,0,0,0,0,1 |

Por lo que el texto “hoy es un gran día” es codificado de la siguiente manera:

1,1,0,1,0,1,0,1

La desventaja es que el vector es tan grande como nuestro vocabulario y que la mayoría de las entradas son ceros, otra manera de codificar el texto es usando tfidf en donde ya no solo se coloca un uno en caso de que la palabra esté presente sino que se cuenta el número de veces que la palabra apareció en el texto entre el número de documentos en los que aparece, de ahí el nombre Term Frequency Inverse Document Frequency. A pesar de que esta es una mejor manera de representar un texto aún tiene defectos, como que todavía tiene varias entradas en ceros o que no toma en cuenta el orden de las palabras, esto se puede resolver usando n gramas pero esto agrega más carga computacional.

Antes de codificar los textos hay que limpiarlos, primero, como nuestro texto tiene contracciones en el idioma inglés se reemplazan todas estas por la forma completa de la expresión y se eliminan todos los caracteres extraños como puntos, comas, signos de interrogación y de admiración entre otros. Luego se tokenizar el texto por palabras, estas se lematizan y se convierten a minúsculas.

## Limpieza de los generos

Ahora hay que limpiar los géneros, para esto se removieron todos los géneros repetidos que estaban escritos de otra manera o que estaban mal escritos.

Luego se cambió el formato de múltiples géneros, cambiando todas la formas que existían para representar que una pelicula pertenece a multiples generos, por una en donde se pone cada género separado por el carácter “\|”.

Hasta este momento ya tenemos los géneros limpios, pero si una película tiene múltiples géneros esto se sigue representando como una cadena, por lo que si queremos usar nuestros clasificadores binarios para cada clase lo que haremos será separar en una lista los géneros de cada película.

Ejemplo:

Si se tiene que una película es de los géneros “romance\|comedy” esto se considera un género distinto del genero “romance” o solo “comedy”, para evitar que las combinaciones de distintas clases formen otra clase totalmente distinta, vamos a separar estas en una lista, convirtiendo asi “romance\|comedy” a [“romance”, “comedy”].

Para representar esta lista se usará una representación de tipo one hot, pero con múltiples entradas, en donde la posición i-ésima del vector representa que la película pertenece al i-ésimo género, al final entrenaremos un clasificador binario para predecir si la i-ésima entrada es un uno o no.

## Aprendizaje

Para el aprendizaje se utilizó una máquina de vectores de soporte lineal por género, ya que estas generalizan bien, se utilizaron solo los 23 géneros con más ocurrencia, eliminando los otros, y se removieron los datos que no pertenecían a ninguno de estos.

Como una muestra de los resultados se eligieron 25 películas aleatorias del conjunto de pruebas y se realizó la clasificación sobre ellas:


| Péliculas | Predicción | Real |
|:-:|:-:|:-:|
| Kitty Kornered| animation | comedy |
| Head in the Clouds | thriller | drama |
| Brown Sugar | | comedy |
| Delavine Affair | crime | comedy\|romance |
| Red Salute | comedy | comedy |
| Take the Lead | drama | drama |
| Thalapathi | | drama |
| Vertigo | drama | comedy |
| Hochchheta ki| | crime |
| Guns of Darkness | | horror |
| Cat Napping | animation | drama |
| Girl with a Pearl Earring | drama | comedy |
| Ratatouille | | drama |
| Bareilly Ki Barfi | romance | drama |
| Fullmetal Alchemist the Movie: Conqueror of Sh... | fantasy | drama |
| Aakrosh | drama |
| A World Apart | drama | comedy\|crime |
| Child 44 | crime | musical |
| Steins;Gate: Fuka Ryōiki no Déjà vu | science_fiction | drama |
| The Gift of Love | drama | drama\|war |
| Maria's Lovers | | adventure |
| Beeba Boys | | musical |
| Alik Sukh | drama | drama |
| The Mysterious Mr. Valentine | drama\|crime | drama |
| Verboten! | | comedy |

#### Precisión de cada clasificador:

| Género | Precisión |
|:-:|:-:|
| drama | 0.719324 |
| comedy | 0.796415 |
| romance | 0.912416 |
| action | 0.923055 |
| thriller | 0.929612 |
| crime | 0.939959 |
| horror | 0.960799 |
| western | 0.984844 |
| musical | 0.966045 |
| science_fiction | 0.976537 |
| animation | 0.978723 |
| adventure | 0.971728 |
| family | 0.977703 |
| war | 0.981638 |
| fantasy | 0.982804 |
| mystery | 0.985281 |
| biography | 0.982221 |
| black | 0.986010 |
| history | 0.991110 |
| short | 0.991256 |
| martial_arts | 0.995337 |
| documentary | 0.994171 |
| sports | 0.995191 |

#### Presición promedio: 0.9528594781594922

#### Presición conjunta: 0.008

Incluso aun cuando tenemos una buena precisión por clase la precisión conjunta de todos los clasificadores no es buena, en nuestro caso estaría cercana a 0.08, esto debido a cómo funciona el método utilizado, la probabilidad de que clasifique bien una clase individualmente es alta, pero la probabilidad de que clasifique bien todas es el producto de las probabilidades individuales de cada clase, como todos son números menores a uno este producto termina siendo muy pequeño, por ejemplo.

En un caso hipotético supongamos que tenemos C clases distintas y que la probabilidad de clasificar buen una clase denotada como P es la misma para todas las clases, entonces la probabilidad de clasificar bien todas las clases es P<sup>C</sup>, aunque P sea un número muy grande como 0.95, si tenemos muchas clases P<sup>C</sup> terminará siendo un número muy pequeño, como es en nuestro caso.

## Regresión para predecir los años de lanzamiento

Se realizó una regresión lineal usando las tramas vectorizadas de las películas para intentar predecir los años en que habían sido lanzadas, estos fueron algunos de  los resultados:

| Pélicula | Predicción | Real |
|:-:|:-:|:-:|
| A Doll's House | 1976 | 1973 |
| Adventure of the King | 2020 | 2010 |
| The Shoes of the Fisherman | 2031 | 1968 |
| There's a Girl in My Soup | 1949 | 1970 |
| La Bohème | 1946 | 1926 |
| A Simple Plan | 1990 | 1998 |
| Chicken Run | 1989 | 2000 |
| Murder, Inc. | 1951 | 1960 |
| Home to Stay | 1947 | 1978 |
| The Number 23 | 1995 | 2007 |
| The Invisible Boy | 1968 | 1957 |
| A Boy, a Girl and a Bike | 1943 | 1949 |
| Disco Dancer | 2009 | 1982 |
| Rama Rama Krishna Krishna | 1991 | 2010 |
| Pyaar Ke Side Effects | 2020 | 2006 |

## Conclusiones

A Pesar de las ventajas que ofrece la solución mostrada en el blog ya que se puede alcanzar un error muy bajo en la clasificación por clase, el error por clasificar un dato en las todas las clases juntas puede llegar a ser muy alto, ya que es el producto de los errores de todas las clases, por lo que si se tienen en particular muchas clases quiza sea mejor intentar usar otro metodo.

En la regresión para encontrar los años de las películas encontramos que puede que no exista una relación entre la trama y el año, o tal vez se puedan alcanzar mejores resultados usando una red neuronal.

Para ambos casos quizá usar redes neuronales o otro método de codificación pueda mejorar la precisión, world 2 vec permite codificar las palabras a vectores densos de una dimensión especificada, en donde la cual se trata de modelar la cercanía de las palabras, o tal vez usar el universal sentence encoder, el cual toma un texto y lo convierte a un vector denso de 512, esto está pensado para obtener la similitud de textos en base a la distancia coseno. Además de estos también existen otros métodos que se podrían utilizar.


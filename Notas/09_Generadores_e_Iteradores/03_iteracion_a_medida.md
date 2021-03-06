[Contenidos](../Contenidos.md) \| [Anterior (1 El protocolo de iteración)](02_protocolo_Iteracion.md) \| [Próximo (3 Productores, consumidores, cañerías.)](04_Producers_consumers.md)

# 9.2 Iteración a medida

Acá vemos como usar una función generadora para obtener el iterador que necesites.

### Un problema

Suponé que querés crear su secuencia particular de iteración, una cuenta regresiva, por decir algo.

```python
>>> for x in regresiva(10):
...   print(x, end=' ')
...
10 9 8 7 6 5 4 3 2 1
>>>
```
Existe una forma fácil de hacer esto.

### Generadores

Un generador es una función que define un patrón de iteración.

```python
def regresiva(n):
    while n > 0:
        yield n
        n -= 1
```
*Nota: "yield" se traduce como "rendir" ó "entregar"*

Por ejemplo:

```python
>>> for x in regresiva(10):
...   print(x, end=' ')
...
10 9 8 7 6 5 4 3 2 1
>>>
```

Un generador es *cualquier* función que usa el commando `yield`.

El comportamiento de los generadores es algo diferente al del resto de las funciones.

Al llamar a un generador creás un objeto generador, pero su función no se ejecuta de inmediato. 

```python
def regresiva(n):
    # Agreguemos este print para ver qué pasa...
    print('Cuenta regresiva desde', n)
    while n > 0:
        yield n
        n -= 1
```

```python
>>> x = regresiva(10)
# No se ejecuta ningún PRINT !
>>> x
# sin embargo x es un objeto generador
<generator object at 0x58490>
>>>
```
La función sólo se ejecuta ante un llamado al método `__next__()`

```python
>>> x = regresiva(10)
>>> x
<generator object at 0x58490>
>>> x.__next__()
Cuenta regresiva desde 10
10
>>>
```

Lo que hace `yield` es notable: produce un valor, y luego suspende la ejecución de la función. La ejecución continúa al volver a llamar a `__next__()`.

```python
>>> x.__next__()
9
>>> x.__next__()
8
```
Cuando finalmente se llega al final de la función, la iteración da un error. 

```python
>>> x.__next__()
1
>>> x.__next__()
Traceback (most recent call last):
File "<stdin>", line 1, in ? StopIteration
>>>
```

*Observación: Una función generadora implementa el mismo protocolo de bajo nivel que los  `for` usan sobre listas, tuplas, diccionarios, archivos, etc.*

## Ejercicios

### Ejercicio 9.4: Un generador simplelabel_ej{Un generador simple}

Si te encontrás con la necesidad de obtener una iteración particular, siempre pensá en funciones generadoras. Son fáciles de escribir: hacé una función que implemente la lógica de iteración deseada y use `yield` para entregar valores.

Por ejemplo, probá este generador que busca un archivo y entrega las líneas que incluyen cierto substring.

```python
>>> def filematch(filename, substr):
        with open(filename, 'r') as f:
            for line in f:
                if substr in line:
                    yield line

>>> for line in open('Data/camion.csv'):
        print(line, end='')

nombre,cajones,precio
"Lima",100,32.20
"Naranja",50,91.10
"Limon",150,83.44
"Mandarina",200,51.23
"Durazno",95,40.37
"Mandarina",50,65.10
"Naranja",100,70.44
>>> for line in filematch('Data/camion.csv', 'Naranja'):
        print(line, end='')

"Naranja",50,91.10
"Naranja",100,70.44
>>>
```

Esta idea es muy interesante: podés armar una función que encapsule todo el procesamiento de datos y después recorrerla con un ciclo `for` para que te entregue los datos uno a uno.

El próximo ejemplo es de un caso aún mas especial.

### Ejercicio 9.5: Monitoreo de datos en tiempo real.
Un generador puede ser una forma interesante de vigilar datos a medida que son producidos. En esta sección vamos a probar esa idea. Para empezar, hacé lo siguiente.

El programa `Data/stocksim.py` es un generador de datos de precios. Al ejecutarlo, el programa escribe datos en un archivo llamado `Data/stocklog.csv` contínuamente hasta que es detenido. Corre durante varias horas y una vez que inicies su ejecución podés dejarlo correr y olvidarte de él. Abrí una consola del sistema operativo nueva y ejecutá el programa. Si estás en Windows, dale un doble click al ícono de `stocksim.py`, desde unix:

```bash
bash % python3 stocksim.py
```

Después, olvidate de él. Dejálo ahí, corriendo.

Usando otra consola, mirá el contenido de `Data/stocklog.csv`. Vas a ver que cada tanto se agrega una nueva línea al archivo. 

Ahora que el programa generador de datos está en ejecución, escribamos un programa que abra el archivo, vaya al final, y espere nuevos datos. Para esto creá un programa llamado `vigilante.py` que contenga el siguiente código.

```python
# vigilante.py
import os
import time

f = open('Data/stocklog.csv')
f.seek(0, os.SEEK_END)   # Mover el índice 0 posiciones desde el EOF

while True:
    line = f.readline()
    if line == '':
        time.sleep(0.1)   # Esperar un rato y volver a probar
        continue
    fields = line.split(',')
    name = fields[0].strip('"')
    price = float(fields[1])
    change = float(fields[4])
    if change < 0:
        print(f'{name:>10s} {price:>10.2f} {change:>10.2f}')
```

*Nota: EOF = End Of File (fin de archivo)*

Cuando ejecutes el programa vas a ver un indicador de precios en tiempo real. 

Observación: La forma en que usamos el método `readline()` en este ejemplo es un poco rara, no es la forma en que se suele usar (detro de un ciclo `for` para recorrer el contenido de un archivo). En este caso la estamos usando para mirar constantemente el fin de archivo para obtener los últimos datos que se hayan agregado (`readline()` devuelve ó bien el último dato o bien una cadena vacía) 

### Ejercicio 9.6: Uso de un generador para producir datos
Si analizás un poco el código en el ejercicio [Ejercicio 9.5](../09_Generadores_e_Iteradores/03_iteracion_a_medida.md#ejercicio-95-monitoreo-de-datos-en-tiempo-real) vas a notar que la primera parte del programa "produce" datos (los obtiene del archivo) y la segunda los procesa y los imprime , es decir "consume" datos. Una característica importante de las funciones generadoras es que podés mover todo el código a una función reutilizable. Fijate en esto...
 
Modificá el código del [Ejercicio 9.5](../09_Generadores_e_Iteradores/03_iteracion_a_medida.md#ejercicio-95-monitoreo-de-datos-en-tiempo-real) de modo que la lectura del archivo esté resuelta por una función generadora `seguir()` a la que se le pasa un nombre de archivo como parámetro. Hacelo de modo que el siguiente código funcione:

```python
>>> for line in follow('Data/stocklog.csv'):
          print(line, end='')

... Acá deberías ver las líneas impresas ...
```

Modificá el programa `vigilante.py` de modo que tenga esta pinta:

```python
if __name__ == '__main__':
    for line in follow('Data/stocklog.csv'):
        fields = line.split(',')
        name = fields[0].strip('"')
        price = float(fields[1])
        change = float(fields[4])
        if change < 0:
            print(f'{name:>10s} {price:>10.2f} {change:>10.2f}')
```

### Ejercicio 9.7: Cambios de precio de un camión
Modificá el programa `follow.py` para que sólo informe las líneas que tienen precios de fruta incluída en un camión, e ignore el resto de los productos. Por ejemplo: 

```python
if __name__ == '__main__':
    import report

    portfolio = report.read_portfolio('Data/portfolio.csv')

    for line in follow('Data/stocklog.csv'):
        fields = line.split(',')
        name = fields[0].strip('"')
        price = float(fields[1])
        change = float(fields[4])
        if name in portfolio:
            print(f'{name:>10s} {price:>10.2f} {change:>10.2f}')
```

Observación: para que esto funcione, tu clase `Portfolio` tiene que haber implementado el operador `in`. Controlá que tu solución al ejercicio [Ejercicio 9.3](../09_Generadores_e_Iteradores/02_protocolo_Iteracion.md#ejercicio-93-un-iterador-adecuado) implemente el operador `__contains__()`.

### Discusión

Hay que mencionar que acaba de suceder algo muy potente: Moviste tu patrón de iteración (el que toma las últimas líneas de un archivo) y lo pusiste en su propia función. La función `follow()` ahora es una función de uso amplio, que podés usar en cualquier programa. Por ejemplo la podrias usar para mirar historial (logs) en un servidor ó de un debugger, o de otras fuentes contínuas de datos.

No está bueno ?


[Contenidos](../Contenidos.md) \| [Anterior (1 El protocolo de iteración)](02_protocolo_Iteracion.md) \| [Próximo (3 Productores, consumidores, cañerías.)](04_Producers_consumers.md)


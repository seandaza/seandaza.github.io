
> `Flask` es un framework para desarrollo web escrito en Python. Se puede utilizar para diversos tipos de aplicación, entre ellas, desarrollo de APIs. De hecho, es uno de los mejores frameworks de desarrollo para implementar este tipo de soluciones por lo sencillo que resulta y las facilidades que ofrece.

> Existen muchas maneras de implementar un API REST con Flask. Desde usar el framework con lo que ofrece de base, hasta instalar varias extensiones con diferentes configuraciones.

> En este Notebook veremos cómo consumir una API usando python flask en Google Colab a traves de `ngrok` para hacer la IP pública desde nuestro local. 

### [](#header-2)ngrok


> `Ngrok` nos permite exponer a internet una URL generada dinámicamente, la cual apunta a un servicio web que se está ejecutando en nuestra máquina local. Por ejemplo: si tenemos un servicio web arrancado en http://localhost:8080, ngrok genera dinámicamente una URL del tipo http://xxxxxx.ngrok.io visible en internet, y que apunta directamente a nuestro localhost.  


### [](#header-2)Webservice con flask

> Nuestro objetivo en esta sección es  crear `werbservices` y hacer que corran en  `google colab` con la ayuda de `flask`. Para ello, veamos como funciona google colab(GC): 

> Cuando se ejecuta cualquier código en GC, este código se está ejecutando en una máquina virtual en mi entorno local `127.0.0.1`. Esta direccion ip no es accesible desde fuera. Para ello, `ngrok` nos ayuda a exponer esta ip y hacerla visible desde fuera en una url pública.

> veamos como implementar esto de manera práctica:

```python
# Python code with syntax highlighting
# Instalamos dependencias
!pip install flask-ngrok
```

En la salida de la siguiente ejecución, encontrarás dos url que corren nuestra aplicación en modo local y otra con extensión `ngrok.io`, la cual me expone mi aplicación en una url pública. Da click en ella y navega por las rutas que creamos.

```python
# Python code with syntax highlighting
# Instalamos dependencias
from flask_ngrok import run_with_ngrok
from flask import Flask

#Corramos la aplicación Flask
app = Flask(__name__) 

# Necesitamos iniciar ngrok cuando la app este corriendo.
run_with_ngrok(app)

#Definamos unas rutas

#Ruta raiz
@app.route("/")
def index ():
  return "<h1>Trail of Flask with Google Colab</h1>"

#Ruta get_details
@app.route("/get_details")
def get_details():
  return "<h1>This is teh get details page!</h1>"

#ruta_test
@app.route("/test")
def test_page():
  return "<h1>This is the test page</h1>"

#Ejecutemos la aplicacion
app.run()

```

### [](#header-4)Ejemplo

> Nota: Para el siguiente ejemplo, es importante que guardes primero en local el archivo `products.py` para que puedas ejecutarlo. Puedes descargarlo ejecutando la siguiente linea de código:

```python
# Python code with syntax highlighting
# Instalamos dependencias

!wget -O products.py  https://github.com/andresrosso/resources/blob/main/coding/uan_gc_2021/products.py?raw=True
```

### [](#header-5)Creando un Webservice
Suponga que tiene una lista de productos informáticos almacenados en un documento `products.py` los cuales estan en formato `.json`. Necesitamos crear un webservice que me permita consultar los diversos productos y sus características, entonces para ello, creamos diferentes rutas que me permitan obtenerlos a través del método `GET`. Veamos: 

Ejecuta el siguiente código, ingresa a la url que entrega `ngrok` y navega por las diferentes rutas para que obtengas los datos. 

```python
# Python code with syntax highlighting
# Instalamos dependencias
from flask_ngrok import run_with_ngrok
from flask import Flask, jsonify

app = Flask(__name__)

# Necesitamos iniciar ngrok cuando la app este corriendo.
run_with_ngrok(app)




#Ruta raiz
@app.route("/")
def index ():
  return "<h1>Office Products</h1>\n<p>Here you found office products!</p>\n<u>navigate to 'producs' rout for more details</u>"

#Ruta /ping que muestra un mensaje al visitarla
@app.route('/ping')
def ping ():
      return jsonify({"message": "Pong"})


#Ruta '/products'
@app.route('/products', methods=['GET'])
def getProducts(): 
  return jsonify(products)

#Ruta '/products/XXXXXX'
@app.route('/products/<string:product_name>', methods=['GET'])
def getProduct(product_name):
      productsFound = [product for product in products if product['name'] == product_name]
      if len(productsFound) > 0:     
        return jsonify({"product": productsFound[0]})
      return jsonify({"message": "Product not found"})


app.run()
```

En la primera ruta, cuando se navega a la ruta `'/ping'`, el navegador nos muestra en formato json un mensaje. 
En la segunda ruta, cuando se navega a la ruta `'/products'`, el navegador nos muestra el conjunto total de porductos que se hayan en `products.py`.
Si estando el la ruta de `/products` se navega a `/products/laptop` por ejemplo, entonces el navegador nos muestra detalle de ese producto en específico. Prueba navegando sobre los demas productos o sobre un producto inexistente para que veas lo que te muestra el navegador.



## [](#header-6)Webservice para una base de Datos


> Nuestro objetivo será ahora exponer las consultas que podamos hacer desde base de datos local, a una url pública a través de `ngrok`.

> Para ello, hay que tener en cuenta en primer lugar, que nuestra base de datos en este caso, es de tipo relacional; es decir, está conformada por una serie de tablas con información desplegada en las mismas. Es por ello, que en el proceso, hacemos las consultas a la base de datos a través de `sqlite3` y luego transformamos los DataFrames en diccionarios para poder de esta manera, construír las rutas a través de sus llaves.

> Para el siguiente ejemplo, descargue la base de datos `hr.db` en la siguiente ejecución de código. Esta base de datos como ya dijimos, consta de una serie de datos de recursos humanos de detrminada empresa.



```python
# Python code with syntax highlighting
!wget -O hr.db https://github.com/andresrosso/resources/blob/main/coding/uan_gc_2021/hr.db?raw=true/
```
Importemos las dependecias necesarias

```python
# Python code with syntax highlighting
# Importemos Librerías
import sqlite3 as sq
import pandas as pd
from sqlite3 import Error
from flask import Flask, request, jsonify
import json
```
Establezcamos conexión con la base de datos

```python
# Python code with syntax highlighting
# Conexion a la base de datos llamada hr.db
conn = sq.connect('hr.db')
cur = conn.cursor()

```

Para un mejor reconocimento de las tablas que comprenden esta base de datos, listemos las tablas ejecutando el siguiente código:

```python
# Python code with syntax highlighting

# Identifiquemos las tablas que contiene esta Base de Datos
tables_list = [a for a in cur.execute("SELECT name FROM sqlite_master WHERE type = 'table'")]

# Lista de Tablas
print(tables_list)
```

Elijamos una de las Tablas mostradas y establezcamos un webservice para ella:


```python

#Webservice para Tabla 'employees':

# Conexion a la base de datos llamada hr.db
conn = sq.connect('hr.db')
cur = conn.cursor()


# Definiendo la Consulta a la BD
q1 = ('select * from employees ')

# Convert the SQL query to Pandas data Frame
r1 = pd.read_sql(q1, conn)

# Convirtiendo el DataFrame 'employees' en Diccionario
result1 = r1.to_dict()

#Listando las columnas del DataFrame 'employees'
container1 = list(result1)
container1

```


```python
# Python code with syntax highlighting

#Corremos la aplicacion flask
app = Flask(__name__)


# Iniciamos ngrok cuando la app este corriendo.
run_with_ngrok(app)

#Ruta raiz
@app.route("/")
def index ():
  return "<h1>Employees Table</h1>\n<p>Principal Route: 'employees'</p>\n<u>Subroutes:</u><ul><li>employee_id</li><li>first_name</li><li>last_name</li><li>email</li><li>phone_number</li><li>hire_date</li><li>job_id</li><li>salary</li><li>manager_id</li><li>department_id</li></ul>"


#Ruta para la Tabla Empleados
@app.route('/employees', methods=['GET'])
def getEmployees(): 
  return jsonify(result1)


#Subrutas de la Tabla Empleados
@app.route('/employees/<string:atributes_employee>', methods=['GET'])
def getProduct(atributes_employee):
  if atributes_employee in container1:
    return jsonify(result1[f'{atributes_employee}'])
  return jsonify({"message": f"{atributes_employee} not Found!"})

 
 

app.run()


```

---
Creemos ahora el Webservice para la Tabla 'departments'. El procediiento es el mismo, solo que cambiaremos el nombre de algunas variables haciendo uso de indices numericos para preservar el sentido de las mismas.


```python
# Python code with syntax highlighting

#Webservice para Tabla 'departments':

# Conexion a la base de datos llamada hr.db
conn = sq.connect('hr.db')
cur = conn.cursor()


# Definiendo la Consulta a la BD
q2 = ('select * from departments ')

# Convert the SQL query to Pandas data Frame
r2 = pd.read_sql(q2, conn)

# Convirtiendo el DataFrame 'employees' en Diccionario
result2 = r2.to_dict()

#Listando las columnas del DataFrame 'employees'
container2 = list(result2)
container2

```
Comprueba el Webservice y navega por sus Subrutas:

```python
# Python code with syntax highlighting

#Corremos la aplicacion flask
app = Flask(__name__)


# Iniciamos ngrok cuando la app este corriendo.
run_with_ngrok(app)


#Ruta para la Tabla Empleados
@app.route('/departments', methods=['GET'])
def getEmployees(): 
  return jsonify(result2)


#Subrutas de la Tabla Empleados
@app.route('/departments/<string:atributes_employee>', methods=['GET'])
def getProduct(atributes_employee):
  if atributes_employee in container2:
    return jsonify(result2[f'{atributes_employee}'])
  return jsonify({"message": f"{atributes_employee} not Found!"})

 
 

app.run()

```

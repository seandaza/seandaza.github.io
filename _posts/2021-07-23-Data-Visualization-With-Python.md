


> El objetivo de este apartado es proporcionar las habilidades básicas para procesar y visualizar datos a partir de una base de datos relacional  mediante el lenguaje de programación Python.


### [](#header-2)Definición

> Una Base de Datos es un sistema para archivar información en computadora, cuyo propósito general es mantener información y hacer que esté disponible cuando se solicite.


 
### [](#header-2)Modelos de Datos

> Es una representación abstracta de los datos de una organización y las relaciones entre ellos. Un modelo de datos describe una organización.

### [](#header-2)Tipos de Modelos de Datos


1.Lógicos Basados en Objetos :

	a. Modelo Entidad-Realación
	b. Orientado a Objetos

2.Lógicos Basados en Registros:

	a.  Relacional
	b.  Red
	c.  Jerárquico


3.Físicos de Datos


### [](#header-2)Sentencias SQL

>Algunas sentencias a tener presentes a la hora de trabajar con bases de datos son la siguientes:



1.   Data Manipulation languaje (DML) : SELECT, INSERT, UPDATE, DELETE, MERGE.
2.   Data Definition languaje (DDL) :  CREATE, ALTER, DROP, RENAME, TRUNCATE, COMMENT.
3.   Data Control Languaje (DCL) : GRANT, REVOKE.
2.   Transaction Control : COMMIT, ROLLBACK, SAVEPOINT.




### [](#header-2) Preliminares

> Antes que nada, es importante que almacenes en local la base de datos `hr.db` con la que estaremos trabajando para que no presentes problemas de conexión. 

> Esta base de datos comprende un conjunto de tablas que informan al respecto de los recursos humanos de una empresa. Entre las tablas estan: employees, departments, jobs, locations, countries, regions, dependents, job_grades.
puedes ver el modelo Entidad-Relacion de la base de datos en el siguiente [Link](https://github.com/andresrosso/resources/blob/main/images/schema.png). Detalla cada tabla junto a las columnas que las componen.


```python
!wget https://github.com/andresrosso/resources/blob/main/coding/uan_gc_2021/hr.db
```

```python
!ls
```
### [](#header-2) Configuración

```python
# Standard libraries
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import glob
import plotly.express as px
import os
import warnings

# Local server SQL database
import sqlite3 as sq

# Setting of Large numbers format
pd.options.display.float_format = '{:,.2f}'.format

# Set data frame display max 10 rows
pd.set_option('display.max_rows', 10)

# Warning is suppressed
warnings.simplefilter(action='ignore', category=FutureWarning)

```

```python
# Conexion a la base de datos llamada hr.db
con = sq.connect('hr.db')
cur = con.cursor()
```

```python
# Identidiquemos las tablas que contiene esta Base de Datos
table_list = [a for a in cur.execute("SELECT name FROM sqlite_master WHERE type = 'table'")]

# Lista de Tablas
print(table_list)
```

### [](#header-4)Consultas SQL y visualización a través de Pandas

> Enecerramos la consulta dentro de una variable, la procesamos a través de Pandas estableciendo conexion a la base de datos y la respuesta la guardamos de otra variable.

```python
# Consulta a SQL para ver la tabla 'employees'
q0 = ('select * from employees;')

# Convert the SQL query to Pandas data Frame
r0 = pd.read_sql(q0, con)
r0
```
Esa consulta me trae toda la tabla. Pero si queremos visualizar solo algunas columnas de la tabla hacemos los siguiente:

```python
# De la tabla 'employees' veamos solo algunas de sus columnas
q1 = ( 'select first_name, last_name, email from employees;')

# Convert the SQL query to Pandas data Frame
r1 = pd.read_sql(q1, con)
r1
```


### [](#header-5)Filtrando Datos

> Si queremos que nuestra búsqueda sea mas específica, usamos la sentencia WHERE, para dar mas detalle del dato que queremos visualizar a partir de uno de sus atributos.

```python
# De la tabla 'employees' filtremos en funcion a un atributo
q2 = ('SELECT * from employees WHERE email = "diana.lorentz@sqltutorial.org"')

# Convert the SQL query to Pandas data Frame
r2 = pd.read_sql(q2, con)
r2
```

Visualicemos ahora la tabla jobs:

```python
# Consultando toda la tabla 'jobs'
q3 = ('select * from jobs j ')

# Convert the SQL query to Pandas data Frame
r3 = pd.read_sql(q3, con)
r3
```

Podemos hacer una Descripción estadística de los datos de esa tabla usando el método .describe()

```python
r3.describe() 
```

### [](#header-5)Graficando Datos

```python
# Graficando los salarios máximos por categoría de Empleo
fig1 = px.bar(r3, x="job_title", y="max_salary", orientation='v', title='Salario Maximo por Categoria de Empleo')
fig1.show()
```
### [](#header-5)Operaciones Aritméticas y Sentencia **As**

> Podemos crear una nueva columna en la tabla cuyas entradas sean una combinación lineal de los valores numéricos de otra columna dada, y también podemos nombrarla con un Alias usando la sentencia as.

```python
# creando la Columna "comission"
q4 = ('select first_name, last_name, salary, (salary*0.1) as comission from employees ')

# Convert the SQL query to Pandas data Frame
r4 = pd.read_sql(q4, con)
r4
```
### [](#header-5)Sentencia BETWEEN

> Podemos hacer consultas entre rangos numéricos, mediante BETWEEN

```python
# Consulta a SQL para vaer un rango específico
q5 = ('SELECT employee_id, first_name, last_name, email FROM employees WHERE employee_id BETWEEN 100 AND 150; ')

# Convert the SQL query to Pandas data Frame
r5 = pd.read_sql(q5, con)
r5
```

### [](#header-5)Función SUM

```python
# Consulta a SQL para ver el total pagado por concepto de salarios
q6 = ('SELECT SUM(salary) as summary FROM employees; ')

# Convert the SQL query to Pandas data Frame
r6 = pd.read_sql(q5, con)
r6
```
### [](#header-5)Agrupando Datos con GROUP BY

> Si queremos agrupar datos usamos la sentencia GROUP BY

```python
# Consulta a SQL para ver el total pagado por concepto de salarios por Departamento
q7 = (
      'SELECT department_id, SUM(salary) AS summary '
      'FROM employees '
      'GROUP BY department_id'
     )

# Convert the SQL query to Pandas data Frame
r7 = pd.read_sql(q6, con)
r7
```

Note que se muestra los totales de gastos por salario agrupado por departamento, pero no vemos con detalle el nombre del departamento, sino solo su id. A la hora de general un reporte, ver el nombre del departamento aportaria mas al detalle. Veamos como ver el nombre del departamento

### [](#header-5)Función INNER JOIN

> Me va a traer la información de los datos que están en común entre dos tablas.

> Revisa el modelo de Entidad- Relación de nuestra base de datos en el siguiente Link

### [](#header-5)Ejemplo 1

> Queremos ver el total de salario pagados por departamentos. Para ello, Seleccionamos las columnas de las tablas que me vinculan la información necesaria haciendo uso de los Alias e intersectamos ambas tablas con la columna que tiene en comun (department_id) y las agrupamos bajo un criterio conveniente (department_name). Veamos: 

```python
# Consulta a SQL para ver el total pagado por concepto de salarios
q8 = (
      'SELECT d.department_name, SUM(e.salary) AS summary '
      'FROM employees e '
      'INNER JOIN departments d ON e.department_id = d.department_id'
      ' GROUP BY d.department_name;'
     )

# Convert the SQL query to Pandas data Frame
r8 = pd.read_sql(q7, con)
r8
```

### [](#header-5)Graficando Datos

```python
# Graficando los salarios máximos por categoría de Empleo
fig2 = px.line(r7, x="department_name", y="summary", title='Salario Total por Departamento')
fig2.show() 
```
### [](#header-5)Ejemplo 2

> Vamos a ver la lista total de empleados asociados a el nombre del departamento al cual pertenecen. Para ello, hacemos uso nuevamente de la sentencia INNER JOIN

```python
# Consulta a SQL para ver el total pagado por concepto de salarios
q9 = (
      'SELECT e.employee_id, e.first_name, e.last_name, d.department_name '
      'FROM employees e '
      'INNER JOIN departments d ON e.department_id = d.department_id'
     )

# Convert the SQL query to Pandas data Frame
r9 = pd.read_sql(q8, con)
r9 
```
### [](#header-5)TALLER

1. El departamento de recursos humanos desea que una consulta muestre el apellido, la identificación del trabajo, la fecha de contratación y la identificación del empleado, apareciendo primero la identificación del empleado. Proporcione un alias STARDATE para la columna HIRE_DATE.

2. A la tabla que se generó en el ejemplo 2 de la sección de INNER JOIN, cree una nueva columna que muestre el tipo de empleo (job_title) que tiene cada empleado y adjuntela ala tabla.

3. El departamento de Recursos Humanos necesita un informe de los empleados en Toronto. Muestre el Apellido, el trabajo, el numero de departamento y el nombre del departamento para todos lo empleados que trabajan en Toronto.





# DBD-2025

- Banda = (codigoB, nombreBanda, genero_musical, anio_creacion)
- Integrante = (DNI, nombre, apellido, dirección, email, fecha_nacimiento, codigoB(fk))
- Escenario = (nroEscenario, nombre_escenario, ubicación, cubierto, m2, descripción)
- Recital = (fecha, hora, nroEscenario (fk), codigoB (fk))


### Iniciso 1

* Listar DNI, nombre, apellido,dirección y email de integrantes nacidos entre 1980 y 1990 y que
hayan realizado algún recital durante 2023.

```sql
SELECT DNI, nombre, apellido, direccion, email
FROM Integrante i
WHERE fecha_nacimiento BETWEEN '1980-01-01' and '1990-12-31'
      and exists ( SELECT 1
                   FROM Recital r
                   WHERE r.fecha BETWEEN '2023-01-01' and '2023-12-31' and (r.codigoB = i.codigoB));
```

### Iniciso 2

*  Reportar nombre, género musical y año de creación de bandas que hayan realizado recitales
durante 2023, pero no hayan tocado durante 2022 .


```sql
(SELECT nombreBanda, genero_musical, anio_creacion
FROM Banda NATURAL JOIN Recital
WHERE fecha BETWEEN '2023-01-01' and '2023-12-31')

except

(SELECT nombreBanda, genero_musical, anio_creacion
FROM Banda NATURAL JOIN Recital
WHERE fecha BETWEEN '2022-01-01' and '2022-12-31');
```

### Iniciso 3

* Listar el cronograma de recitales del día 04/12/2023. Se deberá listar nombre de la banda que
ejecutará el recital, fecha, hora, y el nombre y ubicación del escenario correspondiente.

```sql
SELECT nombreBanda, fecha, hora, nombre_escenario, ubicacion
FROM Banda NATURAL JOIN Recital NATURAL JOIN Escenario
WHERE fecha = '2023-12-04' 
```

### Iniciso 4

* Listar DNI, nombre, apellido,email de integrantes que hayan tocado en el escenario con nombre
‘Gustavo Cerati’ y en el escenario con nombre ‘Carlos Gardel’.

```sql
SELECT DNI, nombre, apellido, email
FROM Integrante i
WHERE EXISTS (SELECT 1
              FROM Banda b 
                NATURAL JOIN Recital
                NATURAL JOIN (SELECT nroEscenario 
                              FROM Escenario 
                              WHERE nombre_escenario = 'Gustavo Cerati') r
              WHERE i.codigoB = b.codigoB)
      AND EXISTS (SELECT 1
              FROM Banda b2 
                NATURAL JOIN Recital
                NATURAL JOIN (SELECT nroEscenario 
                              FROM Escenario 
                              WHERE nombre_escenario = 'Carlos Gardel') r2
              WHERE i.codigoB = b2.codigoB);
```

### Iniciso 5

* Reportar nombre, género musical y año de creación de bandas que tengan más de 5 integrantes.

```sql
SELECT nombreBanda, genero_musical, anio_creacion
FROM Banda b INNER JOIN Integrante i ON (i.codigoB = b.codigoB)
GROUP BY b.codigoB, nombreBanda, genero_musical, anio_creacion
HAVING COUNT(*) > 5;
```

### Iniciso 6

* Listar nombre de escenario, ubicación y descripción de escenarios que solo tuvieron recitales
con el género musical rock and roll. Ordenar por nombre de escenario

```sql
(SELECT nombre_escenario, ubicacion, descripcion
FROM (SELECT codigoB FROM Banda WHERE genero_musical = 'rock and roll') b
      NATURAL JOIN Recital 
      NATURAL JOIN Escenario)

except

(SELECT nombre_escenario, ubicacion, descripcion
FROM (SELECT codigoB FROM Banda WHERE genero_musical <> 'rock and roll') b
      NATURAL JOIN Recital 
      NATURAL JOIN Escenario)

ORDER BY nombre_escenario;
```

### Iniciso 7

*  Listar nombre, género musical y año de creación de bandas que hayan realizado recitales en
escenarios cubiertos durante 2023.// cubierto es true, false según corresponda

```sql
SELECT DISTINCT nombreBanda, genero_musical, anio_creacion
FROM Banda 
     NATURAL JOIN (SELECT codigoB, nroEscenario 
                   FROM Recital 
                   WHERE fecha BETWEEN '2023-01-01' and '2023-12-31') r
     NATURAL JOIN (SELECT nroEscenario
                  FROM Escenario
                  WHERE cubierto) e

```

### Iniciso 8

* 8. Reportar para cada escenario, nombre del escenario y cantidad de recitales durante 2024.

```sql
SELECT nombre_escenario, COUNT(*) AS cantidadDeRecitales
FROM Escenario e NATURAL JOIN (SELECT nroEscenario
                             FROM Recital
                             WHERE fecha BETWEEN '2024-01-01' and '2024-12-31') r
GROUP BY e.nroEscenario, nombre_escenario
```

### Iniciso 9

* Modificar el nombre de la banda ‘Mempis la Blusera’ a: ‘Memphis la Blusera’.

```sql
UPDATE Banda SET nombreBanda = 'Memphis la Blusera';
```

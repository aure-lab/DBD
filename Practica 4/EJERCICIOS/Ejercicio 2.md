# DBD-2025

- Localidad = (codigoPostal, nombreL, descripcion, nroHabitantes)
- Arbol = (nroArbol, especie, anios, calle, nro, codigoPostal(fk))
- Podador = (DNI, nombre, apellido, telefono, fnac, codigoPostalVive(fk))
- Poda = (codPoda, fecha, DNI(fk), nroArbol(fk))

### Iniciso 1

* Listar especie, años, calle, nro y localidad de árboles podados por el podador ‘Juan Perez’ y por
el podador ‘Jose Garcia’.

```sql
(SELECT especie, anios, calle, nro, codigoPostal
FROM Arbol NATURAL JOIN Poda NATURAL JOIN Podador p
WHERE p.nombre = "Juan" and p.apellido = "Perez")
intersect
(SELECT especie, anios, calle, nro, codigoPostal
FROM Arbol NATURAL JOIN Poda NATURAL JOIN Podador
WHERE nombre = "Jose" and apellido = "Garcia");
```

### Iniciso 2

* Reportar DNI, nombre, apellido, fecha de nacimiento y localidad donde viven de aquellos
podadores que tengan podas realizadas durante 2023.

```sql
SELECT DISTINCT DNI, nombre, apellido, fnac, codigoPostalVive
FROM Podador NATURAL JOIN Poda
WHERE fecha between "2023-01-01" and "2023-12-31";  
```

### Iniciso 3

* Listar especie, años, calle, nro y localidad de árboles que no fueron podados nunca.

```sql
SELECT especie, anios, calle, nro, codigoPostal
FROM Arbol
WHERE nroArbol NOT IN (select nroArbol from Poda) 
```

### Iniciso 4

* Reportar especie, años,calle, nro y localidad de árboles que fueron podados durante 2022 y no
fueron podados durante 2023.

```sql
SELECT especie, anios, calle, nro, codigoPostal
FROM Arbol
WHERE nroArbol IN (Select nroArbol from Poda where fecha between "2022-01-01" and "2022-12-31")
    AND nroArbol NOT IN (Select nroArbol from Poda where fecha between "2023-01-01" and "2023-12-31")
```

### Iniciso 5

* Reportar DNI, nombre, apellido, fecha de nacimiento y localidad donde viven de aquellos
podadores con apellido terminado con el string ‘ata’ y que tengan al menos una poda durante
2024. Ordenar por apellido y nombre.

```sql
SELECT DNI, nombre, apellido, fnac, nombreL
FROM Podador NATURAL JOIN Localidad
WHERE apellido like "%ata" and DNI IN (select DNI from Poda where fecha between  "2024-01-01" and "2024-12-31")
ORDER BY apellido, nombre;
```

### Iniciso 6

* Listar DNI, apellido, nombre, teléfono y fecha de nacimiento de podadores que solo podaron
árboles de especie ‘Coníferas’.

```sql
(SELECT DNI, apellido, nombre, telefono, fnac
FROM Podador p NATURAL JOIN Poda NATURAL JOIN Arbol
WHERE especie = 'Coniferas')

except

(SELECT DNI, apellido, nombre, telefono, fnac
FROM Podador p NATURAL JOIN Poda NATURAL JOIN Arbol
WHERE especie <> 'Coniferas')
```

### Iniciso 7

* Listar especies de árboles que se encuentren en la localidad de ‘La Plata’ y también en la localidad de ‘Salta’.

```sql
SELECT especie
FROM Arbol 
WHERE codigoPostal IN (SELECT codigoPostal FROM Localidad WHERE nombreL = 'La Plata')

intersect

SELECT especie
FROM Arbol 
WHERE codigoPostal IN (SELECT codigoPostal FROM Localidad WHERE nombreL = 'Salta')

```

### Iniciso 8

* Eliminar el podador con DNI 22234566.

```sql
DELETE FROM Poda WHERE DNI = 22234566;
DELETE FROM Podador WHERE DNI = 22234566;
```

### Iniciso 9

*  Reportar nombre, descripción y cantidad de habitantes de localidades que tengan menos de 5
árboles.

```sql
SELECT nombreL, descripcion, nroHabitantes
FROM Arbol NATURAL JOIN Localidad 
GROUP BY codigoPostal, nombreL, descripcion, nroHabitantes
HAVING COUNT(nroArbol) < 5
```
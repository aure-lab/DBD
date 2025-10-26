# DBD-2025

- Persona = (DNI, Apellido, Nombre, Fecha_Nacimiento, Estado_Civil, Genero)
- Alumno = (DNI (fk), Legajo, Anio_Ingreso)
- Profesor = (DNI (fk), Matricula, Nro_Expediente)
- Titulo = (Cod_Titulo, Nombre, Descripcion)
- Titulo-Profesor = (Cod_Titulo (fk), DNI (fk), Fecha)
- Curso = (Cod_Curso, Nombre, Descripcion, Fecha_Creacion, Duracion)
- Alumno-Curso = (DNI (fk), Cod_Curso (fk), Anio, Desempenio, Calificacion)
- Profesor-Curso = (DNI (fk), Cod_Curso (fk), Fecha_Desde, Fecha_Hasta?)


### Iniciso 1

* Listar DNI, legajo y apellido y nombre de todos los alumnos que tengan año de ingreso inferior a
2014.

```sql
SELECT DNI, Legajo, Apellido, Nombre
FROM Persona NATURAL JOIN Alumno
WHERE Anio_Ingreso < 2014;
```

### Iniciso 2

* Listar DNI, matrícula, apellido y nombre de los profesores que dictan cursos que tengan más de
100 horas de duración. Ordenar por DNI.

```sql
SELECT DNI, Matricula, Apellido, Nombre
FROM Persona 
NATURAL JOIN Profesor 
NATURAL JOIN Profesor_Curso 
NATURAL JOIN (SELECT Cod_Curso FROM Curso WHERE Duracion>100) c
ORDER BY DNI;
```

### Iniciso 3

* Listar el DNI, Apellido, Nombre, Género y Fecha de nacimiento de los alumnos inscriptos al
curso con nombre “Diseño de Bases de Datos” en 2023.

```sql
SELECT DNI, Apellido, Nombre, Genero, Fecha_Nacimiento
FROM Persona 
NATURAL JOIN Alumno_Curso 
NATURAL JOIN (SELECT Cod_Curso FROM Curso WHERE Nombre = 'Diseño de Bases de Datos') c
WHERE Anio = 2023; 
```

### Iniciso 4

* Listar el DNI, Apellido, Nombre y Calificación de aquellos alumnos que obtuvieron una
calificación superior a 8 en algún curso que dicta el profesor “Juan Garcia”. Dicho listado deberá
estar ordenado por Apellido y nombre.

```sql
SELECT DNI, Apellido, Nombre, Calificacion
FROM Persona 
NATURAL JOIN Alumno 
NATURAL JOIN Alumno_Curso
NATURAL JOIN (SELECT Cod_Curso 
              FROM (SELECT DNI FROM Persona WHERE Nombre = 'Juan' AND Apellido = 'Garcia') jg
              NATURAL JOIN Profesor 
              NATURAL JOIN Profesor_Curso) CursosDeJg
WHERE Calificacion>8
ORDER BY Apellido, Nombre;
```

### Iniciso 5

* Listar el DNI, Apellido, Nombre y Matrícula de aquellos profesores que posean más de 3 títulos.
Dicho listado deberá estar ordenado por Apellido y Nombre.

```sql
SELECT DNI, Apellido, Nombre, Matricula
FROM Persona
NATURAL JOIN Profesor 
NATURAL JOIN  Titulo_Profesor t
GROUP BY DNI, Apellido, Nombre, Matricula
HAVING COUNT(t.Cod_Titulo) > 3
ORDER BY Apellido, Nombre;
```

### Iniciso 6

* Listar el DNI, Apellido, Nombre, Cantidad de horas y Promedio de horas que dicta cada profesor.
La cantidad de horas se calcula como la suma de la duración de todos los cursos que dicta.

```sql
SELECT DNI, Apellido, Nombre, SUM(Duracion) AS Cantidad_de_horas, AVG(Duracion) AS Promedio_de_horas
FROM Persona 
NATURAL JOIN Profesor
NATURAL JOIN Profesor_Curso
NATURAL JOIN (SELECT Cod_Curso, Duracion FROM Curso) c
GROUP BY DNI, Apellido, Nombre; 
```

### Iniciso 7

*  Listar Nombre y Descripción del curso que posea más alumnos inscriptos y del que posea
menos alumnos inscriptos durante 2024.

```sql
SELECT Nombre, Descripcion
FROM Curso NATURAL JOIN Alumno_Curso
GROUP BY Cod_Curso
HAVING COUNT(DNI) >=ALL (SELECT COUNT(DNI)
                      FROM Curso NATURAL JOIN Alumno_Curso
                      WHERE Anio = 2024
                      GROUP BY Cod_Curso)
       OR COUNT(DNI) <=ALL (SELECT COUNT(DNI)
                      FROM Curso NATURAL JOIN Alumno_Curso
                      WHERE Anio = 2024
                      GROUP BY Cod_Curso);
```

### Iniciso 8

* Listar el DNI, Apellido, Nombre y Legajo de alumnos que realizaron cursos con nombre
conteniendo el string ‘BD’ durante 2022 pero no realizaron ningún curso durante 2023.

```sql
SELECT DNI, Apellido, Nombre, Legajo
FROM Alumno 
NATURAL JOIN Persona p
WHERE EXISTS (SELECT 1
              FROM Alumno_Curso a
              NATURAL JOIN (SELECT Cod_Curso
                            FROM Curso
                            WHERE Nombre LIKE '%BD%') c
              WHERE Anio = 2022 AND p.DNI = a.DNI)
      AND NOT EXISTS (SELECT 1
                      FROM Alumno_Curso a
                      NATURAL JOIN Curso 
               WHERE Anio = 2023 AND p.DNI = a.DNI);
```

### Iniciso 9

* Agregar un profesor con los datos que prefiera y agregarle el título con código: 25.

```sql
INSERT INTO Persona (DNI, Apellido, Nombre, Fecha_Nacimiento, Estado_Civil, Genero) VALUES (47, 'Perez', 'Juan', '2005-12-27', 'soltero', 'M');
INSERT INTO Profesor (DNI, Matricula, Nro_Expediente) VALUES  (47, 'no tiene', 2);
INSERT INTO Titulo (Cod_Titulo, Nombre, Descripcion) VALUES (25, 'Informatico', 'epico');
INSERT INTO Titulo_Profesor (Cod_Titulo, DNI, Fecha) VALUES (25, 47, '2023-01-01');
```

### Iniciso 10

*  Modificar el estado civil del alumno cuyo legajo es ‘2020/09’, el nuevo estado civil es divorciado

```sql
UPDATE Persona SET Estado_Civil = 'Divorciado' WHERE DNI = (SELECT DNI FROM Alumno WHERE Legajo = '2020/09'); 
```

### Iniciso 11

* Dar de baja el alumno con DNI 30568989. Realizar todas las bajas necesarias para no dejar el
conjunto de relaciones en un estado inconsistente.

```sql
DELETE FROM Alumno_Curso WHERE DNI = 30568989;
DELETE FROM Alumno WHERE DNI = 30568989;
DELETE FROM Persona WHERE DNI = 30568989;
```
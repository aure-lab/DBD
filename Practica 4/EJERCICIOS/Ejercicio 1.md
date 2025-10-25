# DBD-2025

Cliente = (idCliente, nombre, apellido, DNI, telefono, direccion)
Factura = (nroTicket, total, fecha, hora, idCliente (fk))
Detalle = (nroTicket (fk), idProducto (fk), cantidad, preciounitario)
Producto = (idProducto, nombreP, descripcion, precio, stock)

### Ejercicio 1

> Listar datos personales de clientes cuyo apellido comience con el string ‘Pe’. Ordenar por DNI.

```sql
SELECT nombre, apellido, DNI, telefono, direccion
FROM Cliente
WHERE apellido like "Pe%"
ORDER BY DNI
```

### Ejercicio 2

> Listar nombre, apellido, DNI, teléfono y dirección de clientes que realizaron compras solamente
> durante 2024.

```sql
(SELECT nombre, apellido, DNI, telefono, direccion
FROM Cliente NATURAL JOIN Factura
WHERE fecha between "2024-01-01" and "2024-12-31")
except
(SELECT nombre, apellido, DNI, telefono, direccion
FROM Cliente NATURAL JOIN Factura
WHERE fecha < "2024-01-01" or fecha > "2024-12-31");
```

### Ejercicio 3

> Listar nombre, descripción, precio y stock de productos vendidos al cliente con DNI 45789456,
> pero que no fueron vendidos a clientes de apellido ‘Garcia’.

```sql
(SELECT nombreP, descripcion, precio, stock
FROM Cliente NATURAL JOIN Factura NATURAL JOIN Detalle NATURAL JOIN Producto
WHERE DNI = 45789456)
except
(SELECT nombreP, descripcion, precio, stock
FROM Cliente NATURAL JOIN Factura NATURAL JOIN Detalle NATURAL JOIN Producto
WHERE apellido = "Garcia");
```

### Ejercicio 4

> Listar nombre, descripción, precio y stock de productos no vendidos a clientes que tengan
> teléfono con característica 221 (la característica está al comienzo del teléfono). Ordenar por
> nombre.

```sql
SELECT nombreP, descripcion, precio, stock
FROM Producto p INNER JOIN Detalle d ON (d.idProducto = p.idProducto) INNER JOIN Factura f ON (f.nroTicket = d.nroTicket) INNER JOIN Cliente c ON (c.idCliente = f.idCliente)
WHERE telefono <> 221 and p.idProducto not in (select p1.idProducto
from Producto p1 INNER JOIN Detalle d1 ON (d1.idProducto = p1.idProducto) INNER JOIN Factura f1 ON (f1.nroTicket = d1.nroTicket) INNER JOIN Cliente c1 ON (c1.idCliente = f1.idCliente)
where telefono = 221)
Order by p.nombrep;
```

### Ejercicio 5

> Listar para cada producto nombre, descripción, precio y cuantas veces fue vendido. Tenga en
> cuenta que puede no haberse vendido nunca el producto.

```sql
SELECT p.nombreP, p.descripcion, p.precio, COUNT(d.idProducto) AS vecesVendido
FROM Producto p LEFT JOIN Detalle d ON p.idProducto = d.idProducto
GROUP BY p.idProducto, p.nombreP, p.descripcion, p.precio;
```

### Ejercicio 6

> Listar nombre, apellido, DNI, teléfono y dirección de clientes que compraron los productos con
> nombre ‘prod1’ y ‘prod2’ pero nunca compraron el producto con nombre ‘prod3’.

```sql
((SELECT nombre, apellido, DNI, telefono, direccion
from Cliente
NATURAL JOIN Factura
NATURAL JOIN Detalle
NATURAL JOIN Producto
WHERE nombreP = "Prod1")

intersect

(SELECT c.nombre, c.apellido, c.DNI, c.telefono, c.direccion
from Cliente c
NATURAL JOIN Factura
NATURAL JOIN Detalle
NATURAL JOIN Producto p
WHERE p.nombreP = "Prod2"))

except

(SELECT c2.nombre, c2.apellido, c2.DNI, c2.telefono, c2.direccion
from Cliente c2
NATURAL JOIN Factura
NATURAL JOIN Detalle
NATURAL JOIN Producto p2
WHERE p2.nombreP = "Prod3");
```

### Ejercicio 7

> Listar nroTicket, total, fecha, hora y DNI del cliente, de aquellas facturas donde se haya
> comprado el producto ‘prod38’ o la factura tenga fecha de 2023.

```sql
SELECT nombreP, nroTicket, total, fecha, hora, dni
FROM Cliente c
NATURAL JOIN Factura
NATURAL JOIN Detalle
NATURAL JOIN Producto
WHERE nombreP = "prod38" or (fecha between 2023-01-01 and 2023-12-31);
```

### Ejercicio 8

> Agregar un cliente con los siguientes datos: nombre:’Jorge Luis’, apellido:’Castor’, DNI:
> 40578999, teléfono: ‘221-4400789’, dirección:’11 entre 500 y 501 nro:2587’ y el id de cliente: 500002. Se supone que el idCliente 500002 no existe.

```sql
INSERT INTO Cliente (nombre, apellido, DNI, telefono, direccion, idCliente)
VALUES ("Jorge Luis", "Castor", 40578999, "221-4400789", "11 entre 500 y 501 nro:2587", 500002)
```

### Ejercicio 9

> Listar nroTicket, total, fecha, hora para las facturas del cliente ´Jorge Pérez´ donde no haya
> comprado el producto ´Z´.

```sql
SELECT nroTicket, total, fecha, hora
FROM Cliente c
NATURAL JOIN Factura
NATURAL JOIN Detalle
NATURAL JOIN Producto
WHERE c.nombre = "Jorge" and c.apellido = "Perez"
      and idProducto NOT IN (select idProducto from Producto where nombreP = "Z");
```

### Ejercicio 10

> Listar DNI, apellido y nombre de clientes donde el monto total comprado, teniendo en cuenta
> todas sus facturas, supere $100000.

```sql
SELECT DNI, apellido, nombre
FROM Cliente c
	NATURAL JOIN Factura f
GROUP BY c.idCliente, c.dni, c.nombre, c.apellido
HAVING sum(f.total) > 100000;
```

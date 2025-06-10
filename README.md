# EXAMEN EXTENSO DE SQL - Casos Complicados y Técnicas Avanzadas

## ENUNCIADO Y CHULETA DE ESTUDIO

### 📚 **ESQUEMAS DE BASES DE DATOS**

#### 🚴 **Base de Datos CICLISMO**
```sql
EQUIPO(nomeq, director)
CICLISTA(dorsal, nombre, edad, nomeq)
ETAPA(netapa, km, salida, llegada, dorsal)
PUERTO(nompuerto, altura, pendiente, categoria, netapa, dorsal)
MAILLOT(codigo, tipo, premio, color)
LLEVAR(codigo, netapa, dorsal)
```

#### 🚗 **Base de Datos SEGUNDA MANO**
```sql
CONCESIONARIO(codi_con, nom, ciudad, director)
VENDEDOR(dni, nom, tlf, ventas, codi_con)
COCHE(matricula, marca, modelo, color, kms, precio, codi_con)
```

#### 🎵 **Base de Datos MÚSICA**
```sql
COMPANYIA(cod, nombre, dir, fax, tfno)
DISCO(cod, nombre, fecha, cod_comp, cod_gru)
GRUPO(cod, nombre, fecha, pais)
ARTISTA(dni, nombre)
CLUB(cod, nombre, sede, num, cod_gru)
CANCION(cod, titulo, duracion)
ESTA(cod, can)
PERTENECE(dni, cod, funcion)
```

#### 📚 **Base de Datos BIBLIOTECA**
```sql
AUTOR(autor_id, nombre, nacionalidad)
LIBRO(id_lib, titulo, año, varias_obras)
TEMA(tematica, descripcion)
OBRA(cod_ob, titulo, tematica)
AMIGO(num, nombre, telefono)
PRESTAMO(num, id_lib)
ESTA_EN(cod_ob, id_lib)
ESCRIBIR(cod_ob, autor_id)
```

---

## 🔥 **CASOS CRÍTICOS: NOT EXISTS vs NOT IN**

### ⚠️ **REGLA DE ORO:**
- **NOT IN**: Se rompe con valores NULL (devuelve UNKNOWN)
- **NOT EXISTS**: Funciona correctamente con NULL
- **LEFT JOIN + IS NULL**: Alternativa robusta

---

## 🎯 **CASOS LEFT JOIN ESENCIALES**

### 🔍 **Cuándo usar LEFT JOIN:**
1. **Incluir registros sin coincidencias** (ej: equipos sin ciclistas)
2. **Contar elementos que pueden ser cero**
3. **Evitar perder datos en agregaciones**

---

## 📝 **EXAMEN PRÁCTICO - 30 EJERCICIOS**

### **EJERCICIO 1 - DIVISIÓN RELACIONAL (★★★★★)**
**Obtener el nombre de los ciclistas que han ganado todos los puertos de una etapa y además han ganado esa misma etapa.**

```sql
SELECT c.nombre
FROM ciclista c 
INNER JOIN etapa e ON c.dorsal = e.dorsal
INNER JOIN puerto p ON e.netapa = p.netapa AND c.dorsal = p.dorsal
GROUP BY c.nombre, e.netapa
HAVING COUNT(DISTINCT p.nompuerto) = (
    SELECT COUNT(*)
    FROM puerto p2
    WHERE p2.netapa = e.netapa
);
```

---

### **EJERCICIO 2 - NOT EXISTS COMPLEJO (★★★★★)**
**Obtener el nombre de los equipos tal que sus ciclistas SOLO hayan ganado puertos de 1ª categoría.**

```sql
SELECT DISTINCT e.nomeq
FROM equipo e INNER JOIN ciclista c ON e.nomeq = c.nomeq
WHERE NOT EXISTS (
    SELECT 1
    FROM ciclista c2 INNER JOIN puerto p ON c2.dorsal = p.dorsal
    WHERE c2.nomeq = e.nomeq AND p.categoria != 1
);
```

---

### **EJERCICIO 3 - RANKING SIN LIMIT (★★★★)**
**Obtener el décimo club con mayor número de fans (debe haber solo 9 por encima de él).**

```sql
SELECT c.nombre, c.num
FROM club c
WHERE 9 = (
    SELECT COUNT(*)
    FROM club c2
    WHERE c.num < c2.num
);
```

---

### **EJERCICIO 4 - LEFT JOIN + HAVING (★★★★)**
**Obtener el nombre de los ciclistas que han ganado más de un puerto, indicando cuántos han ganado.**

```sql
SELECT c.nombre, COUNT(p.nompuerto)
FROM ciclista c LEFT JOIN puerto p ON c.dorsal = p.dorsal
GROUP BY c.dorsal
HAVING COUNT(p.nompuerto) > 1;
```

---

### **EJERCICIO 5 - NOT EXISTS CON MÚLTIPLES CONDICIONES (★★★★)**
**Obtener el nombre de las compañías discográficas que solo han trabajado con grupos españoles.**

```sql
SELECT DISTINCT c.nombre
FROM companyia c INNER JOIN disco d ON c.cod = d.cod_comp
WHERE NOT EXISTS (
    SELECT 1
    FROM companyia c2 INNER JOIN disco d2 ON c2.cod = d2.cod_comp
    INNER JOIN grupo g ON g.cod = d2.cod_gru
    WHERE c2.cod = c.cod AND g.pais != "España"
);
```

---

### **EJERCICIO 6 - TODOS vs ALGUNOS (★★★★★)**
**Obtener el nombre de los amigos que han leído todas las obras del autor 'RUKI'.**

```sql
SELECT a.nombre
FROM amigo a INNER JOIN prestamo p ON a.num = p.num
INNER JOIN esta_en ee ON p.id_lib = ee.id_lib
INNER JOIN escribir e ON e.cod_ob = ee.cod_ob
WHERE e.autor_id = "RUKI"
GROUP BY a.num
HAVING COUNT(DISTINCT e.cod_ob) = (
    SELECT COUNT(*)
    FROM escribir e2
    WHERE e2.autor_id = "RUKI"
);
```

---

### **EJERCICIO 7 - MÁXIMO CON >= ALL (★★★★)**
**¿Cuál es la compañía discográfica que más canciones ha grabado?**

```sql
SELECT c.nombre
FROM companyia c INNER JOIN disco d ON c.cod = d.cod_comp
INNER JOIN esta e ON e.cod = d.cod
GROUP BY c.cod
HAVING COUNT(e.can) >= ALL (
    SELECT COUNT(e2.can)
    FROM disco d2 INNER JOIN esta e2 ON e2.cod = d2.cod
    GROUP BY d2.cod_comp
);
```

---

### **EJERCICIO 8 - ETAPAS SIN PUERTOS (★★★★)**
**Obtener el número de etapa y la ciudad de salida de aquellas etapas que no tengan puertos de montaña.**

```sql
SELECT e.netapa, e.salida
FROM etapa e
WHERE NOT EXISTS (
    SELECT 1
    FROM puerto p
    WHERE p.netapa = e.netapa
);
```

---

### **EJERCICIO 9 - TODOS LOS MIEMBROS DE UN GRUPO (★★★★★)**
**Obtener el nombre de los equipos y la edad media de sus ciclistas de aquellos equipos cuya media de edad sea la máxima de todos los equipos.**

```sql
SELECT c.nomeq, AVG(c.edad)
FROM ciclista c
GROUP BY c.nomeq
HAVING AVG(c.edad) >= ALL (
    SELECT AVG(c2.edad)
    FROM ciclista c2
    GROUP BY c2.nomeq
);
```

---

### **EJERCICIO 10 - CONDICIÓN ÚNICA (★★★★★)**
**Obtener el título de la canción de mayor duración si es única.**

```sql
SELECT c.titulo, c.duracion
FROM cancion c
WHERE c.duracion = (SELECT MAX(duracion) FROM cancion)
GROUP BY c.duracion
HAVING COUNT(*) = 1;
```

---

### **EJERCICIO 11 - LEFT JOIN PARA INCLUIR TODOS (★★★★)**
**Obtener el nombre de todos los equipos indicando cuántos ciclistas tiene cada uno (incluyendo los que no tienen ninguno).**

```sql
SELECT e.nomeq, COUNT(c.dorsal)
FROM equipo e LEFT JOIN ciclista c ON e.nomeq = c.nomeq
GROUP BY e.nomeq;
```

---

### **EJERCICIO 12 - AUTORES SIN OBRAS (★★★★)**
**Obtener el nombre de los autores de los que no se tiene ninguna obra.**

```sql
SELECT a.nombre
FROM autor a
WHERE NOT EXISTS (
    SELECT 1
    FROM escribir e
    WHERE e.autor_id = a.autor_id
);
```

---

### **EJERCICIO 13 - MÚLTIPLES CONDICIONES EN HAVING (★★★★)**
**Obtener el nombre de los ciclistas que pertenezcan a un equipo que tenga más de cinco corredores y que hayan ganado alguna etapa, indicando cuántas etapas ha ganado.**

```sql
SELECT c.nombre, COUNT(e.netapa) as etapas_ganadas
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal
WHERE c.nomeq IN (
    SELECT c2.nomeq
    FROM ciclista c2
    GROUP BY c2.nomeq
    HAVING COUNT(c2.dorsal) > 5
)
GROUP BY c.dorsal;
```

---

### **EJERCICIO 14 - COMPARACIÓN CON SUBCONSULTA (★★★★)**
**Obtener los nombres de los puertos cuya altura es mayor que la media de altura de los puertos de 2ª categoría.**

```sql
SELECT p.nompuerto
FROM puerto p
WHERE p.altura > (
    SELECT AVG(p2.altura)
    FROM puerto p2
    WHERE p2.categoria = 2
);
```

---

### **EJERCICIO 15 - MÚLTIPLES NIVELES DE AGRUPACIÓN (★★★★★)**
**Obtener el nombre de los artistas que tengan la función de bajo en un único grupo y que además éste tenga más de dos miembros.**

```sql
SELECT a.nombre
FROM artista a INNER JOIN pertenece p ON a.dni = p.dni
WHERE p.funcion = "bajo"
AND p.cod IN (
    SELECT p2.cod
    FROM pertenece p2
    GROUP BY p2.cod
    HAVING COUNT(p2.dni) > 2
)
GROUP BY a.dni
HAVING COUNT(p.cod) = 1;
```

---

### **EJERCICIO 16 - ETAPAS CON CONDICIONES ESPECIALES (★★★★)**
**Obtener el valor del atributo netapa de aquellas etapas tales que todos los puertos que están en ellas tienen más de 700 metros de altura.**

```sql
SELECT e.netapa
FROM etapa e INNER JOIN puerto p ON e.netapa = p.netapa
GROUP BY e.netapa
HAVING MIN(p.altura) > 700;
```

---

### **EJERCICIO 17 - TODOS LOS DE UN TIPO (★★★★★)**
**Obtener el nombre y el director de los equipos tales que todos sus ciclistas son mayores de 26 años.**

```sql
SELECT e.nomeq, e.director
FROM equipo e INNER JOIN ciclista c ON e.nomeq = c.nomeq
GROUP BY e.nomeq
HAVING MIN(c.edad) > 26;
```

---

### **EJERCICIO 18 - SOLO UN TIPO DE CARACTERÍSTICA (★★★★★)**
**Obtener el nombre de los amigos que solo han leído obras de un autor.**

```sql
SELECT a.nombre
FROM amigo a INNER JOIN prestamo p ON a.num = p.num
INNER JOIN esta_en ee ON p.id_lib = ee.id_lib
INNER JOIN escribir e ON ee.cod_ob = e.cod_ob
GROUP BY a.num
HAVING COUNT(DISTINCT e.autor_id) = 1;
```

---

### **EJERCICIO 19 - NACIONALIDADES MENOS FRECUENTES (★★★★)**
**Obtener la nacionalidad (o nacionalidades) menos frecuentes entre los autores.**

```sql
SELECT a.nacionalidad, COUNT(a.autor_id)
FROM autor a
GROUP BY a.nacionalidad
HAVING COUNT(a.autor_id) <= ALL (
    SELECT COUNT(*)
    FROM autor a2
    GROUP BY a2.nacionalidad
);
```

---

### **EJERCICIO 20 - CICLISTA MÁS JOVEN CON CONDICIÓN (★★★★)**
**Obtener el nombre del ciclista más joven que ha ganado al menos una etapa.**

```sql
SELECT c.nombre, c.edad
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal
WHERE c.edad = (
    SELECT MIN(c2.edad)
    FROM ciclista c2 INNER JOIN etapa e2 ON c2.dorsal = e2.dorsal
);
```

---

### **EJERCICIO 21 - OBRAS CON MÚLTIPLES AUTORES (★★★★)**
**Obtener el título y el código de las obras que tengan más de un autor.**

```sql
SELECT o.titulo, o.cod_ob
FROM obra o INNER JOIN escribir e ON o.cod_ob = e.cod_ob
GROUP BY o.cod_ob
HAVING COUNT(e.autor_id) > 1;
```

---

### **EJERCICIO 22 - CONCESIONARIO CON MÁS VENTAS (★★★★)**
**Obtener el nombre del concesionario que tenga la mayor suma de ventas de sus vendedores.**

```sql
SELECT c.nom
FROM concesionario c INNER JOIN vendedor v ON c.codi_con = v.codi_con
GROUP BY c.codi_con
HAVING SUM(v.ventas) >= ALL (
    SELECT SUM(v2.ventas)
    FROM vendedor v2
    GROUP BY v2.codi_con
);
```

---

### **EJERCICIO 23 - ARTISTAS EN MÚLTIPLES GRUPOS (★★★★)**
**Obtener el nombre de los artistas que pertenecen a más de un grupo.**

```sql
SELECT a.nombre
FROM artista a INNER JOIN pertenece p ON a.dni = p.dni
GROUP BY a.dni
HAVING COUNT(DISTINCT p.cod) > 1;
```

---

### **EJERCICIO 24 - LIBROS CON TÍTULO Y MÚLTIPLES OBRAS (★★★★)**
**Obtener el título y el identificador de los libros que tengan título y más de dos obras, indicando el número de obras.**

```sql
SELECT l.titulo, l.id_lib, l.varias_obras
FROM libro l
WHERE l.titulo IS NOT NULL AND l.varias_obras > 2;
```

---

### **EJERCICIO 25 - EDAD MEDIA DE GANADORES (★★★★)**
**Obtener la edad media de los ciclistas que han ganado alguna etapa.**

```sql
SELECT AVG(c.edad)
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal;
```

---

### **EJERCICIO 26 - INSERT COMPLEJO (★★★)**
**Añadir un nuevo coche con los siguientes datos:**
- Matrícula: '9876ZXY', Precio: 25000, Código concesionario: 'VAL03'
- Marca: 'TESLA', Modelo: 'MODEL_3', Color: 'BLANCO', Kilómetros: 0

```sql
INSERT INTO coche (matricula, marca, modelo, color, kms, precio, codi_con) 
VALUES ('9876ZXY', 'TESLA', 'MODEL_3', 'BLANCO', 0, 25000, 'VAL03');
```

---

### **EJERCICIO 27 - UPDATE CON PORCENTAJE (★★★)**
**Incrementar un 15% el precio de todos los coches del concesionario de Valencia que tengan más de 50000 km.**

```sql
UPDATE coche c 
INNER JOIN concesionario con ON c.codi_con = con.codi_con
SET c.precio = c.precio * 1.15
WHERE con.ciudad = 'Valencia' AND c.kms > 50000;
```

---

### **EJERCICIO 28 - INSERT VENDEDOR (★★★)**
**Añadir un nuevo vendedor con DNI '12345678Z', nombre 'Ana García López', teléfono 966123456, ventas 85000, en el concesionario 'VAL01'.**

```sql
INSERT INTO vendedor (dni, nom, tlf, ventas, codi_con)
VALUES ('12345678Z', 'Ana García López', 966123456, 85000, 'VAL01');
```

---

### **EJERCICIO 29 - UPDATE PENDIENTE CON PORCENTAJE (★★★)**
**Incrementar un 10% la pendiente del puerto 'Aitana' al haberse cerrado la carretera que había en buen estado.**

```sql
UPDATE puerto p 
SET p.pendiente = p.pendiente * 1.10 
WHERE p.nompuerto = 'Aitana';
```

---

### **EJERCICIO 30 - RANKING TOP 3 SIN LIMIT (★★★★★)**
**Obtener el nombre de los ciclistas que tengan la edad de las 3 mayores edades que hay en la vuelta.**

```sql
SELECT c.nombre
FROM ciclista c
WHERE c.edad IN (
    SELECT DISTINCT edad
    FROM ciclista c2
    WHERE 3 > (
        SELECT COUNT(DISTINCT c3.edad)
        FROM ciclista c3
        WHERE c2.edad < c3.edad
    )
);
```

---



### **Los 3 ciclistas con las 3 mayores edades:**

**MÉTODO 1: Usando subconsulta con COUNT**
```sql
SELECT c.dorsal, c.nombre, c.edad, c.nomeq
FROM ciclista c
WHERE (
    SELECT COUNT(DISTINCT c2.edad)
    FROM ciclista c2
    WHERE c2.edad > c.edad
) < 3
ORDER BY c.edad DESC, c.nombre;
```

**MÉTODO 2: Usando subconsulta correlacionada con EXISTS**

```sql
SELECT c.dorsal, c.nombre, c.edad, c.nomeq
FROM ciclista c
WHERE 3 > (
    SELECT COUNT(*)
    FROM ciclista c2
    WHERE c2.edad > c.edad
)
ORDER BY c.edad DESC, c.nombre;
```


## 🎓 **CONCEPTOS CLAVE PARA RECORDAR**

### 🔸 **NOT EXISTS vs NOT IN:**
- **NOT EXISTS** es más seguro con NULLs
- **NOT IN** se rompe si hay NULLs en la subconsulta
- **NOT EXISTS** funciona mejor para lógica de "no existe ninguno que..."

### 🔸 **LEFT JOIN Casos Críticos:**
- Cuando necesitas incluir registros sin coincidencias
- Para contar elementos que pueden ser cero
- En agregaciones donde no quieres perder filas
- Siempre que el enunciado diga "todos" o "incluyendo los que no tienen"

### 🔸 **HAVING vs WHERE:**
- **WHERE**: filtra antes de agrupar (no puede usar funciones agregadas)
- **HAVING**: filtra después de agrupar (puede usar COUNT, SUM, AVG, etc.)

### 🔸 **Evitar LIMIT - Técnicas Alternativas:**
- Usar subconsultas con COUNT para rankings
- **>= ALL** para obtener máximos
- **<= ALL** para obtener mínimos
- Contar cuántos elementos hay por encima/debajo

### 🔸 **División Relacional - Patrón "TODOS":**
- Para consultas tipo "todos los X que cumplen Y"
- Usar COUNT con subconsulta
- Verificar que el conteo coincida exactamente
- Ejemplo: "ciclistas que han ganado TODOS los puertos de una etapa"

### 🔸 **Actualizar con Porcentajes:**
- Usar operaciones matemáticas: `precio * 1.15` (aumentar 15%)
- `precio * 0.85` (descuento del 15%)
- `precio * 1.10` (incremento del 10%)

### 🔸 **Subconsultas Correlacionadas:**
- La subconsulta hace referencia a la consulta externa
- Se ejecuta una vez por cada fila de la consulta externa
- Muy útiles con EXISTS/NOT EXISTS

### 🔸 **Funciones Agregadas Importantes:**
- **COUNT(DISTINCT campo)**: cuenta valores únicos
- **MIN/MAX**: para encontrar extremos con condiciones
- **AVG**: para medias con filtros específicos
- **SUM**: para totales por grupos

### 🔸 **Patrones de Consulta Críticos:**
1. **"Solo/Únicamente"** → NOT EXISTS o COUNT DISTINCT = 1
2. **"Todos"** → COUNT = subconsulta total o MIN/MAX con condiciones
3. **"Ninguno"** → NOT EXISTS o LEFT JOIN + IS NULL
4. **"Al menos uno"** → EXISTS o INNER JOIN
5. **"Más que todos"** → >= ALL
6. **"Menos que todos"** → <= ALL

# Consultas SQL Corregidas - Soluciones

## 📋 Índice
- [Consultas DDL](#consultas-ddl)
- [Consultas DML](#consultas-dml)

---

## ✅ Consultas DDL

### **1. Crear una tabla copia_agente con todos los datos de la tabla agente**
```sql
CREATE TABLE Copia_agente
SELECT * FROM agente;
```

### **1.2. Eliminar la columna dirección de la tabla agente**
```sql
ALTER TABLE agente DROP direccion;
```

### **1.3. Reducir un 10% las dietas de los agentes que no han puesto ninguna multa**
```sql
UPDATE agente
SET dietas = dietas * 0.90
WHERE codagente NOT IN (
    SELECT codagente 
    FROM multas 
    WHERE codagente IS NOT NULL
);
```

### **1.4. Restaurar las dietas originales de los agentes desde la tabla copia_agente**
```sql
UPDATE agente a INNER JOIN copia_agente ca 
ON a.codagente = ca.codagente
SET a.dietas = ca.dietas;
```

### **2. Añadir clave primaria a la tabla vehiculos usando el campo nmat**
```sql
ALTER TABLE vehiculos ADD PRIMARY KEY (nmat);
```

### **2.1. Añadir clave foránea en multas que referencie a agente**
```sql
ALTER TABLE multas ADD FOREIGN KEY(codagente) REFERENCES agente(codagente);
```

---

## ✅ Consultas DML

### **3. Obtener el nombre de los agentes y el importe de las multas menores a 200€**
```sql
SELECT a.nombre, m.importe
FROM agente a INNER JOIN multas m ON a.codagente = m.codagente
WHERE m.importe < 200;
```

### **4. Obtener el nombre de cada agente y el promedio de sus multas (incluyendo agentes sin multas)**
```sql
SELECT a.nombre, a.codagente, AVG(m.importe) as promedio_multas
FROM agente a LEFT JOIN multas m ON a.codagente = m.codagente 
GROUP BY a.codagente, a.nombre;
```

### **5. Obtener el nombre y promedio de multas del agente más joven que haya puesto más de 2 multas**
```sql
SELECT a.nombre, AVG(m.importe) as importeMedio
FROM agente a INNER JOIN multas m ON a.codagente = m.codagente
WHERE a.fnac = (SELECT MAX(a2.fnac) FROM agente a2)
GROUP BY a.codagente, a.nombre
HAVING COUNT(m.codmulta) > 2;
```

### **6. Obtener los agentes que han puesto más de 2 multas**
```sql
SELECT a.codagente, a.nombre, COUNT(m.codmulta) as total_multas
FROM agente a INNER JOIN multas m ON a.codagente = m.codagente
GROUP BY a.codagente, a.nombre
HAVING COUNT(m.codmulta) > 2;
```

### **7. Obtener las personas que tienen más de 2 multas en sus vehículos**
```sql
SELECT p.nombre, p.dni
FROM personas p INNER JOIN vehiculos v ON p.dni = v.dni_prop
INNER JOIN multas m ON m.nmat = v.nmat
GROUP BY p.dni, p.nombre
HAVING COUNT(m.codmulta) > 2;
```

### **8. Obtener la persona que tiene el mayor importe total en multas de todos sus vehículos**
```sql
SELECT p.nombre, p.dni, SUM(m.importe) as importe_total
FROM personas p INNER JOIN vehiculos v ON p.dni = v.dni_prop
INNER JOIN multas m ON m.nmat = v.nmat
GROUP BY p.dni, p.nombre
HAVING SUM(m.importe) >= ALL (
    SELECT SUM(m2.importe)
    FROM personas p2 INNER JOIN vehiculos v2 ON p2.dni = v2.dni_prop
    INNER JOIN multas m2 ON m2.nmat = v2.nmat 
    GROUP BY p2.dni
);
```

### **9. Obtener la persona que tiene el mayor número de vehículos**
```sql
SELECT p.nombre, COUNT(v.nmat) as total_vehiculos
FROM personas p INNER JOIN vehiculos v ON p.dni = v.dni_prop
GROUP BY p.dni, p.nombre
HAVING COUNT(v.nmat) >= ALL (
    SELECT COUNT(v2.nmat)
    FROM personas p2 INNER JOIN vehiculos v2 ON p2.dni = v2.dni_prop
    GROUP BY p2.dni
);
```

### **10. Obtener las comisarías con los 2 presupuestos más altos**
```sql
SELECT c.nombre, c.Presupuesto
FROM comisaria c
WHERE 2 > (SELECT COUNT(DISTINCT c2.Presupuesto)
           FROM comisaria c2
           WHERE c.Presupuesto < c2.Presupuesto);
```

### **11. Obtener los agentes cuyo promedio de multas es menor que el presupuesto de su comisaría**
```sql
SELECT a.codagente, a.nombre, AVG(m.importe) as promedio_multas, c.Presupuesto
FROM agente a INNER JOIN comisaria c ON a.Codcomisaria = c.codcomisaria
INNER JOIN multas m ON m.codagente = a.codagente
GROUP BY a.codagente, a.nombre, c.Presupuesto
HAVING AVG(m.importe) < c.Presupuesto;
```

### **12. Obtener vehículos que tienen tantas multas como agentes hay en la comisaría más pequeña**
```sql
SELECT m.nmat
FROM multas m 
GROUP BY m.nmat
HAVING COUNT(m.codmulta) = (
    SELECT COUNT(a.codagente)
    FROM agente a
    WHERE a.Codcomisaria = (SELECT MIN(Codcomisaria) FROM agente)
);
```


# Consultas SQL de ejemplo

### 1. Equipo que más etapas ha ganado

Obtener el nombre del equipo que ha ganado el mayor número de etapas.

```sql
/* victorias de cada equipo y selección del máximo sin LIMIT */
SELECT v.nombre, v.etapas_ganadas
FROM (
        SELECT  e.id_equipo,
                e.nombre,
                COUNT(*) AS etapas_ganadas
        FROM    Equipos  e
        INNER JOIN Etapas t ON t.id_equipo_ganador = e.id_equipo
        GROUP BY e.id_equipo, e.nombre
     ) AS v
WHERE NOT EXISTS (
        SELECT 1
        FROM (
                SELECT e2.id_equipo,
                       COUNT(*) AS total
                FROM   Equipos  e2
                INNER JOIN Etapas t2 ON t2.id_equipo_ganador = e2.id_equipo
                GROUP BY e2.id_equipo
             ) AS v2
        WHERE v2.total > v.etapas_ganadas
);
```

---

### 2. LEFT JOIN – lista de etapas con el nombre del equipo ganador (si lo hubiera)

```sql
SELECT  t.id_etapa,
        t.nombre AS etapa,
        e.nombre AS equipo_ganador
FROM    Etapas  t
LEFT JOIN Equipos e ON e.id_equipo = t.id_equipo_ganador
ORDER BY t.id_etapa;
```

---

### 3. Tres ciclistas con más edad

Mostrar los tres ciclistas más veteranos.

```sql
SELECT  c1.nombre,
        c1.fecha_nacimiento
FROM    Ciclistas c1
WHERE   2 >= (
        SELECT COUNT(*)
        FROM   Ciclistas c2
        WHERE  c2.fecha_nacimiento < c1.fecha_nacimiento
);
```

> *Si hay empates en la tercera posición, todos los ciclistas con esa edad serán incluidos.*

---

### 4. Ciclista más joven

#### a) Sin condición adicional

```sql
SELECT  c.nombre,
        c.fecha_nacimiento
FROM    Ciclistas c
WHERE   NOT EXISTS (
        SELECT 1
        FROM   Ciclistas c2
        WHERE  c2.fecha_nacimiento > c.fecha_nacimiento
);
```

#### b) Más joven que haya ganado alguna etapa

```sql
SELECT  c.nombre,
        c.fecha_nacimiento
FROM    Ciclistas c
WHERE   EXISTS (
          SELECT 1
          FROM   Resultados_Etapa r
          WHERE  r.id_ciclista = c.id_ciclista
            AND  r.posicion    = 1
       )
  AND   NOT EXISTS (
          SELECT 1
          FROM   Ciclistas c2
          WHERE  EXISTS (
                    SELECT 1
                    FROM   Resultados_Etapa r2
                    WHERE  r2.id_ciclista = c2.id_ciclista
                      AND  r2.posicion    = 1 )
            AND  c2.fecha_nacimiento > c.fecha_nacimiento
       );
```

---

### 5. Grupo que ha tocado en **todos** los festivales

```sql
SELECT  g.nombre
FROM    Grupos g
INNER JOIN Actuaciones a ON a.id_grupo = g.id_grupo
GROUP BY g.id_grupo, g.nombre
HAVING  COUNT(DISTINCT a.id_festival) = (SELECT COUNT(*) FROM Festivales);
```

---

### 6. Coche sancionado por **todos** los agentes de una misma comisaría (`:comisaria`)

```sql
SELECT  c.matricula
FROM    Coches c
WHERE   NOT EXISTS (
        SELECT 1
        FROM   Agentes ag
        WHERE  ag.id_comisaria = :comisaria
          AND NOT EXISTS (
                SELECT 1
                FROM   Multas m
                WHERE  m.id_coche  = c.id_coche
                  AND  m.id_agente = ag.id_agente
          )
);
```

---

### 7. Modificar una tabla

Añadir la columna **nacionalidad** en `Ciclistas`.

```sql
ALTER TABLE Ciclistas
ADD COLUMN nacionalidad VARCHAR(50);
```

---

### 8. Recuperar el precio de otra tabla

Calcular el importe de cada línea de pedido (`cantidad × precio`).

```sql
SELECT  dp.id_detalle,
        p.descripcion,
        dp.cantidad,
        p.precio,
        dp.cantidad * p.precio AS importe_linea
FROM    Detalle_Pedido dp
INNER JOIN Productos      p ON p.id_producto = dp.id_producto;
```

---

### 9. Coches con más de una multa en el mismo sitio

```sql
SELECT  c.matricula,
        m.lugar,
        COUNT(*) AS num_multas
FROM    Coches  c
INNER JOIN Multas m ON m.id_coche = c.id_coche
GROUP BY c.id_coche, c.matricula, m.lugar
HAVING  COUNT(*) > 1;
```

---

### 10. La canción más larga (solo si es única)

```sql
SELECT  c.titulo,
        c.duracion_seg
FROM    Canciones c
WHERE   NOT EXISTS (
          SELECT 1
          FROM   Canciones c2
          WHERE  c2.duracion_seg > c.duracion_seg
       )
  AND   NOT EXISTS (
          SELECT 1
          FROM   Canciones c3
          WHERE  c3.duracion_seg = c.duracion_seg
            AND  c3.id_cancion  <> c.id_cancion
       );
```

---

### 11. Compañías discográficas que **no** han trabajado con grupos españoles

```sql
SELECT  d.nombre
FROM    Discograficas d
WHERE   NOT EXISTS (
        SELECT 1
        FROM   Contratos  co
        INNER JOIN Grupos g ON g.id_grupo = co.id_grupo
        WHERE  co.id_discografica = d.id_discografica
          AND  g.pais_origen      = 'España'
);
```

---

### 12. Compañías discográficas que **solo** han trabajado con grupos españoles

```sql
SELECT  d.nombre
FROM    Discograficas d
WHERE   EXISTS (
        SELECT 1
        FROM   Contratos co
        WHERE  co.id_discografica = d.id_discografica
)
  AND  NOT EXISTS (
        SELECT 1
        FROM   Contratos  co
        INNER JOIN Grupos g ON g.id_grupo = co.id_grupo
        WHERE  co.id_discografica = d.id_discografica
          AND  g.pais_origen      <> 'España'
);
```




### CREAR COPIA DE SEGURIDAD DE LA TABLA CONCIERTOS


```sql
-- Crear tabla de respaldo con todos los datos originales
CREATE TABLE copia_conciertos
SELECT * FROM conciertos;

-- Verificar que la copia se creó correctamente
SELECT COUNT(*) as registros_originales FROM conciertos;
SELECT COUNT(*) as registros_copia FROM copia_conciertos;

```


### AÑADIR CLAVE FORÁNEA: cod_conc EN TABLA VENDEDOR → CONCIERTOS

```sql

-- Primero verificar que existe la columna cod_conc en vendedor
-- Si no existe, crearla primero
ALTER TABLE vendedor 
ADD COLUMN cod_conc INT;

-- Añadir la clave foránea
ALTER TABLE vendedor 
ADD CONSTRAINT fk_vendedor_concierto 
FOREIGN KEY (cod_conc) REFERENCES conciertos(cod_concierto);

```

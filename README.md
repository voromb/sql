EXAMEN EXTENSO DE SQL - Casos Complicados y Técnicas Avanzadas
ENUNCIADO Y CHULETA DE ESTUDIO
📚 ESQUEMAS DE BASES DE DATOS
🚴 Base de Datos CICLISMO
sqlEQUIPO(nomeq, director)
CICLISTA(dorsal, nombre, edad, nomeq)
ETAPA(netapa, km, salida, llegada, dorsal)
PUERTO(nompuerto, altura, pendiente, categoria, netapa, dorsal)
MAILLOT(codigo, tipo, premio, color)
LLEVAR(codigo, netapa, dorsal)
🚗 Base de Datos SEGUNDA MANO
sqlCONCESIONARIO(codi_con, nom, ciudad, director)
VENDEDOR(dni, nom, tlf, ventas, codi_con)
COCHE(matricula, marca, modelo, color, kms, precio, codi_con)
🎵 Base de Datos MÚSICA
sqlCOMPANYIA(cod, nombre, dir, fax, tfno)
DISCO(cod, nombre, fecha, cod_comp, cod_gru)
GRUPO(cod, nombre, fecha, pais)
ARTISTA(dni, nombre)
CLUB(cod, nombre, sede, num, cod_gru)
CANCION(cod, titulo, duracion)
ESTA(cod, can)
PERTENECE(dni, cod, funcion)
📚 Base de Datos BIBLIOTECA
sqlAUTOR(autor_id, nombre, nacionalidad)
LIBRO(id_lib, titulo, año, varias_obras)
TEMA(tematica, descripcion)
OBRA(cod_ob, titulo, tematica)
AMIGO(num, nombre, telefono)
PRESTAMO(num, id_lib)
ESTA_EN(cod_ob, id_lib)
ESCRIBIR(cod_ob, autor_id)

🔥 CASOS CRÍTICOS: NOT EXISTS vs NOT IN
⚠️ REGLA DE ORO:

NOT IN: Se rompe con valores NULL (devuelve UNKNOWN)
NOT EXISTS: Funciona correctamente con NULL
LEFT JOIN + IS NULL: Alternativa robusta


🎯 CASOS LEFT JOIN ESENCIALES
🔍 Cuándo usar LEFT JOIN:

Incluir registros sin coincidencias (ej: equipos sin ciclistas)
Contar elementos que pueden ser cero
Evitar perder datos en agregaciones


📝 EXAMEN PRÁCTICO - 30 EJERCICIOS
EJERCICIO 1 - DIVISIÓN RELACIONAL (★★★★★)
Obtener el nombre de los ciclistas que han ganado todos los puertos de una etapa y además han ganado esa misma etapa.
sqlSELECT c.nombre
FROM ciclista c 
INNER JOIN etapa e ON c.dorsal = e.dorsal
INNER JOIN puerto p ON e.netapa = p.netapa AND c.dorsal = p.dorsal
GROUP BY c.nombre, e.netapa
HAVING COUNT(DISTINCT p.nompuerto) = (
    SELECT COUNT(*)
    FROM puerto p2
    WHERE p2.netapa = e.netapa
);

EJERCICIO 2 - NOT EXISTS COMPLEJO (★★★★★)
Obtener el nombre de los equipos tal que sus ciclistas SOLO hayan ganado puertos de 1ª categoría.
sqlSELECT DISTINCT e.nomeq
FROM equipo e INNER JOIN ciclista c ON e.nomeq = c.nomeq
WHERE NOT EXISTS (
    SELECT 1
    FROM ciclista c2 INNER JOIN puerto p ON c2.dorsal = p.dorsal
    WHERE c2.nomeq = e.nomeq AND p.categoria != 1
);

EJERCICIO 3 - RANKING SIN LIMIT (★★★★)
Obtener el décimo club con mayor número de fans (debe haber solo 9 por encima de él).
sqlSELECT c.nombre, c.num
FROM club c
WHERE 9 = (
    SELECT COUNT(*)
    FROM club c2
    WHERE c.num < c2.num
);

EJERCICIO 4 - LEFT JOIN + HAVING (★★★★)
Obtener el nombre de los ciclistas que han ganado más de un puerto, indicando cuántos han ganado.
sqlSELECT c.nombre, COUNT(p.nompuerto)
FROM ciclista c LEFT JOIN puerto p ON c.dorsal = p.dorsal
GROUP BY c.dorsal
HAVING COUNT(p.nompuerto) > 1;

EJERCICIO 5 - NOT EXISTS CON MÚLTIPLES CONDICIONES (★★★★)
Obtener el nombre de las compañías discográficas que solo han trabajado con grupos españoles.
sqlSELECT DISTINCT c.nombre
FROM companyia c INNER JOIN disco d ON c.cod = d.cod_comp
WHERE NOT EXISTS (
    SELECT 1
    FROM companyia c2 INNER JOIN disco d2 ON c2.cod = d2.cod_comp
    INNER JOIN grupo g ON g.cod = d2.cod_gru
    WHERE c2.cod = c.cod AND g.pais != "España"
);

EJERCICIO 6 - TODOS vs ALGUNOS (★★★★★)
Obtener el nombre de los amigos que han leído todas las obras del autor 'RUKI'.
sqlSELECT a.nombre
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

EJERCICIO 7 - MÁXIMO CON >= ALL (★★★★)
¿Cuál es la compañía discográfica que más canciones ha grabado?
sqlSELECT c.nombre
FROM companyia c INNER JOIN disco d ON c.cod = d.cod_comp
INNER JOIN esta e ON e.cod = d.cod
GROUP BY c.cod
HAVING COUNT(e.can) >= ALL (
    SELECT COUNT(e2.can)
    FROM disco d2 INNER JOIN esta e2 ON e2.cod = d2.cod
    GROUP BY d2.cod_comp
);

EJERCICIO 8 - ETAPAS SIN PUERTOS (★★★★)
Obtener el número de etapa y la ciudad de salida de aquellas etapas que no tengan puertos de montaña.
sqlSELECT e.netapa, e.salida
FROM etapa e
WHERE NOT EXISTS (
    SELECT 1
    FROM puerto p
    WHERE p.netapa = e.netapa
);

EJERCICIO 9 - TODOS LOS MIEMBROS DE UN GRUPO (★★★★★)
Obtener el nombre de los equipos y la edad media de sus ciclistas de aquellos equipos cuya media de edad sea la máxima de todos los equipos.
sqlSELECT c.nomeq, AVG(c.edad)
FROM ciclista c
GROUP BY c.nomeq
HAVING AVG(c.edad) >= ALL (
    SELECT AVG(c2.edad)
    FROM ciclista c2
    GROUP BY c2.nomeq
);

EJERCICIO 10 - CONDICIÓN ÚNICA (★★★★★)
Obtener el título de la canción de mayor duración si es única.
sqlSELECT c.titulo, c.duracion
FROM cancion c
WHERE c.duracion = (SELECT MAX(duracion) FROM cancion)
GROUP BY c.duracion
HAVING COUNT(*) = 1;

EJERCICIO 11 - LEFT JOIN PARA INCLUIR TODOS (★★★★)
Obtener el nombre de todos los equipos indicando cuántos ciclistas tiene cada uno (incluyendo los que no tienen ninguno).
sqlSELECT e.nomeq, COUNT(c.dorsal)
FROM equipo e LEFT JOIN ciclista c ON e.nomeq = c.nomeq
GROUP BY e.nomeq;

EJERCICIO 12 - AUTORES SIN OBRAS (★★★★)
Obtener el nombre de los autores de los que no se tiene ninguna obra.
sqlSELECT a.nombre
FROM autor a
WHERE NOT EXISTS (
    SELECT 1
    FROM escribir e
    WHERE e.autor_id = a.autor_id
);

EJERCICIO 13 - MÚLTIPLES CONDICIONES EN HAVING (★★★★)
Obtener el nombre de los ciclistas que pertenezcan a un equipo que tenga más de cinco corredores y que hayan ganado alguna etapa, indicando cuántas etapas ha ganado.
sqlSELECT c.nombre, COUNT(e.netapa) as etapas_ganadas
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal
WHERE c.nomeq IN (
    SELECT c2.nomeq
    FROM ciclista c2
    GROUP BY c2.nomeq
    HAVING COUNT(c2.dorsal) > 5
)
GROUP BY c.dorsal;

EJERCICIO 14 - COMPARACIÓN CON SUBCONSULTA (★★★★)
Obtener los nombres de los puertos cuya altura es mayor que la media de altura de los puertos de 2ª categoría.
sqlSELECT p.nompuerto
FROM puerto p
WHERE p.altura > (
    SELECT AVG(p2.altura)
    FROM puerto p2
    WHERE p2.categoria = 2
);

EJERCICIO 15 - MÚLTIPLES NIVELES DE AGRUPACIÓN (★★★★★)
Obtener el nombre de los artistas que tengan la función de bajo en un único grupo y que además éste tenga más de dos miembros.
sqlSELECT a.nombre
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

EJERCICIO 16 - ETAPAS CON CONDICIONES ESPECIALES (★★★★)
Obtener el valor del atributo netapa de aquellas etapas tales que todos los puertos que están en ellas tienen más de 700 metros de altura.
sqlSELECT e.netapa
FROM etapa e INNER JOIN puerto p ON e.netapa = p.netapa
GROUP BY e.netapa
HAVING MIN(p.altura) > 700;

EJERCICIO 17 - TODOS LOS DE UN TIPO (★★★★★)
Obtener el nombre y el director de los equipos tales que todos sus ciclistas son mayores de 26 años.
sqlSELECT e.nomeq, e.director
FROM equipo e INNER JOIN ciclista c ON e.nomeq = c.nomeq
GROUP BY e.nomeq
HAVING MIN(c.edad) > 26;

EJERCICIO 18 - SOLO UN TIPO DE CARACTERÍSTICA (★★★★★)
Obtener el nombre de los amigos que solo han leído obras de un autor.
sqlSELECT a.nombre
FROM amigo a INNER JOIN prestamo p ON a.num = p.num
INNER JOIN esta_en ee ON p.id_lib = ee.id_lib
INNER JOIN escribir e ON ee.cod_ob = e.cod_ob
GROUP BY a.num
HAVING COUNT(DISTINCT e.autor_id) = 1;

EJERCICIO 19 - NACIONALIDADES MENOS FRECUENTES (★★★★)
Obtener la nacionalidad (o nacionalidades) menos frecuentes entre los autores.
sqlSELECT a.nacionalidad, COUNT(a.autor_id)
FROM autor a
GROUP BY a.nacionalidad
HAVING COUNT(a.autor_id) <= ALL (
    SELECT COUNT(*)
    FROM autor a2
    GROUP BY a2.nacionalidad
);

EJERCICIO 20 - CICLISTA MÁS JOVEN CON CONDICIÓN (★★★★)
Obtener el nombre del ciclista más joven que ha ganado al menos una etapa.
sqlSELECT c.nombre, c.edad
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal
WHERE c.edad = (
    SELECT MIN(c2.edad)
    FROM ciclista c2 INNER JOIN etapa e2 ON c2.dorsal = e2.dorsal
);

EJERCICIO 21 - OBRAS CON MÚLTIPLES AUTORES (★★★★)
Obtener el título y el código de las obras que tengan más de un autor.
sqlSELECT o.titulo, o.cod_ob
FROM obra o INNER JOIN escribir e ON o.cod_ob = e.cod_ob
GROUP BY o.cod_ob
HAVING COUNT(e.autor_id) > 1;

EJERCICIO 22 - CONCESIONARIO CON MÁS VENTAS (★★★★)
Obtener el nombre del concesionario que tenga la mayor suma de ventas de sus vendedores.
sqlSELECT c.nom
FROM concesionario c INNER JOIN vendedor v ON c.codi_con = v.codi_con
GROUP BY c.codi_con
HAVING SUM(v.ventas) >= ALL (
    SELECT SUM(v2.ventas)
    FROM vendedor v2
    GROUP BY v2.codi_con
);

EJERCICIO 23 - ARTISTAS EN MÚLTIPLES GRUPOS (★★★★)
Obtener el nombre de los artistas que pertenecen a más de un grupo.
sqlSELECT a.nombre
FROM artista a INNER JOIN pertenece p ON a.dni = p.dni
GROUP BY a.dni
HAVING COUNT(DISTINCT p.cod) > 1;

EJERCICIO 24 - LIBROS CON TÍTULO Y MÚLTIPLES OBRAS (★★★★)
Obtener el título y el identificador de los libros que tengan título y más de dos obras, indicando el número de obras.
sqlSELECT l.titulo, l.id_lib, l.varias_obras
FROM libro l
WHERE l.titulo IS NOT NULL AND l.varias_obras > 2;

EJERCICIO 25 - EDAD MEDIA DE GANADORES (★★★★)
Obtener la edad media de los ciclistas que han ganado alguna etapa.
sqlSELECT AVG(c.edad)
FROM ciclista c INNER JOIN etapa e ON c.dorsal = e.dorsal;

EJERCICIO 26 - INSERT COMPLEJO (★★★)
Añadir un nuevo coche con los siguientes datos:

Matrícula: '9876ZXY', Precio: 25000, Código concesionario: 'VAL03'
Marca: 'TESLA', Modelo: 'MODEL_3', Color: 'BLANCO', Kilómetros: 0

sqlINSERT INTO coche (matricula, marca, modelo, color, kms, precio, codi_con) 
VALUES ('9876ZXY', 'TESLA', 'MODEL_3', 'BLANCO', 0, 25000, 'VAL03');

EJERCICIO 27 - UPDATE CON PORCENTAJE (★★★)
Incrementar un 15% el precio de todos los coches del concesionario de Valencia que tengan más de 50000 km.
sqlUPDATE coche c 
INNER JOIN concesionario con ON c.codi_con = con.codi_con
SET c.precio = c.precio * 1.15
WHERE con.ciudad = 'Valencia' AND c.kms > 50000;

EJERCICIO 28 - INSERT VENDEDOR (★★★)
Añadir un nuevo vendedor con DNI '12345678Z', nombre 'Ana García López', teléfono 966123456, ventas 85000, en el concesionario 'VAL01'.
sqlINSERT INTO vendedor (dni, nom, tlf, ventas, codi_con)
VALUES ('12345678Z', 'Ana García López', 966123456, 85000, 'VAL01');

EJERCICIO 29 - UPDATE PENDIENTE CON PORCENTAJE (★★★)
Incrementar un 10% la pendiente del puerto 'Aitana' al haberse cerrado la carretera que había en buen estado.
sqlUPDATE puerto p 
SET p.pendiente = p.pendiente * 1.10 
WHERE p.nompuerto = 'Aitana';

EJERCICIO 30 - RANKING TOP 3 SIN LIMIT (★★★★★)
Obtener el nombre de los ciclistas que tengan la edad de las 3 mayores edades que hay en la vuelta.
sqlSELECT c.nombre
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

🎓 CONCEPTOS CLAVE PARA RECORDAR
🔸 NOT EXISTS vs NOT IN:

NOT EXISTS es más seguro con NULLs
NOT IN se rompe si hay NULLs en la subconsulta
NOT EXISTS funciona mejor para lógica de "no existe ninguno que..."

🔸 LEFT JOIN Casos Críticos:

Cuando necesitas incluir registros sin coincidencias
Para contar elementos que pueden ser cero
En agregaciones donde no quieres perder filas
Siempre que el enunciado diga "todos" o "incluyendo los que no tienen"

🔸 HAVING vs WHERE:

WHERE: filtra antes de agrupar (no puede usar funciones agregadas)
HAVING: filtra después de agrupar (puede usar COUNT, SUM, AVG, etc.)

🔸 Evitar LIMIT - Técnicas Alternativas:

Usar subconsultas con COUNT para rankings
>= ALL para obtener máximos
<= ALL para obtener mínimos
Contar cuántos elementos hay por encima/debajo

🔸 División Relacional - Patrón "TODOS":

Para consultas tipo "todos los X que cumplen Y"
Usar COUNT con subconsulta
Verificar que el conteo coincida exactamente
Ejemplo: "ciclistas que han ganado TODOS los puertos de una etapa"

🔸 Actualizar con Porcentajes:

Usar operaciones matemáticas: precio * 1.15 (aumentar 15%)
precio * 0.85 (descuento del 15%)
precio * 1.10 (incremento del 10%)

🔸 Subconsultas Correlacionadas:

La subconsulta hace referencia a la consulta externa
Se ejecuta una vez por cada fila de la consulta externa
Muy útiles con EXISTS/NOT EXISTS

🔸 Funciones Agregadas Importantes:

COUNT(DISTINCT campo): cuenta valores únicos
MIN/MAX: para encontrar extremos con condiciones
AVG: para medias con filtros específicos
SUM: para totales por grupos

🔸 Patrones de Consulta Críticos:

"Solo/Únicamente" → NOT EXISTS o COUNT DISTINCT = 1
"Todos" → COUNT = subconsulta total o MIN/MAX con condiciones
"Ninguno" → NOT EXISTS o LEFT JOIN + IS NULL
"Al menos uno" → EXISTS o INNER JOIN
"Más que todos" → >= ALL
"Menos que todos" → <= ALL


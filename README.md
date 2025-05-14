# gmod-lua-performance
Una comparación simple de optimizaciones de rendimiento para gLUA

Estos son algunos benchmarks probados en un servidor de Garry's Mod inactivo con DarkRP y solo 1 jugador en línea.
Algunos de estos benchmarks, cuando fue posible, también se probaron en una consola LUA5.3 en Debian 10. A menos que se indique lo contrario, todas las pruebas se realizaron en un servidor gmod de 32 bits en linux.  

Estos resultados de benchmarks no significan nada en comparación con otros resultados de benchmarks. Tus números probablemente se verán completamente diferentes a los míos. Por favor compara tus resultados de benchmarks solo con tus propios números, ya que cada pequeño cambio en el hardware altera notablemente el resultado.

**El objetivo de esta colección de benchmarks es mostrar la ganancia real de rendimiento al implementar "Consejos de Codificación" de la web.**

**Cualquiera puede probar el código por sí mismo y ver la diferencia. El código fuente siempre está enlazado al final de cada comparación.**

Las fuentes de algunos de los "Consejos de Codificación" son:

[lua-users.org](http://lua-users.org/wiki/OptimisationCodingTips)  
[lua.org](https://www.lua.org/gems/sample.pdf)  
[stackoverflow.com](https://stackoverflow.com/questions/154672/what-can-i-do-to-increase-the-performance-of-a-lua-program/12865406#12865406)  

Si quieres leer más sobre gmod lua, también puedes visitar mi wiki en [luctus.at/wiki](https://luctus.at/wiki/)


# Prólogo

Garry's Mod no utiliza solo lua. Utiliza lua-jit.  
Esto significa que los siguientes benchmarks muestran, cuando es aplicable, también una comparación entre lua y lua-jit.  
Muchos de los consejos de rendimiento en internet están pensados para lua, no para luajit.

Un ejemplo de lua vs lua-jit con el ejemplo de "multiplicación vs división":

    $ time luajit-2.1.0-beta3 mult_vs_div.lua
    Multiplicación: 0.00111624
    División: 0.00110691

    real    0m1.255s
    user    0m1.054s
    sys     0m0.182s

    $ time lua5.3 mult_vs_div.lua
    Multiplicación: 0.0284177
    División: 0.03088507

    real    0m21.773s
    user    0m21.623s
    sys     0m0.120s

Si encuentras algún error, equivocación o resultados completamente diferentes en tus pruebas, por favor crea un pull request.  
Quiero que esta página sea lo más precisa posible para que todos puedan ver y aprender qué realmente beneficia el rendimiento de tu código.



# Almacenamiento en caché de funciones en HUDPaint

Resumen: Si usas una función frecuentemente en un hook HUDPaint (como LocalPlayer()), recomiendo almacenarla en caché al principio.

Esta es una comparación de los siguientes fragmentos de código. La pregunta es: "¿Deberías almacenar en caché una función que solo usas 3 veces?".

Versión no optimizada:

```lua
draw.RoundedBox(0, LocalPlayer():Health(), 500, 500, 500, color_white )
draw.RoundedBox(0, LocalPlayer():Health(), 200, 200, 200, color_white )
draw.RoundedBox(0, LocalPlayer():Health(), 10, 10, 10, color_white )
```

y la versión mejorada:

```lua
local lph = LocalPlayer():Health()
draw.RoundedBox(0, lph, 500, 500, 500, color_white )
draw.RoundedBox(0, lph, 200, 200, 200, color_white )
draw.RoundedBox(0, lph, 10, 10, 10, color_white )
```

Resultados (10000 mediciones de tiempo por frame):

    Sin caché:  0.00001756282000116   (1.756282000116e-05)
    Con caché:      0.000006777780000516  (6.777780000516e-06)

Código: [files/hudpaint_3call_cache.lua](files/hudpaint_3call_cache.lua)



# Almacenamiento en caché de la función LocalPlayer

Resumen: Es más rápido almacenar en caché el resultado de `LocalPlayer()` y sobrescribir la función para que siempre devuelva este valor.

Es bastante autoexplicativo. `LocalPlayer()` siempre te devuelve a ti (el jugador), pero toma más tiempo que si simplemente almacenaras la entidad del jugador en caché y la devolvieras cada vez.  
Revisa el código para más información.

Resultados:

    --- Benchmark completado
    En Cliente
    repeticiones	20	vueltas	50000
    LocalPlayer original	7.4946100011402e-08
    fastLocalPlayer	6.2921299662776e-08

Código: [files/localplayer_cache.lua](files/localplayer_cache.lua)



# Almacenamiento en caché de LocalPlayer con "or"

Resumen: Es ligeramente más rápido usar "ply = ply or LocalPlayer()" que usar siempre LocalPlayer(), pero la ganancia de tiempo es casi imperceptible.

Bastante autoexplicativo. Esto almacena LocalPlayer() en la primera ejecución y luego reutiliza ese valor.  
Consulta el código para más información.

Resultados:

    --- Benchmark completado
    En Cliente
    repeticiones	20	vueltas	10000
    Con caché	5.8824900024945e-07
    Sin caché 	6.021605001348e-07

Código: [files/localplayer_cached_or.lua](files/localplayer_cached_or.lua)



# Angle:Zero() vs Angle() nuevo

Resumen: Es más rápido reutilizar el mismo objeto Angle y reiniciarlo a cero antes de usarlo que crear un nuevo objeto Angle cada vez.

El wiki lo establece como un hecho, y es cierto. Te ahorra la sobrecarga de crear un nuevo objeto y recolectar el anterior.  
Consulta el código para más información.

Resultados:

    --- Benchmark completado
    En Servidor
    repeticiones	30	vueltas	5000
    Reiniciado	1.3485066712595e-07
    Crear nuevo	2.3365133318293e-07
    --- Benchmark completado
    En Cliente
    repeticiones	30	vueltas	5000
    Reiniciado	1.1277866673481e-07
    Crear nuevo	2.4979333345679e-07

Código: [files/angle_zero_vs_new.lua](files/angle_zero_vs_new.lua)



# Almacenamiento en caché de función SteamID

Resumen: Es más rápido almacenar en caché el resultado de `ply:SteamID()` y sobrescribir la función para que siempre devuelva este valor para un jugador desde una tabla de caché.

Bastante autoexplicativo. Igual que "Almacenamiento en caché de función LocalPlayer" mencionado anteriormente.

Resultados:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	50000
    SteamID Clásico	4.6951399998613e-07
    SteamID en Caché	7.0017300033214e-08
    --- Benchmark completado
    En Cliente
    repeticiones	20	vueltas	50000
    SteamID Clásico	5.2189459993372e-07
    SteamID en Caché	6.247109999822e-08

Código: [files/steamid_cache.lua](files/steamid_cache.lua)



# GetNWString vs getDarkRPVar

Resumen: getDarkRPVar siempre es más rápido en el servidor, mientras que en el cliente parece ser aleatorio. Recomendaría usar getDarkRPVar, ya que está más optimizado para el uso de red en comparación con el tráfico constante de red cada 10 segundos de NW1.

Esta es una comparación entre `ply:GetNWString("job")` y `ply:getDarkRPVar("job")`.

Resultados:

    --- Benchmark completado
    repeticiones	10	vueltas	10000
    En Servidor
    darkrpvar	1.2346400001078e-07
    gmodnwvar	2.0404500001689e-07
    --- Benchmark completado
    repeticiones	10	vueltas	10000
    En Cliente
    darkrpvar	2.4343500003823e-07
    gmodnwvar	2.036369999766e-07

Como se puede ver, el resultado difiere entre cliente y servidor. El resultado en el cliente siempre varía cada vez que ejecuto estas pruebas. A veces, en el cliente, gmodnwvar es un 25% más rápido y otras veces es un 300% más lento que darkrpvar.

Código: [files/nwvar_vs_darkrpvar.lua](files/nwvar_vs_darkrpvar.lua)



# Tabla de Configuración vs Variable

Resumen: Usar una tabla es más lento que usar una variable.

Esta es una comparación entre usar una tabla de configuración como `myaddon.config.color =` y una simple variable como `myaddon_config_color =`. Esto existe principalmente porque algunas personas no me creen.

El resultado:

    --SERVIDOR
    tabla=	1.9999993128295e-07
    variable=	1.0000007932831e-07
    --CLIENTE
    tabla=	1.0000007932831e-07
    variable=	0

Como se puede ver arriba, usar una sola variable en lugar de una estructura de tabla es más rápido.


## Explicación

Obtener una variable es tan simple como "obtenerla". Pero con una tabla tienes que obtener la tabla y luego el elemento dentro de la tabla.  
Puedes inspeccionar esto con el comando luac de lua, por ejemplo `luac5.4 -l archivo.lua`.  
Un ejemplo: `print(abc)` vs `print(a.b.c)`

```
$ luac5.4 -l var.lua
[...]
        GETTABUP        0 0 2   ; _ENV "print"
        GETTABUP        1 0 0   ; _ENV "abc"
        CALL            0 2 1   ; 1 in 0 out
        RETURN          0 1 1   ; 0 out

$ luac5.4 -l tab.lua
[...]
        GETTABUP        0 0 4   ; _ENV "print"
        GETTABUP        1 0 0   ; _ENV "a"
        GETFIELD        1 1 1   ; "b"
        GETFIELD        1 1 2   ; "c"
        CALL            0 2 1   ; 1 in 0 out
        RETURN          0 1 1   ; 0 out
```

Como puedes observar arriba, la variante de tabla simplemente tiene más ejecuciones y por lo tanto es más lenta tanto en teoría como en práctica como se mostró anteriormente.

Código: [files/table_vs_variable.lua](files/table_vs_variable.lua)



# pairs vs ipairs vs for i=1

Resumen: No son tan diferentes como la gente cree. Pairs es más lento porque también funciona con tablas no secuenciales.

Resultados (10 pruebas de 100 ejecuciones cada una, calculando la suma de una tabla de 10,000 números):

    pairs 	8.8780000373845e-07
    ipairs	2.4300000836774e-07
    for #t	1.639000013256e-07


Código: [files/pairs_vs_ipairs_vs_for.lua](files/pairs_vs_ipairs_vs_for.lua)



# Comparación de iteradores para player.GetAll()

Resumen: El más rápido es el Iterador, pero yo (y el wiki) no lo recomendaría porque es propenso a errores. Usar ipairs es una forma muy legible, fácil de usar y rápida de iterar sobre todos los jugadores.

Resultados:

    pairs 	2.7375729999562e-06
    ipairs	2.5955539999833e-06
    for #t	2.5866950000025e-06
    iter 	1.7860300003008e-07

Código: [files/player_getall_compare.lua](files/player_getall_compare.lua)



# Iteración en tablas de strings vs números

Resumen: Las tablas indexadas numéricamente y secuenciales son siempre más rápidas gracias a que ipairs y los bucles for pueden optimizar la iteración. No hay diferencia en velocidad según la longitud de los strings usados como índice. Las tablas con números secuenciales son más rápidas que las no secuenciales.

Con una tabla de 100 elementos y 1000 ejecuciones para cada una de las 5 pruebas:

    pairs con strings cortos	1.016059995527e-06
    pairs con strings largos	9.8490000327729e-07
    pairs con enteros secuenciales	8.060999940426e-07
    ipairs con enteros secuenciales	2.9101999789418e-07
    pairs con enteros no ordenados	1.0006600032284e-06

Hay poca diferencia entre tablas con strings largos o cortos. La principal diferencia de velocidad ocurre con tablas indexadas numéricamente de forma secuencial, que pueden usar `ipairs` para ser más rápidas que usando `pairs`.

Código: [files/string_vs_number_table.lua](files/string_vs_number_table.lua)



# Distance vs DistToSqr

Resumen: La diferencia de tiempo entre Distance y DistToSqr es casi imperceptible.

Resultados (promedio de 10000 cálculos):

    Distance:   0.00000026040000557259 (2.6040000557259e-07)
    DistToSqr:  0.00000025130000176432 (2.5130000176432e-07)

Código: [files/distance_vs_disttosqr.lua](files/distance_vs_disttosqr.lua)



# Operador ternario vs if

Resumen: Toman la misma cantidad de tiempo. La diferencia fluctúa aleatoriamente con variaciones mínimas en cada prueba.

Pruebas realizadas:  
`a = t and 7 or 1` vs `a=1 if t then a=7 end`

Resultados (promedio de 1,000,000 cálculos):

    if:      4.1537399886693e-08
    ternario: 4.168440006697e-08

Código: [files/ternary_vs_if.lua](files/ternary_vs_if.lua)



# table.insert vs asignación directa

Resumen: La diferencia de tiempo entre usar `table.insert(myTable,9)` y `myTable[#myTable+1] = 9` es casi imperceptible, ya que los resultados fueron prácticamente idénticos (incluso probado en consola Lua5.3).

Resultados (insertando 1 millón de valores):

    table.insert:         0.066328400000003
    table.insert local:   0.07047399999999
    [#res+1]:             0.066876700000023

Código: [files/table.insert_vs_table.lua](files/table.insert_vs_table.lua)



# table.HasValue(table,x) vs table[x]

Resumen: Verificar directamente la clave de la tabla con table[valor] es mucho más rápido.

Resultados (1 millón de búsquedas):

    table.HasValue:   0.032319600000051
    table[valor]:     0.00024150000001555


Código: [files/table.hasvalue_vs_table.lua](files/table.hasvalue_vs_table.lua)

Nota:
Esto es una situación O(n) vs O(1). Significa que mientras más grande sea la tabla, más lento será table.HasValue.  
Para más ejemplos visita [https://wiki.facepunch.com/gmod/table.HasValue](https://wiki.facepunch.com/gmod/table.HasValue)  
Para aprender más sobre la "Notación Big O" visita, por ejemplo: [https://web.mit.edu/16.070/www/lecture/big_o.pdf](https://web.mit.edu/16.070/www/lecture/big_o.pdf)



# table.Empty(tab) vs tab = {}

Resumen: Asignar una nueva tabla vacía es generalmente más rápido que vaciar una existente.

La siguiente prueba compara tablas pequeñas (10 elementos) y grandes (10,000) con cada método. También diferencia entre tablas numéricas (claves 1,2,3,...) y tablas con índices string (claves a,b,c,...).  
Resultados:

    --- Benchmark completado
    repeticiones	1	vueltas	1000
    En Servidor
    table.Empty numérico pequeño	1.5931000018554e-06
    table.Empty numérico grande	5.295060000276e-05
    table.Empty string pequeño	4.9099999841928e-07
    table.Empty string grande	4.7529999847029e-07
    tabla = {} numérico pequeño	9.8249999609834e-07
    tabla = {} numérico grande	6.0400000597838e-07
    tabla = {} string pequeño	5.8179999678032e-07
    tabla = {} string grande	4.8149999975067e-07


Los tiempos anteriores varían mucho según el tamaño de la tabla y la cantidad de pruebas. En general, asignar una nueva tabla es más rápido que vaciar una existente, siendo la única excepción las tablas pequeñas con índices string. La mayor diferencia se da al vaciar una tabla numérica grande versus asignar una nueva.  
Ventaja de asignar siempre una nueva tabla: El tiempo requerido es casi constante, por lo que no hay que preocuparse por variaciones.


Código: [files/table_empty_vs_new.lua](files/table_empty_vs_new.lua)



# ply:IsDoing() vs IsDoing(ply)

Resumen: No usar metatables es más rápido.

Esta es una comparación entre usar la MetaTable PLAYER versus simplemente pasar el jugador como argumento.

Resultados (100,000 ejecuciones, 100 veces):

    ply:IsVal()    2.5642945799856e-06
    IsVal(ply)     1.4792759700133e-06


Como se puede observar, no usar la metatable es más rápido. Esto ocurre porque con `ply:Func()` hay que verificar si la función existe (lo que llama al método __index de la metatable PLAYER), lo cual es lento, mientras que con `Func(ply)` ya se tiene la función directamente disponible.

Código: [files/meta_vs_argument.lua](files/meta_vs_argument.lua)



# Concatenación de Strings vs Tabla de Concatenación

Resumen: Es más rápido unir strings mediante tablas que concatenarlos directamente.

Resultados (10000 strings):

    CONSOLA DEL SERVIDOR GMOD
    Concatenación directa:   0.32185959999993
    Concatenación con tabla: 0.0099451999999474
    CONSOLA LUA5.3
    Concatenación directa:   0.086629
    Concatenación con tabla: 0.006428

Código: [files/string_vs_table_concat.lua](files/string_vs_table_concat.lua)



# string.format vs concatenación (..)

Resumen: Ambos tienen casi la misma velocidad de ejecución. Varía entre cuál es más rápido en cada ejecución, están tan cerca.  
Lo que es mejor depende del caso de uso, recomiendo usar `string.format` para casos donde la entrada no es fija, porque con esta función puedes añadir entidades/jugadores sin tener que convertirlos a string explícitamente.

Esta es una comparación de las 2 formas de crear un string único a partir de múltiples partes, ya sea con string.format o con `..` entre los argumentos. Ejemplo:

```lua
format = string.format("%s tiene steamid %s", ply:Nick(), ply:SteamID())
concat = ply:Nick().." tiene steamid "..ply:SteamID()
```

Resultado:

    --- Benchmark completado
    repeticiones	10	vueltas	1000
    En Servidor
    concat str	8.088840000346e-06
    format str	8.3178000002022e-06
    concat int	8.2556100002705e-06
    format int	7.6555600000233e-06

Como puedes ver arriba, la velocidad varía mucho y aquí la función format está empatada con la concatenación de strings.

El código: [files/string_format_vs_concat.lua](files/string_format_vs_concat.lua)



# surface.DrawText vs draw.DrawText

Resumen: Usar `draw.SimpleText` es casi tan rápido como usar las funciones `surface.*`. DrawText es un poco más lento y SimpleTextOutlined es, por mucho, el más lento.

Esta prueba simplemente dibuja una palabra en 3 coordenadas diferentes en tu pantalla con 4 funciones diferentes y mide el tiempo tomado.  
Resultado:

    surface     4.4340000022203e-05
    simple      4.8489999971935e-05
    drawtext    5.563999984588e-05
    outlined    0.00012634444445009

Como se puede ver arriba, el método `surface.*` es el más rápido, seguido de cerca por draw.SimpleText. Posiblemente es más rápido que `draw.DrawText` porque no "maneja saltos de línea correctamente" según la wiki de gmod. SimpleTextOutlined es el más lento porque tiene que dibujar un contorno adicional comparado con las otras 3 funciones.

El código: [files/surface_vs_draw_text.lua](files/surface_vs_draw_text.lua)



# file.Read vs sql.Query

Resumen: Usar file.Read es un poco más rápido, pero apenas hay diferencia entre los 2 métodos.

Si quieres guardar texto y recuperarlo después, ¿qué método es más rápido? ¿Guardarlo en un archivo y leerlo o guardarlo en sqlite y leerlo?  
Resultado:

    --- Benchmark completado
    repeticiones    20  vueltas  100
    En Servidor
    file    0.00013605510000012
    sql     0.00014516330000007

No hay una diferencia real de velocidad, por lo que no se recomienda elegir el tipo de almacenamiento basado en velocidad. Deberías elegir tu tipo de almacenamiento según lo que necesites y con qué frecuencia quieras cambiar los datos.

El código: [files/file_vs_sql_text.lua](files/file_vs_sql_text.lua)



# sql.Query vs QueryRow vs QueryValue

Resumen: No son comparables ya que Query es una función en C y las otras son funciones en LUA que lo utilizan.

Si imprimes el "short source" de estas 3 funciones con `debug.getinfo` obtienes:

    Query       [C]
    QueryRow    lua/includes/util/sql.lua
    QueryValue  lua/includes/util/sql.lua

El código (simplificado) de QueryRow es:

    function sql.QueryRow(query,row)
        row = row or 1
        local r = sql.Query(query)
        if r then return r[row] end
        return r
    end

Esto significa que no importa realmente (en términos de velocidad) si usas Query y seleccionas la primera fila o usas QueryRow directamente.



# Encontrar entidades cerca de un jugador

Resumen: ents.FindInBox y ents.FindInCone son las formas más rápidas de encontrar entidades cerca de un jugador porque son fáciles de calcular y no devuelven demasiadas entidades.

Los métodos probados para obtener entidades alrededor de un jugador son:

 - ents.FindInPVS
 - ents.FindInCone
 - ents.FindInSphere
 - ents.FindInBox

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	10	vueltas	10000
    inPVS	2.2498948000473e-05
    inBox	1.9315759995857e-06
    Sphere	3.8246590005724e-06
    Cone	1.6260100000977e-06

El código: [files/finding_entities.lua](files/finding_entities.lua)



# Encontrar jugadores cerca de ti

Resumen: Es más rápido iterar a través de todos los jugadores y verificar su distancia a ti que usar `ents.FindInSphere` y filtrar por jugadores.

Esto se probó con 40 jugadores (bots) en un servidor. Al iterar a través de todos los jugadores y verificar la distancia entre tú y ellos, eres más rápido que al encontrar e iterar a través de todas las entidades cerca de ti y verificar si son jugadores.

Resultado:
    
    --- Benchmark completado
    En Servidor
    repeticiones	10	vueltas	1000
    FindInSphere	0.00011627801000012
    PlyGetAllChk	1.9475780000045e-05
    --- Benchmark completado
    En Cliente
    repeticiones	10	vueltas	1000
    FindInSphere	8.301507999995e-05
    PlyGetAllChk	2.4392489999832e-05

El código: [files/finding_players.lua](files/finding_players.lua)



# Hashes (MD5, SHA1, SHA256)

Resumen: Como era de esperar, cuanta más unicidad necesites, más tiempo tomará calcularlo. Esto significa que MD5 es el más rápido pero tiene mayor probabilidad de colisiones que el más lento SHA256.

CRC32 y Base64 están incluidos solo como comparación. No son "buenos algoritmos de hash" en la misma categoría que MD5 por ejemplo.  
Cada función probada está definida en C y no en LUA (según short_src de `debug.getinfo`).

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	10	vueltas	10000
    CRC32	4.5670400035306e-07
    MD5	    2.5526380002202e-06
    SHA1	3.5287520001384e-06
    SHA256	7.7691459993503e-06
    Base64	4.1977299973951e-07

El código: [files/hashes.lua](files/hashes.lua)



# Comparación de "se están mirando"

Resumen: Usar la distancia entre vectores normalizados de la dirección de visión de los jugadores es el método más rápido.

La pregunta es: ¿Cómo calcular de manera más eficiente "¿Están estos 2 jugadores viéndose mutuamente en sus pantallas ahora mismo?"  
Esto fue un problema para una mecánica de juego que necesitaba calcularse eficientemente para no causar lag.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones    5   vueltas  100
    AngleBetweenVectorsManual     	1.1895999687113e-06
    AngleBetweenYawOnly           	6.7819999458152e-07
    WikisLookingAtVecDotFunction	1.0634000118444e-06
    DistanceBetweenNormVecManual	1.5873999782343e-06
    DistanceBetweenNormVecAuto	    1.0834000095201e-06
    DistanceBetweenAimVectors     	8.4879999485565e-07

La forma más rápida es usar `ply:GetAimVector()` de ambos jugadores, negar uno de esos vectores y verificar la distancia entre ellos.

## Explicación detallada

El problema por defecto que muchos conocen es: ¿Apuntan estos 2 vectores en la misma dirección?  
Este problema es similar al nuestro: ¿Se están mirando estos 2 jugadores?  
La diferencia principal es: Uno de los vectores está invertido.

Esto significa que solo necesitamos invertir uno de los 2 vectores y luego podemos usar las fórmulas matemáticas estándar para calcular el "problema por defecto" mencionado anteriormente.  
La ralentización al calcular esto viene del uso de `math.acos` (arco coseno) en la fórmula oficial. Necesitas esto para calcular el vector desde la fórmula del producto punto.

Una forma más fácil es simplemente usar la distancia entre los 2 AimVectors. `GetAimVector` devuelve un Vector normalizado que siempre tiene longitud 1. Esto significa que la distancia máxima entre los 2 vectores es `2` (si los vectores miran en direcciones exactamente opuestas) y el mínimo es `0` (si los vectores se miran directamente). Esto lo hace más rápido que todos los otros métodos porque te saltas un cálculo de ángulo y solo comparas la distancia directamente.


El código: [files/looking_at_each_other.lua](files/looking_at_each_other.lua)



# for v in plys hook.Run vs hook.Run plys

Resumen: Es más rápido llamar un hook una vez con todos los jugadores iterando dentro, que iterar a través de los jugadores y llamar un hook para cada uno individualmente.

Esta es una comparación entre:

 - Llamar un hook por cada jugador
 - Llamar un hook con una lista de todos los jugadores

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones    10  vueltas  10000
    single	9.2695600087609e-07
    many  	2.602249987649e-07

El código: [files/hook_many_vs_hook_once.lua](files/hook_many_vs_hook_once.lua)



# fn de DarkRP vs lua normal

Resumen: La librería fn de DarkRP es (la mayoría de veces) más lenta y difícil de entender que código lua simple.

Esta es una comparación entre 2 funciones que hacen lo mismo pero están escritas de forma completamente diferente.  
Por favor revisa el código para ver exactamente qué fragmentos fueron probados.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones    10  vueltas  1000
    darkrp  7.9022700008863e-06 (0.0000079022700008863)
    default	8.2877000014605e-07 (0.00000082877000014605)

No solo el código fn de DarkRP es más difícil de leer, sino que puede ser hasta 10 veces más lento que una simple cadena de funciones que hacen lo mismo.

El código: [files/darkrp_fn_vs_standard_lua.lua](files/darkrp_fn_vs_standard_lua.lua)



# surface.DrawRect vs draw.RoundedBox

Resumen: Es ligeramente más rápido usar surface.DrawRect en lugar de draw.RoundedBox.  
Sin embargo, no debería ser importante para el rendimiento ya que draw.RoundedBox también soporta bordes redondeados y es más simple de leer y escribir.

    surface.DrawRect:   7.4859800010245e-06
    draw.RoundedBox:    8.335099999681e-06

El código: [files/surface_vs_draw_box.lua](files/surface_vs_draw_box.lua)



# surface.SetDrawColor: objeto vs valores separados

Resumen: Es más rápido llamar a SetDrawColor con los valores RGB separados que usar un objeto Color.

Puedes usar la función SetDrawColor de 2 formas:

 - surface.SetDrawColor(color_white)
 - surface.SetDrawColor(255,255,255)

Esta es una comparación entre estos 2 métodos. "Full" significa pasar el objeto Color y "Split" significa pasar los 3 valores numéricos (RGB) a la función.

    --- Benchmark completado
    En Cliente
    repeticiones	10  vueltas	10000
    SetColorFull	6.0781200035763e-07
    SetColorSplit	1.1340800017479e-07

El código: [files/setdrawcolor_comparison.lua](files/setdrawcolor_comparison.lua)



# Creación de tablas: asignaciones vs índice

Resumen: Es más rápido asignar todas las variables de una tabla dentro de los corchetes iniciales.

Actualmente veo 2 formas diferentes de inicializar tablas en el código.  
La primera es simplemente:
```lua
local tab = {}
tab.name = "Test"
tab.money = 9
```
y la segunda es:
```lua
local tab = {
    name = "Test",
    money = 9,
}
```

Esta es una comparación de ambas:

    --- Benchmark completado
    En Servidor
    repeticiones	10	vueltas	10000
    table = { x = y,	4.5506299968565e-07
    table.x = y      	6.2055800030748e-07

Como se puede ver, es más rápido si creas la tabla con los datos ya incluidos.

El código: [files/table_assignments.lua](files/table_assignments.lua)



# print vs Msg

Resumen: MsgN y Msg son más rápidos que `print`. Msg es solo un poco más rápido que MsgN, porque no añade un salto de línea al final.

La prueba simplemente imprime mensajes usando Msg, MsgN y print, y mide el tiempo tomado.

Resultado:

    --- Benchmark completado
    repeticiones	90	vueltas	10
    En Servidor
    Msg	            1.9603333061645e-06
    print	        3.1246666549123e-06
    MsgN	        2.0377778006756e-06
    
    Msg vars	    2.9552222390016e-06
    MsgN vars	    2.9710000045371e-06
    print vars	    9.2549999974128e-06
    
    MsgN vars+tab	3.3575555587757e-06

La prueba está dividida en 2 secciones: La primera donde solo imprimimos un string y la segunda donde imprimimos múltiples argumentos (2 strings y 1 número).  
Como se puede ver, print es siempre el más lento.  
El último benchmark usa MsgN con tabulaciones para imitar lo que hace la función print, y aún así es más rápido que usar print directamente.

El código: [files/print_vs_msgn.lua](files/print_vs_msgn.lua)



# Multiplicación vs División

Resumen: En Garry's Mod no es más rápido calcular x * 0.5 que x / 2.
PERO: Es más rápido multiplicar que dividir en LUA5.3.

Resultado (promedio de 100 rondas con 1,000,000 de cálculos cada una):

    GMOD SERVERSIDE CONSOLE
    Multiplicación:   0.00081622600000003
    División:         0.00081775100000215
    LUA5.3 CONSOLE
    Multiplicación:   0.01865942
    División:         0.02342666

El código: [files/multiplication_vs_division.lua](files/multiplication_vs_division.lua)



# math.Clamp vs math.min y math.max

Resumen: Es mínimamente más rápido usar math.min en combinación con math.max en lugar de math.Clamp, pero la diferencia es insignificante.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	50000
    clamp  	6.9868600001882e-08
    max/min	6.806609987666e-08
    --- Benchmark completado
    En Cliente
    repeticiones	20	vueltas	50000
    clamp  	6.9836399990663e-08
    max/min	6.7090800087044e-08

El código: [files/math_clamp.lua](files/math_clamp.lua)



# Sobreescritura de Vector:Dot(Vector)

Resumen: No es más rápido sobreescribir e implementar el cálculo del producto punto de vectores en LUA.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	30000
    vec:Dot old	1.2875416667678e-07
    vec:Dot new	5.2033999999855e-07

El código: [files/vector_dot_overwrite.lua](files/vector_dot_overwrite.lua)



# Sobreescritura de Vector:Normalize()

Resumen: No es más rápido sobreescribir e implementar el cálculo de normalización de vectores en LUA.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	30000
    vec:Normalize old	1.1955383335495e-07
    vec:Normalize new	5.1291399999911e-07

El código: [files/vector_normalize_overwrite.lua](files/vector_normalize_overwrite.lua)



# Sobreescritura de Vector:Length()

Resumen: No es más rápido sobreescribir e implementar el cálculo de longitud de vectores en LUA.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	30000
    vec:Length old	1.1400050000068e-07
    vec:Length new	3.8188000000545e-07

El código: [files/vector_length_overwrite.lua](files/vector_length_overwrite.lua)



# Sobreescritura de math.Round

Resumen: Es ligeramente más rápido sobreescribir e implementar el cálculo de math.Round en LUA.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	30000
    math.Round old	7.1794499987448e-08
    math.Round new	6.0206499994801e-08

El código: [files/math_round_overwrite.lua](files/math_round_overwrite.lua)



# Velocidad de variables locales vs globales

Resumen: Es un poquito más rápido hacer todas las variables locales en Garry's Mod (14% en este ejemplo).

Resultado (promedio de 100 rondas con 10000 cálculos cada una):

    GMOD SERVERSIDE CONSOLE
    math.sin global:     0.0000059390000023996 (5.9390000023996e-06)
    math.sin local:      0.0000056840000024749 (5.6840000024749e-06)
    LUA5.3 CONSOLE
    math.sin global:     0.00057612
    math.sin local:      0.00049858

El código: [files/local_vs_global.lua](files/local_vs_global.lua)



# Almacenamiento en caché de GetConVar

Resumen: No es más rápido almacenar en caché los ConVars en LUA.

La página del wiki indica que la función GetConVar "...almacena en caché el resultado en Lua para búsquedas más rápidas", por lo que volver a almacenar el resultado no lo hará más rápido.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20	vueltas	1000
    Sin caché	2.0708000000838e-07
    Con caché  	2.1247999993221e-07

El código: [files/getconvar_cache.lua](files/getconvar_cache.lua)



# x^2 vs x*x

Esta prueba compara la función math.pow incorporada con el operador de potencia estándar.

Resumen: Usar el operador "^" es ligeramente más rápido en GMod. La velocidad es igual en Lua5.4.

Resultado (promedio de 100 rondas con 10000 cálculos cada una):

    GMOD SERVERSIDE CONSOLE
    x^2 math.pow:  4.0210000111074e-06
    x*x:           3.7120000251889e-06
    x^2 ^:         3.5500000080901e-06
    LUA5.4 CONSOLE
    x^2 math.pow:  0.00015625
    x^2 ^:         0.00015625
    x*x:           0.00015625

El código: [files/x_squared_vs_x_times.lua](files/x_squared_vs_x_times.lua)



# while vs for

Resumen: Un bucle for es significativamente más rápido que while.

Resultado (100x 10000 iteraciones):

    GMOD SERVERSIDE CONSOLE
    while:    0.00000957507999874 (9.57507999874e-06)
    for:      0.0000024306500009516 (2.4306500009516e-06)
    LUA5.3 CONSOLE
    while:    0.00015975869999999
    for:      0.000092178400000006 (9.2178400000006e-05)

El código: [files/while_vs_for.lua](files/while_vs_for.lua)



# Color() local vs redefinición

Resumen: Definir el color una vez es más rápido que usar Color() repetidamente en un bucle.

Resultado (1000 iteraciones medido con SysTime()):

    Local:     0.0000070583999997496 (7.0583999997496e-06)
    No-Local: 0.0000091475999998067 (9.1475999998067e-06)

El código: [files/local_color_vs_redefining.lua](files/local_color_vs_redefining.lua)



# Impacto de variables globales

Resumen: Crear muchas variables globales no ralentiza el juego ni tiene un impacto notable en el rendimiento.

Escuché de múltiples fuentes que "crear muchas variables globales es malo".  
Probé esto creando 10,000 variables globales y observando el juego después. Un ejemplo con el tiempo de ejecución de 10,000 llamadas a funciones matemáticas:

    Tiempo    Conteo _G    Tiempo tomado
    Antes	3133        0.00044090000005781
    Después	13144       0.00043160000018361

Las estadísticas de FProfiler y la sensación general del juego no cambiaron.  
Para más detalles sobre la prueba (o si quieres verificarlo tú mismo) revisa el script que usé:  
[files/global_variables.lua](files/global_variables.lua)



# table.Count vs operador #

Resumen: Es mejor usar siempre el operador `#` en lugar de table.Count para tablas secuenciales.

Esta comparación solo funciona en tablas secuenciales (o numéricas). Estas son tablas que tienen una clave numérica ascendente, en otros lenguajes se les llamaría "lista" o array.

Table.Count siempre debería usarse para contar tablas indexadas por strings (también llamadas diccionarios o mapas), pero en casos como este ambos pueden usarse.

Resultado:

    # Tamaño de tabla: 1000
    Operador #: 6.2949998141448e-08
    table.Count: 1.5430600009495e-06
    # Tamaño de tabla: 10
    Operador #: 4.4980000029682e-08
    table.Count: 9.0410000939301e-08

Como se puede ver arriba, table.Count se vuelve más lento cuanto más grande es la tabla. Esto es porque recorre todos los elementos de la tabla y los cuenta, según una edición antigua de la wiki de gmod.

El código: [files/table_count_vs_hashtag.lua](files/table_count_vs_hashtag.lua)



# Sobreescritura de table.Random

Resumen: Es más rápido generar un índice aleatorio de una tabla secuencial y obtener un valor aleatorio de esta manera en lugar de usar table.Random().

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones    20  vueltas  10000
    small old	4.6385350004698e-07
    small new	1.2625500000013e-07
    big old	    3.1202909999735e-06
    big new	    1.2906200004807e-07

El código: [files/table_random_overwrite.lua](files/table_random_overwrite.lua)



# Sobreescritura de table.Count

Resumen: No es más rápido sobreescribir la función table.Count y contar la tabla manualmente en LUA.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones    20  vueltas  10000
    count small old	2.7576250000664e-07
    count small new	2.6563899996773e-07
    count big old	1.8296299999935e-06
    count big new	1.8260360000758e-06

El código: [files/table_count_overwrite.lua](files/table_count_overwrite.lua)



# Sobreescritura de file.Write

Resumen: No es más rápido sobreescribir la función file.Write para usar funciones de bajo nivel de Lua para escribir archivos.

Resultado:

    --- Benchmark completado
    En Servidor
    repeticiones	20  vueltas	100
    write old	0.00076962965000047
    write new	0.00079238309999975

El código: [files/file_write_overwrite.lua](files/file_write_overwrite.lua)



# ply.x vs tab[ply][x]

Resumen: Es más rápido obtener variables de una tabla que del objeto ply. Establecer una variable tiene casi la misma velocidad, solo obtenerla es más lento en ply.

Muchos usamos código como `ply.lastSpawned = CurTime()`. Esto es fácil de usar y entender, pero en términos de rendimiento es más lento que usar una tabla. Un ejemplo con tabla sería `tab[ply]["lastSpawned"] = CurTime()`.  
Esto se debe a que leer esta variable de un jugador usa un método __index más lento. Si has usado FProfiler, probablemente hayas visto esta función "metamethod __index player" cerca de la parte superior de "most runtime in ms".

Probé esto estableciendo y obteniendo números aleatorios con ambos métodos 10,000 veces, ejecutando esto 1000 veces y calculando el tiempo promedio. Resultado:

    ply:	0.0033284901999953
    tab:	0.0016247142000013

Como se puede ver, es bastante más rápido (2x) con una tabla.  
Usar una tabla también tiene desventajas, como tener que limpiarla regularmente para evitar fugas de memoria y tener que verificar 2 índices de tabla cada vez que quieras verificarlo.  
Código fuente: [files/ply_vs_table_index.lua](files/ply_vs_table_index.lua)



# MySQL vs SQLite

Para la documentación completa visita [https://shira.at/gmod/mysql_sqlite.html](https://shira.at/gmod/mysql_sqlite.html).  
Configuración: MariaDB en Debian 10 instalado en el mismo servidor que el servidor de Garry's Mod. Probado con funciones MySQL de DarkRP y el módulo mysqloo.  

El tiempo se mide desde la primera solicitud de datos hasta que podemos usar los datos.  

Resumen: MySQL es más lento en comparación con SQLite. Necesita una conexión externa y debe esperar retrasos de solicitud/respuesta ya sea a través de red o sockets internos de Linux. Es excelente para sincronización de múltiples servidores, pero no es un "must-have" para servidores individuales.

## Comparación de retraso en consultas

Para obtener el retraso de una consulta sin operación de lectura/escritura podemos consultar `SELECT 1+1`.

Resultado:

    --Primera prueba
    SQLite   TiempoRespuesta:   0.000045883003622293s
    DRPMySQL TiempoRespuesta:   0.020215623022523s
    --Segunda prueba
    SQLite   TiempoRespuesta:   0.000027909001801163s
    DRPMySQL TiempoRespuesta:   0.049109992978629s

## Comparación de Lectura y Escritura

Consulta: `SELECT rpname FROM darkrp_player WHERE uid = 76561198071737444`

Resultado:

    SQLite   TiempoLectura:   0.00016302999938489s
    DRPMySQL TiempoLectura:   0.048014674001024s

Consulta: `INSERT INTO darkrp_player VALUES(123123,'Test User',1,2)`

Resultado:

    SQLite   TiempoEscritura:   0.00020395400133566s
    DRPMySQL TiempoEscritura:   0.06270497100013s

## 1000 inserciones (Inserciones masivas)

El método SQLite de DarkRP usa sql.Begin y sql.Commit para importaciones masivas. Esto lo hace mucho más rápido que hacer 1000 inserciones una tras otra.  

El método MySQL de DarkRP simplemente pone las consultas en una tabla y las envía una tras otra al servidor mysql. Esto es mucho más lento, ya que el tiempo necesario es simplemente el tiempo que tarda una inserción multiplicado por 1000.

Resultado:

    SQlite   1000 Inserciones:     0.004849937000472s
    DRPMySQL 1000 Inserciones:   ~30.000000000000000s (calculado)


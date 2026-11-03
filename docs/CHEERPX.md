# Vascolet 3: corrección de arranque y estado de rendimiento

CheerpX 1.2.8 ejecuta el ELF x86 original desde `/game/bin/vascolet3`, respetando
su ruta de recursos. Canvas dibuja los sprites y Web Audio reproduce sonidos.
Los originales conservan sus hashes, registrados en `native/cheerpx/sources.json`.

## Cierre al recibir entrada

El adaptador copiaba 24 bytes por evento SDL. En i386, SDL_Event mide 20 bytes.
Los bytes adicionales sobrescribían una variable del bucle de eventos y el
juego se cerraba al mover el ratón o pulsar una tecla. La copia ahora respeta
los 20 bytes. Se verificó el recorrido desde «¡Atención!» hasta una carrera
en Uruguay, con aceleración, dirección y audio.

## Rendimiento medido, noviembre de 2026

Se ajustó el límite interno de 15 a 60 mediante el método original del motor.
Los menús rondan los 59 FPS, pero **la carrera no alcanza 60 FPS**: la última
medición, con aceleración y dirección durante 12,17 s, dio **9,20 FPS**, mediana
98,97 ms y p95 179,90 ms. No se presenta el FPS del menú como rendimiento jugable.

La rasterización se transporta como sprites y líneas. También se aceleran
conversiones de color, casos exactos de transformaciones/colores y la
interpolación del trazado, con validación previa frente a las funciones
originales. Los hashes y direcciones están en
`native/cheerpx/runtime-metadata.json`. El perfil todavía identifica un coste
alto de ejecución emulada durante la carrera; la optimización sigue pendiente.

Prueba local: Chromium y AMD Radeon RX 6500 XT. Audio sin errores. Se verificó
exportar archivos, importar una copia con un archivo de prueba, reiniciar y
volver a exportarla. No se certificó una partida completa ni un teléfono.

Para recompilar: configurar `ZIG` con Zig 0.13.0 y ejecutar
`python native/cheerpx/rebuild.py`. `python build.py` prepara la página.
CheerpX se carga desde su CDN oficial y no se redistribuye en este repositorio.

## Experimento separado: recompilación estática

Se investigó traducir las instrucciones del ELF original a C y compilar ese C
a WebAssembly antes de jugar, usando el traductor MIT de
[Djsmk123/pop-two-thrones-browser](https://github.com/Djsmk123/pop-two-thrones-browser)
con un adaptador ELF local. No es una reescritura de mapas o reglas.

La prueba ejecutó `Color::getYUV` y `TrackData::getHillAt` originales en Chromium
sin CheerpX ni Wine: 131.072 verificaciones frente a cálculos de referencia.
Las 65.536 llamadas de cada función tardaron aproximadamente 7–8 ms en PC.
**Esto no es una medición de FPS ni un port jugable.** El traductor usa doubles
para x87 y todavía no garantiza equivalencia general de precisión extendida.

La auditoría genera código para 9.177 funciones del ejecutable. No prueba que
todas funcionen: quedan 466 símbolos importados, inicialización de objetos,
excepciones C++, despacho de vtables, memoria y adaptación al bucle del navegador.
Las bibliotecas compartidas no están cubiertas por ese conteo. El motor AOT
no está seleccionado en la página ni ha reemplazado el motor actual.

El puente publicado sí incorpora menor presión de memoria: audio bajo demanda
y reutilizado por URL, canales liberados en errores de audio, y un buffer
gráfico reutilizable. La lentitud de la carrera sigue pendiente de resolución.

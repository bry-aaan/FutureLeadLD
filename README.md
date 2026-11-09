# FutureLead LD  — aproximación original a un lead "future" (VST3 / JUCE)

> **Honestidad por delante**
> * Esto es una **aproximación por oído/diseño**, NO una copia ni una réplica verificada de "LD The Future Lead" de reFX Nexus 2.
>   No tuve acceso a una grabación del original ni al preset; **no he hecho comparación auditiva ni espectral contra Nexus**.
>   Los valores por defecto son suposiciones razonables para un lead supersaw brillante (ver `Source/FutureLeadCore.h`, tabla `kParams`).
> * El **motor DSP** (`FutureLeadCore.h`) está compilado y probado (ver abajo).
> * El **wrapper JUCE** (`PluginProcessor/Editor`, `CMakeLists.txt`) está escrito pero **NO se ha compilado** (mi entorno no tenía JUCE, red ni compilador de Windows). Puede requerir ajustes menores al primer build.
> * No hay `.vst3` binario en este ZIP; hay que compilarlo (instrucciones abajo, la vía más fácil es GitHub Actions).

## Estructura
```
Source/FutureLeadCore.h     motor DSP (sin JUCE): osciladores PolyBLEP, unison, capa B, filtro SVF 12/24 dB,
                            envolventes, drive, shelf de brillo, low cut, chorus, anchura M/S, limitador suave
Source/PluginProcessor.*    wrapper JUCE: parametros (APVTS), MIDI, presets, estado
Source/PluginEditor.*       GUI oscura, teclado MIDI visual, controles generados desde la tabla de parametros
Tests/render_test.cpp       pruebas offline + render a WAV (compila con g++/MSVC, sin JUCE)
Tests/audio/*.wav           ejemplos renderizados con el motor
Tools/analyze.py            analisis de WAV (espectro, centroide, anchura, ataque) para comparar con una referencia
.github/workflows/          compilacion automatica del VST3 en Windows
CMakeLists.txt              build JUCE (descarga JUCE 8.0.4 via FetchContent)
```

## Opción A — Compilar sin instalar nada (GitHub Actions)
1. Crea un repositorio en GitHub y sube el contenido de esta carpeta.
2. Pestaña **Actions → Build VST3 (Windows) → Run workflow**.
3. Al terminar, descarga el artefacto `FutureLeadLD-VST3-Windows` (contiene `FutureLead LD.vst3`).

## Opción B — Compilar en tu PC (Windows 10/11)
Requisitos: Visual Studio 2022 (carga de trabajo "Desarrollo para escritorio con C++"), CMake ≥ 3.22, Git.
```
cmake -S . -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release --target FutureLeadLD_VST3
```
Resultado: `build\FutureLeadLD_artefacts\Release\VST3\FutureLead LD.vst3`

## Instalar en FL Studio
1. Copia la carpeta `FutureLead LD.vst3` a `C:\Program Files\Common Files\VST3\`.
2. FL Studio → **Options → Manage plugins → Find installed plugins** (con "Verify plugins" activado).
3. Añade **FutureLead LD** desde *Add → Instruments* (o Plugin database → Installed → Generators).
4. Abre automáticamente con el preset **LD THE FUTURE LEAD**. Toca con el teclado MIDI, el piano roll o el teclado visual.

## Presets
* **RESET**: vuelve a LD THE FUTURE LEAD. **SAVE / LOAD**: archivos `.flpreset` (XML) en `Documentos\FutureLeadLD Presets`.
* El estado también se guarda dentro del proyecto de FL Studio.

## Pruebas del motor DSP (ejecutadas realmente)
`g++ -O2 -std=c++17 -ISource Tests/render_test.cpp -o render_test && ./render_test Tests/audio`
Comprueba: sin NaN y pico ≤ 1.0 a 44.1/48/96 kHz, acorde, melodía y 12 notas sobre 8 voces (robo de voz), correlación estéreo,
modo mono exacto, que **cada uno de los 32 parámetros** altere el audio en mín. y máx. sin inestabilidad, respuesta a velocidad y extinción del release.

## Limitaciones conocidas
* La saturación (tanh) y los osciladores PolyBLEP no usan sobremuestreo: a 44.1 kHz hay algo de aliasing en notas agudas con drive alto.
  Se nota en el centroide espectral (≈5.2 kHz a 44.1 vs ≈6.2 kHz a 96 kHz para el mismo acorde). Mejora recomendada: oversampling 2x–4x en el drive.
* Sin suavizado de parámetros salvo el volumen; cambios bruscos de cutoff/drive pueden dar pequeños clics.
* Sin automatización de expresiones MPE, sin arpegiador, glide ni modo mono/legato.
* No he podido cargar el plugin en FL Studio ni verificar el wrapper JUCE: esas comprobaciones quedan pendientes en tu máquina.

## Para acercarte de verdad al original
Graba (o exporta) la nota C4 y un acorde del preset original de Nexus 2 a WAV 16-bit, y compara con
`python3 Tools/analyze.py original.wav Tests/audio/chord_44100.wav` (ataque, centroide, bandas, anchura).
Ajusta `Cutoff, Env Amount, Detune, Unison, Drive, Brightness, Chorus, Width` hasta igualar. Si me pasas la grabación, hago ese ajuste por ti.

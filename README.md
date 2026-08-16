# Forge Kernel v3.2 RC-2

Un kernel frontend gobernado y con constitución ejecutable, escrito en
HTML/JS puro, con una Analysis Suite integrada y un QualityGate unificado.

## Autor

- **Yoandis Rodríguez**
- Email: curvadigital0@gmail.com

## Licencia

Licencia MIT

Copyright (c) 2026 Yoandis Rodríguez

Se concede permiso, de forma gratuita, a cualquier persona que obtenga
una copia de este software y de los archivos de documentación asociados
(el "Software"), a utilizar el Software sin restricción, incluyendo sin
limitación los derechos de uso, copia, modificación, fusión, publicación,
distribución, sublicenciamiento y/o venta de copias del Software, y a
permitir a las personas a las que se les proporcione el Software a hacer
lo mismo, sujeto a las siguientes condiciones:

El aviso de copyright anterior y este aviso de permiso se incluirán en
todas las copias o partes sustanciales del Software.

EL SOFTWARE SE PROPORCIONA "TAL CUAL", SIN GARANTÍA DE NINGÚN TIPO, EXPRESA
O IMPLÍCITA, INCLUYENDO PERO NO LIMITADO A GARANTÍAS DE COMERCIABILIDAD,
IDONEIDAD PARA UN PROPÓSITO PARTICULAR Y NO INFRACCIÓN. EN NINGÚN CASO LOS
AUTORES O TITULARES DEL COPYRIGHT SERÁN RESPONSABLES POR NINGUNA RECLAMACIÓN,
DAÑO U OTRA RESPONSABILIDAD, YA SEA EN UNA ACCIÓN DE CONTRATO, AGRAVIO O DE
OTRA FORMA, DERIVADA DE, FUERA DE O EN CONEXIÓN CON EL SOFTWARE O EL USO U
OTROS TRATOS EN EL SOFTWARE.

## Estado

- Forge Kernel v3.2 RC-2 — Gobernanza Completa
- 15/15 invariantes constitucionales
- 17 módulos enlazados (bound)
- 14/14 diagnósticos en verde (diag11, diag13, diag15, diag16, diag17, diag20,
  diag20a, diag21a, diag21b, diag22, diag23, diag24, diag25, diag26b)
- QG-INT-01 cerrado: `QualityGateSwitch` y `kernelQG.runAll()` ahora comparten
  la misma semántica observable sobre `kernelQG.results`.
- Analysis Suite integrada (IIFE local, no es un módulo constitucional).

## Resumen de arquitectura

El kernel expone una superficie `window.IX` congelada (37 claves) que:

- declara y valida la constitución ejecutable (`IX.CONTRACT`, 5 contratos
  congelados: ModuleSpec, Signal, ModuleState, PerformanceModel,
  TelemetryStats);
- impone el punto único de entrada para nuevos módulos vía
  `IX.registerModule(spec)` (validado contra la constitución, congelado
  in situ);
- gobierna capacidades mediante un catálogo congelado `IX.CAPABILITIES`
  y los accesos `IX.hasCapability(code, capability)` /
  `IX.getCapability(code, capability)`;
- expone un mapa de versiones semánticas (`IX.VERSION`) y
  `IX.assertCompatibility(spec)` para rechazar contratos obsoletos.

La Analysis Suite se implementa como un controlador IIFE local
(`analysisUI`) que consume el grafo real del kernel (`forgeUI.graph`) y
los resultados del QualityGate (`kernelQG.results`). No está registrada
como módulo y no declara capacidades.

## Módulos (17)

Cada módulo está declarado en `MODULE_SPECS` y se enlaza al arrancar
mediante `bindModule(code)`. Cada uno se identifica con un código
(por ejemplo `IN-01`) y está respaldado por un spec congelado. La
columna *Función* está tomada textualmente de la descripción
`data-ix-layer="function"` del módulo en el HTML.

| Código | Función |
|---|---|
| `IN-01` | Lee el archivo de manifiesto, valida el esquema y registra los módulos declarados en el Registry. |
| `DG-01` | Ordena topológicamente los módulos del Registry y muestra las dependencias entre ellos. |
| `QE-01` | Ordena los módulos por prioridad y los entrega al Executor en orden estable. |
| `EX-01` | Ejecuta los módulos encolados, registra el resultado y emite los eventos del Bus. |
| `TX-01` | Cada soldadura es una transacción: snapshot del runtime, ejecución, commit o rollback completo. |
| `SK-01` | Sin lógica de negocio · consume una señal del DOM y alcanza READY. |
| `ST-01` | Auto-monitoreo de módulos failed/stopped · Watchdog + política de reinicio. |
| `RG-01` | Mantiene el registro de módulos · Hash Merkle · Sincronización con Navigator. |
| `CP-01` | Búsqueda de comandos y paneles · Ejecuta navegación global. Estado = visibilidad del overlay (no contenido del input). |
| `OB-01` | Activity Log alimentado por EventBus · Vistas Timeline, Test Runner, Inspector. Solo lectura. |
| `AT-01` | Registro inmutable de operaciones · Cadena Merkle · Exportable y verificable. |
| `QG-01` | Quality Gate continuo · 7 prerrequisitos · interruptor ON/OFF. |
| `HC-01` | Sondas periódicas de salud por módulo · Métricas CPU/Mem/Latencia · Historial. |
| `PF-01` | Captura tiempo de soldadura, invocaciones, memoria estimada por módulo. Snapshot + Reset. |
| `LC-01` | Orquesta transiciones DECLARED→BOOTING→EXECUTING→RUNNING→STOPPED\|FAILED\|RESTARTING. Diagrama en vivo. |
| `PA-01` | Consume `IX.getObserveStats()` y aplica `IX.PerformanceModel.compute` (puro, determinista). Score + métricas + cuellos de botella. |
| `PA-02` | Registra transiciones (from→to) por código cuando `IX.getObserveStats()` reporta deltas. Buffer privado FIFO con cap 50. |

## Diagnósticos (14)

Cada diagnóstico es un script Node independiente en `/workspace/diag*.js`
que dirige Playwright contra el kernel y verifica invariantes. Los 14
son reproducibles mediante `bash run-diags.sh` (ver *Cómo ejecutar los
diagnósticos*).

| Diagnóstico | Verifica |
|---|---|
| `diag11` | Open/closed: 5 módulos nuevos añadidos solo con HTML + spec, IX sin cambios, 15/15 invariantes. |
| `diag13` | `IX.observe()` expuesto, CP-01 + LC-01 migrados a observación declarativa, I13 se mantiene (ningún spec posee observación). |
| `diag15` | Sincronización del título + perfil lineal de solo-observación (los polls escalan linealmente con muestras de 1s). |
| `diag16` | 4 combinaciones de flags (notify × profile) + alias `silent:true`, stats congelados, sin fugas. |
| `diag17` | RQ-09 + RQ-10: ciclo de vida de observers e intervals equilibrado a cero bajo estrés de 1000 ciclos. |
| `diag20` | PA-01 + PA-02 nativos, sin regresión I13 / I14 / I15. |
| `diag20a` | Test unitario de `IX.PerformanceModel.compute`: puro, determinista, congelado, rangos razonables. |
| `diag21a` | Constitución honesta: 5 contratos congelados, 6 validadores, 17 módulos reales + 17 estados reales, todos pasan. |
| `diag21b` | Constitución resiste ataque: 10/10 intentos (reemplazar, mutar, borrar, redefinir) rechazados. |
| `diag22` | `IX.registerModule()` impuesto en el punto único de entrada: válido se congela, inválido se rechaza, colisión se bloquea con `allowReplace`. |
| `diag23` | Gobernanza de capacidades: catálogo congelado, formas binaria y parametrizada, lookup por `IX.hasCapability` / `IX.getCapability`. |
| `diag24` | Versionado semántico: `IX.VERSION` es un objeto congelado, `IX.assertCompatibility` rechaza contratos obsoletos. |
| `diag25` | Analysis Suite E2E: Dependencias, Seguridad, Cobertura y Refresh operan sobre datos reales del kernel (grafo + meta.raw + kernelQG.results). |
| `diag26b` | QG-INT-01 POST-fix: el switch y `runAll()` producen la misma semántica observable sobre `kernelQG.results`. |

## Reglas para contribuir

Estas reglas son las que han mantenido estable el proyecto hasta ahora.
Romperlas debe ser una decisión deliberada, no un accidente.

1. **La Constitución es ejecutable.** `IX.CONTRACT` (5 contratos
   congelados) es la fuente de verdad de cómo deben verse un módulo,
   una señal, un estado y un modelo de rendimiento válidos. Cualquier
   cambio en la forma de un contrato debe incrementar la versión
   correspondiente en `IX.VERSION`; los specs existentes que declaren
   la versión antigua serán rechazados por `IX.assertCompatibility`.

2. **Los módulos nuevos pasan por `IX.registerModule(spec)`.** La
   manipulación directa de `MODULE_SPECS` es un atajo. Los specs de
   bootstrap (declarados en el IIFE) están exentos porque son
   authored por el kernel y se enlazan al arrancar con
   `bindModule(code)`; las adiciones en tiempo de ejecución deben
   pasar por el punto de entrada y son validadas, congeladas y
   enrutadas.

3. **Las capacidades son gobernanza, no permisos.** Añadir una
   capacidad al catálogo `IX.CAPABILITIES` es una decisión a nivel
   de kernel. Los subsistemas (`IX.tx`, futuros `IX.storage`,
   `IX.export`) consultan `IX.hasCapability(code, capability)` y
   `IX.getCapability(code, capability)`. Nunca confían en el spec
   directamente.

4. **Los servicios de fondo son territorio del kernel.** `setInterval`,
   `requestAnimationFrame`, `new MutationObserver`, `__Poller` y
   llamadas a `refreshModule` dentro de `read()` están prohibidas
   por la invariante I13. La observación es asunto del kernel; un
   spec es puro.

5. **Diagnósticos antes de funcionalidades.** Cualquier cambio en una
   superficie pública (kernel, constitución, capacidades, versionado)
   debe ir acompañado de un diagnóstico que capture tanto el baseline
   (antes) como el estado post-cambio. `diag26a.baseline.out` se
   conserva como evidencia histórica inmutable; `diag26b` documenta
   la semántica post-fix.

## Cómo ejecutar los diagnósticos

```bash
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag11.js
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag25.js
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag26b.js
```

Cada diagnóstico debe imprimir una línea `PASS` para considerar el
build en verde. El estado actual es **14/14 PASS**.

Hay un runner portable disponible:

```bash
bash /workspace/run-diags.sh                              # target por defecto
bash /workspace/run-diags.sh /ruta/a/forge-in01.html      # target explícito
TARGET=/ruta/a/forge-in01.html bash /workspace/run-diags.sh  # target por env
```

El runner auto-detecta el binario de Chromium vía `CHROMIUM_PATH` o
`playwright.chromium.executablePath()`.

## Archivos en este directorio

- `forge-in01.html` — el monolito del kernel (archivo único, ~13,150 líneas).
- `README.md` — este archivo.

## Reporte de errores

Si encuentras un error, por favor repórtalo por escrito a
**curvadigital0@gmail.com** incluyendo:

1. **Qué esperabas que pasara**
2. **Qué pasó realmente**
3. **Pasos para reproducirlo**

Esto ayuda a corregir el código de forma eficiente.

## Screenshots



![Capability Layer](screenshot-capability-layer.jpg)





![Forge Queue](screenshot-forge-queue.jpg)





![Transaction Engine](screenshot-transaction-engine.jpg)





![Performance Analyzer](screenshot-performance-analyzer.jpg) ## Contacto

Yoandis Rodríguez — curvadigital0@gmail.com

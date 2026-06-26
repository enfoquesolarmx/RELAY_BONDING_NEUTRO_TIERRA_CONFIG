# RELAY_BONDING_NEUTRO_TIERRA_CONFIG
Plantea la arquitectura de un relay de vínculo neutro-tierra configurable según el tipo de instalación física del usuario, con sus reglas de seguridad. El objetivo es darle al 99% la opción correcta para SU caso, con la guía para no equivocarse — porque el bonding mal hecho es peligroso en las dos direcciones.

# Diseño para Evaluación — Relay de Bonding Neutro-Tierra (N-G) Configurable

> **DOCUMENTO DE DISEÑO PARA EVALUAR. No es código ni guía de instalación final.**
> Plantea la arquitectura de un relay de vínculo neutro-tierra configurable según
> el tipo de instalación física del usuario, con sus reglas de seguridad. El
> objetivo es darle al 99% la opción correcta para SU caso, con la guía para no
> equivocarse — porque el bonding mal hecho es peligroso en las dos direcciones.
>
> **ADVERTENCIA CENTRAL:** este documento describe la conmutación del VÍNCULO entre
> neutro y tierra. NUNCA la conmutación de la tierra de protección misma, que debe
> permanecer continua y firme en todo momento. Conmutar la tierra de protección
> sería peligroso; conmutar su vínculo con el neutro, según el contexto, es lo
> correcto y es lo que hacen los inversores/cargadores profesionales.
>
> **Este diseño toca la red eléctrica (120/240 VAC) y la seguridad de personas.
> Debe ser revisado por alguien con competencia en instalaciones eléctricas y
> ajustado a la norma local (NOM en México, NEC en EE.UU., etc.) antes de
> construirse. Un error de bonding puede dejar un chasis energizado sin disparar
> protección, o crear lazos de corriente peligrosos.**

---

## 1. Qué es el bonding neutro-tierra y por qué confunde a todos

En un sistema eléctrico hay tres conductores que importan aquí:
- **Línea (L1):** el conductor "vivo", el que lleva el voltaje.
- **Neutro (N):** el conductor de retorno de corriente, normalmente cerca de 0V.
- **Tierra de protección (G/PE):** el conductor de seguridad, conectado a una
  varilla en el suelo, que NO lleva corriente en operación normal y existe para
  desviar corrientes de falla y mantener el chasis seguro.

El **bonding N-G** es el punto donde el neutro se conecta físicamente a la tierra.
Este vínculo es lo que le da al sistema una referencia: hace que el neutro esté
de verdad cerca del potencial de tierra, y permite que las protecciones de falla
a tierra (diferenciales / GFCI) tengan una referencia para detectar fugas.

**La regla de oro, la que casi nadie entiende:** el bonding N-G debe existir en
**UN SOLO punto** de todo el sistema. Ni cero, ni dos. Uno.
- **Cero bonding:** el sistema "flota", sin referencia a tierra. Una falla puede
  energizar el chasis sin que nada lo detecte. Peligroso.
- **Dos o más bonding:** se crean lazos. La corriente de neutro encuentra caminos
  por la tierra, energiza conductores de tierra que deberían estar a 0V, y puede
  confundir o impedir el disparo de las protecciones. Peligroso.
- **Un bonding:** correcto. Referencia clara, protecciones funcionales.

El problema es que DÓNDE va ese único punto **depende del tipo de instalación**.
Y ahí es donde el usuario del 99% necesita la guía.

---

## 2. Los tres escenarios del 99% (y dónde va el bonding en cada uno)

### Escenario A — OFF-GRID PURO (el inversor es la única fuente)
El inversor Lázaro es el origen de toda la energía AC. No hay red eléctrica.
- **El bonding N-G debe estar UNIDO dentro del sistema del inversor.**
- Razón: si el inversor es la fuente, él tiene que crear la referencia a tierra.
  Sin este bonding, el sistema flota y las protecciones no tienen referencia.
- Caso típico: rancho aislado, vivienda sin conexión a CFE, sistema solar autónomo.

### Escenario B — CONECTADO A RED (la red de CFE/utility provee la energía)
La casa está conectada a la red, y el bonding N-G **ya existe en el panel
principal de la acometida** (lo hizo la instalación de la casa, en el origen del
servicio).
- **El bonding N-G del inversor debe estar SEPARADO (abierto).**
- Razón: el bonding ya existe en el panel de la casa. Si el inversor crea un
  segundo bonding, se forma el lazo peligroso (doble bonding). El inversor NO debe
  duplicar lo que la red ya provee.
- Caso típico: casa urbana con respaldo, sistema con conexión a la red.

### Escenario C — TRANSFERENCIA (a veces red, a veces isla)
El sistema opera conectado a la red cuando hay, y pasa a isla (off-grid) cuando la
red se cae. El bonding tiene que **CAMBIAR según el modo**:
- **Modo red presente:** bonding del inversor SEPARADO (la red provee el suyo).
- **Modo isla (red caída):** bonding del inversor UNIDO (ahora el inversor es la
  fuente y debe crear la referencia).
- Este cambio dinámico es exactamente la función del relay de bonding conmutable,
  y es lo que hacen los inversores/cargadores profesionales (Victron, etc.).
- Caso típico: vivienda con respaldo automático ante apagones.

---

## 3. El relay de bonding — qué conmuta y qué NO

**LO QUE EL RELAY CONMUTA:**
- El vínculo entre Neutro (N) y Tierra (G), cerrándolo (UNIDO) o abriéndolo
  (SEPARADO) según el modo del sistema.

**LO QUE EL RELAY NUNCA TOCA:**
- La tierra de protección (G/PE) en sí. La tierra va siempre conectada, continua,
  firme, a la varilla de tierra y al chasis. El relay no la interrumpe jamás.
- Lo que se gestiona es si el NEUTRO se vincula o no a esa tierra continua.

```
   Tierra de proteccion (G) ===== SIEMPRE CONTINUA, a varilla y chasis =====
                                          |
                                    [ RELAY N-G ]   <- esto conmuta
                                          |
   Neutro (N) -------------------------- nodo --------------------------
```

- Relay CERRADO  -> N unido a G (bonding presente) -> para OFF-GRID / ISLA
- Relay ABIERTO  -> N separado de G (sin bonding)   -> para CONECTADO A RED

---

## 4. Reglas de seguridad del control del relay (a prueba de fallos)

El bonding mal conmutado es peligroso en ambos sentidos, así que el control debe
ser conservador y fallar hacia el estado seguro:

1. **Certeza del modo antes de conmutar.** El sistema debe SABER con seguridad si
   la red está presente o no antes de unir o separar el bonding. Conmutar con
   información ambigua es inaceptable. La detección de red debe ser confiable y
   confirmada (no por un solo ciclo, sino sostenida — como el filtro
   anti-transitorio que ya usamos en la batería).

2. **Secuencia correcta en la transferencia.** Al pasar de red a isla, el orden
   importa: primero desconectar de la red (relays de transferencia L/N abiertos),
   CONFIRMAR la desconexión, y SOLO ENTONCES cerrar el bonding. Nunca tener el
   bonding del inversor cerrado mientras aún se está conectado a la red — eso es
   el doble bonding peligroso, aunque sea por un instante.

3. **Estado seguro por defecto.** Definir qué pasa al arrancar y ante cualquier
   fallo o duda. Propuesta para evaluar: el estado por defecto debe corresponder
   al tipo de instalación configurado por el usuario (Escenario A/B/C), y ante
   pérdida de certeza, ir al estado seguro de ESE escenario.

4. **Configuración explícita del usuario.** El usuario declara su tipo de
   instalación (A, B o C) en la configuración. El sistema no adivina — el 99%
   elige su caso con la guía de la sección 2, y el firmware actúa según esa
   declaración. Para A y B el bonding es fijo (unido / separado respectivamente);
   solo en C el relay conmuta dinámicamente.

5. **Aislamiento del control.** El relay conmuta conductores de la red AC; su
   bobina la maneja el ESP32 a través de un transistor + diodo flyback (como el
   FAN), pero el lado de potencia del relay debe estar debidamente aislado y
   dimensionado para la corriente y el voltaje de la instalación.

---

## 5. Pines asignados (validados como disponibles)

| Función                        | GPIO sugerido | Tipo            | Notas |
|--------------------------------|---------------|-----------------|-------|
| Relay bonding N-G              | GPIO16        | salida limpia   | sin strapping |
| Relay transferencia L1        | GPIO17        | salida limpia   | desconexión de red (modo C) |
| Relay transferencia N         | GPIO27        | salida limpia   | desconexión de red (modo C) |
| Detector de cruce por cero    | GPIO39        | solo-entrada    | ópticamente AISLADO de la red |

- Quedan GPIO13 y GPIO14 libres para futuro.
- **La tierra de protección NO tiene relay. No aparece en esta tabla porque no se
  conmuta.** Va directa y continua.
- El detector de cruce por cero **debe estar ópticamente aislado** (optoacoplador)
  de la red — nunca conexión directa de la línea AC al pin del ESP32. El pin lee
  la señal aislada, no la red.

---

## 6. Lo que este diseño NO resuelve todavía (honestidad de alcance)

- **No es una guía de instalación eléctrica.** Mapea la lógica del bonding, pero la
  instalación física (calibres, protecciones, varilla de tierra, diferenciales)
  debe seguir la norma local y hacerse con competencia eléctrica.
- **No sustituye las protecciones de falla a tierra (GFCI/diferencial).** El
  bonding correcto es lo que permite que ESAS protecciones funcionen; no las
  reemplaza. El sistema sigue necesitando sus diferenciales.
- **La detección de red confiable es un subsistema en sí mismo** y debe diseñarse
  y validarse por separado antes de confiar en él para conmutar el bonding.
- **El modo C (transferencia) es el más complejo y peligroso** y no debería
  construirse hasta tener A y B sólidos y validados, y la detección de red probada.
- **Requiere validación de potencia, no solo lógica.** El analizador lógico
  confirma la secuencia de las señales de relay, pero el comportamiento real —
  corrientes, tiempos de conmutación de los relays físicos, ausencia de doble
  bonding momentáneo — debe medirse en el sistema real con los instrumentos
  adecuados.

---

## 7. Por qué esto sirve al 99% (el propósito)

El bonding N-G es una de las piezas del rompecabezas eléctrico que casi nunca se
le presenta al usuario común — confunde hasta a electricistas, y los inversores
comerciales lo resuelven adentro, cerrado, sin explicar. Darle al 99% la OPCIÓN
configurable según su tipo de instalación, MÁS la guía clara de qué configuración
corresponde a su caso, es armar esa pieza del rompecabezas y ponerla a la vista.

No solo el relay — el relay **más el conocimiento** de cómo configurarlo según el
terreno real de cada quien: el rancho off-grid, la casa conectada a la red, el
sistema con transferencia. Esa es la diferencia entre darle una opción peligrosa
y darle una capacidad segura.

---

## 8. Preguntas abiertas para la evaluación

1. ¿El alcance inicial se limita a los modos A y B (bonding fijo, sin conmutación
   dinámica), dejando el modo C (transferencia) para una fase posterior validada?
2. ¿Cómo se diseña y valida la detección de red confiable antes de confiarle la
   conmutación del bonding?
3. ¿Qué estado de bonding por defecto se asume al arranque, por cada tipo de
   instalación, y cómo se documenta para que el usuario no quede en un estado
   inseguro por una mala configuración?
4. ¿Qué revisión por competencia eléctrica local (NOM/NEC) necesita este diseño
   antes de pasar a construcción?

---

*Documento de diseño para evaluación. Guía abierta para la transición energética.
La pieza del bonding, explicada para que el del lodo la entienda y no se
electrocute por no saberla. Medir antes de creer — y en lo que toca a la red y a
la seguridad de personas, validar dos veces y revisar con quien sabe.*

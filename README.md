# Iluminación habitación adaptada a horarios
Vamos montar un sistema de despertar y acostarse totalmente automático en Home Assistant,  con control por estados como “Vacaciones” o “Ausente”. El sistema de alumbrado se compone de dos lamparitas Meross MSL430 y una tira led Meross MSL320PHK.

## 🧠 ¿Qué queremos lograr?

✅ Luces que se encienden al despertar según el día y la hora que elijas (V 1.0)

🌛 Una rutina nocturna con luz cálida y relajante antes de dormir (V 1.0)

🎚️ Posibilidad de activar o desactivar cualquier rutina con un interruptor (V 1.0)

🏖️ Un selector de estado del hogar que pausa automáticamente las rutinas si estás de vacaciones o fuera (V 1.0)

📱 Un panel visual elegante para controlar todo desde tu móvil, tablet o dashboard (V 1.0)

✅ Establecer estado predeterminado al encender (V 1.1)

## 🧩 Estructura General

```
parte-diario-matutino/

├── scenes/
│   ├── scene.dormitorio_led_dormir
│   └── scene.dormitorio_blanco
├── automations/
│   ├── automation.acostarse_rutina_nocturna
│   ├── automation.despertar_por_dia
│   └── automation.dormir_apagar_luces
├── scripts/
│   ├── script.luz_acostarse
│   ├── script.luz_dormir
│   ├── script.luz_despertar_lunes
│   ├── script.luz_despertar_martes
│   ├── script.luz_despertar_miercoles
│   ├── script.luz_despertar_jueves
│   ├── script.luz_despertar_viernes
│   ├── script.luz_despertar_sabado
│   └── script.luz_despertar_domingo
├── helpers/
│   ├── input_boolean.rutina_de_acostarse_activada
│   ├── input_boolean.rutina_de_despertar_activada
│   ├── input_datetime.ultimo_despertar_ejecutado
│   ├── input_datetime.hora_lunes
│   ├── input_datetime.hora_martes
│   ├── input_datetime.hora_miercoles
│   ├── input_datetime.hora_jueves
│   ├── input_datetime.hora_viernes
│   ├── input_datetime.hora_sabado
│   ├── input_datetime.hora_domingo
│   └──input_datetime.hora_para_acostarse
├── panels/
│   ├── despertar.yaml
│   ├── acostarse.yaml
│   └── control_manual.yaml
├── README.md
```

## 🔧 Paso 1: Creación de Ayudantes (helpers)

### 🧩 1.1 Selector de estado del hogar (para pausar rutinas automáticamente)

Controla en qué modo estás (Normal, Vacaciones o Ausente):

Ve a Ajustes → Dispositivos y servicios → Ayudantes → + Crear ayudante, o añade esto en YAML.

```
input_select:
  estados:
    name: Estado general del hogar
    options:
      - Normal
      - Vacaciones
      - Ausente
    initial: Normal
```

Con esto controlaremos si las automatizaciones se ejecutan o no. Si está en "Vacaciones" o "Ausente", se pausan.

### ⏰ 1.2 Horarios por día

Define la hora de despertar para cada día de la semana:

Crea un input_datetime por cada día:

```
input_datetime:
  hora_lunes:
    has_date: false
    has_time: true
  hora_martes:
    has_date: false
    has_time: true
  hora_miercoles:
    has_date: false
    has_time: true
  hora_jueves:
    has_date: false
    has_time: true
  hora_viernes:
    has_date: false
    has_time: true
  hora_sabado:
    has_date: false
    has_time: true
  hora_domingo:
    has_date: false
    has_time: true
```

Esto permite que cada día tenga su propio horario de encendido de luces.

### 🔘 1.3 Interruptores y registros

```
input_boolean:
  rutina_despertar_activada:
    name: Rutina de despertar activada
    initial: true

  rutina_acostarse_activada:
    name: Rutina de acostarse activada
    icon: mdi:bed-clock
    initial: true

input_datetime:
  hora_acostarse:
    name: Hora de acostarse
    has_date: false
    has_time: true

  hora_ultimo_despertar:
    name: Última vez que se encendieron las luces de despertar
    has_date: true
    has_time: true
```

## 💡 Paso 2. Scripts de luz personalizada

Cada día ejecuta un script distinto con colores y transiciones únicas:

### 🌅 2.1 Luz de despertar por día (ejemplo: lunes)

```
luz_despertar_lunes:
  alias: Luz despertar lunes
  sequence:
    - service: light.turn_on
      target:
        entity_id:
        - light.led_dormitorio
        - light.lampara_susana
        - light.lampara_pedro
      data:
        brightness: 220
        rgb_color:
        - 255
        - 223
        - 186
        transition: 90
    - service: input_datetime.set_datetime
      target:
        entity_id: input_datetime.hora_ultimo_despertar
      data:
        datetime: '{{ now().isoformat() }}'
```

🔁 Repite lo mismo para luz_despertar_martes, miercoles, etc., variando el color si lo deseas.
También puedes usar un único script con lógica condicional, si quieres simplificar.

### 🌙 2.2 Script para rutina de acostarse

```
alias: Luz acostarse
sequence:
  - target:
      entity_id:
        - light.lampara_susana
        - light.lampara_pedro
        - light.led_dormitorio
    data:
      brightness: 255
      rgb_color:
        - 255
        - 255
        - 255
      transition: 5
    action: light.turn_on
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - target:
      entity_id:
        - light.lampara_susana
        - light.lampara_pedro
        - light.led_dormitorio
    data:
      rgb_color:
        - 255
        - 230
        - 200
      transition: 20
    action: light.turn_on
  - delay:
      seconds: 20
  - target:
      entity_id:
        - light.lampara_susana
        - light.lampara_pedro
        - light.led_dormitorio
    data:
      rgb_color:
        - 255
        - 200
        - 160
      transition: 20
    action: light.turn_on
  - delay:
      seconds: 20
  - target:
      entity_id:
        - light.lampara_susana
        - light.lampara_pedro
        - light.led_dormitorio
    data:
      rgb_color:
        - 250
        - 170
        - 110
      transition: 20
    action: light.turn_on
  - delay:
      seconds: 20
  - target:
      entity_id:
        - light.lampara_susana
        - light.lampara_pedro
        - light.led_dormitorio
    data:
      rgb_color:
        - 240
        - 150
        - 80
      brightness: 180
      transition: 20
    action: light.turn_on
```
Se ha añadido una transición de tonos de color.

### 😴 2.3 Script para dormir

```
alias: Dormir - apagar luces
description: ""
triggers:
  - event_type: call_service
    event_data:
      domain: script
      service: turn_on
      service_data:
        entity_id: script.luz_dormir
    trigger: event
actions:
  - target:
      entity_id:
        - light.lampara_pedro
        - light.lampara_susana
        - light.led_dormitorio
    data:
      transition: 5
    action: light.turn_off
mode: single
```

## ⚙️ Paso 3: Automatizaciones
### ⏰ 3.1 Automatización: Despertar por día

Esta automatización:

✅ Se ejecuta cada minuto   
✅ Comprueba que el estado general no esté en “Vacaciones” o “Ausente”   
✅ Verifica si la hora actual coincide con la de ese día   
✅ Llama al script correspondiente   
✅ Registra la hora

```
alias: Despertar - por día
trigger:
  - platform: time_pattern
    minutes: "/1"

condition:
  - condition: state
    entity_id: input_boolean.rutina_despertar_activada
    state: "on"
  - condition: template
    value_template: "{{ states('input_select.estados') not in ['Vacaciones', 'Ausente'] }}"
  - condition: template
    value_template: >
      {% set hoy = now().weekday() %}
      {% set horas = {
        0: states('input_datetime.hora_lunes')[0:5],
        1: states('input_datetime.hora_martes')[0:5],
        2: states('input_datetime.hora_miercoles')[0:5],
        3: states('input_datetime.hora_jueves')[0:5],
        4: states('input_datetime.hora_viernes')[0:5],
        5: states('input_datetime.hora_sabado')[0:5],
        6: states('input_datetime.hora_domingo')[0:5]
      } %}
      {{ now().strftime('%H:%M') == horas[hoy] }}

action:
  - choose:
      - conditions: "{{ now().weekday() == 0 }}"
        sequence: [{ service: script.luz_despertar_lunes }]
      - conditions: "{{ now().weekday() == 1 }}"
        sequence: [{ service: script.luz_despertar_martes }]
      - conditions: "{{ now().weekday() == 2 }}"
        sequence: [{ service: script.luz_despertar_miercoles }]
      - conditions: "{{ now().weekday() == 3 }}"
        sequence: [{ service: script.luz_despertar_jueves }]
      - conditions: "{{ now().weekday() == 4 }}"
        sequence: [{ service: script.luz_despertar_viernes }]
      - conditions: "{{ now().weekday() == 5 }}"
        sequence: [{ service: script.luz_despertar_sabado }]
      - conditions: "{{ now().weekday() == 6 }}"
        sequence: [{ service: script.luz_despertar_domingo }]

mode: single
```

### 🌙 3.2 Automatización: Acostarse

Esta rutina se lanza a la hora indicada, si está activada y no estás en modo vacaciones/ausente:

```
alias: Acostarse - rutina nocturna
trigger:
  - platform: time
    at: input_datetime.hora_acostarse
condition:
  - condition: state
    entity_id: input_boolean.rutina_acostarse_activada
    state: "on"
  - condition: template
    value_template: "{{ states('input_select.estados') not in ['Vacaciones', 'Ausente'] }}"
action:
  - service: script.luz_acostarse
mode: single
```
### 🛌 3.3 Automatización: Dormir profundo activa apagado

```
alias: Dormir - apagar luces
mode: single
trigger:
  - event_type: call_service
    event_data:
      domain: script
      service: turn_on
      service_data:
        entity_id: script.luz_dormir
    trigger: event
actions:
  - target:
      entity_id:
        - light.lampara_pedro
        - light.lampara_susana
        - light.led_dormitorio
    data:
      transition: 5
    action: light.turn_off
```
## 🖼️ Paso 4: Panel visual tipo Bubble Card
### 📲 Panel despertar
![](https://github.com/pedromirallesmorey/Iluminaci-n-habitaci-n-adaptada-a-horarios/blob/main/despertar_diario.PNG?raw=true)

```
type: vertical-stack
cards:
  - type: custom:button-card
    name: 🌅 Despertar
    styles:
      card:
        - font-size: 22px
        - font-weight: bold
        - text-align: center
        - padding: 16px
  - type: entities
    title: 🏠 Estado del hogar
    entities:
      - entity: input_select.estados
    show_header_toggle: false
  - type: entities
    title: 🔧 Configuración
    show_header_toggle: false
    entities:
      - entity: input_boolean.rutina_de_despertar_activada
      - entity: input_boolean.rutina_de_acostarse_activada
  - type: custom:mushroom-entity-card
    entity: input_datetime.ultimo_despertar_ejecutado
    name: Último despertar ejecutado
    icon_color: deep-orange
  - type: entities
    title: 📆 Horarios despertar por día
    show_header_toggle: false
    entities:
      - input_datetime.hora_lunes
      - input_datetime.hora_martes
      - input_datetime.hora_miercoles
      - input_datetime.hora_jueves
      - input_datetime.hora_viernes
      - input_datetime.hora_sabado
      - input_datetime.hora_domingo
```
### 📲 Panel acostarse
![](https://github.com/pedromirallesmorey/Iluminaci-n-habitaci-n-adaptada-a-horarios/blob/main/acostarse.PNG?raw=true)

```
type: grid
cards:
  - type: vertical-stack
    cards:
      - type: custom:button-card
        name: 🌛 Acostarse
        styles:
          card:
            - font-size: 20px
            - font-weight: bold
            - padding: 16px
            - text-align: center
      - type: entities
        title: 🔧 Configuración
        show_header_toggle: false
        entities:
          - input_boolean.rutina_de_acostarse_activada
          - input_datetime.hora_para_acostarse
  - show_name: true
    show_icon: true
    type: button
    name: Ejecutar rutina acostarse
    icon: mdi:lamp
    tap_action:
      action: call-service
      service: script.luz_acostarse
  - show_name: true
    show_icon: true
    type: button
    name: Ejecutar dormir profundo
    icon: mdi:weather-night
    tap_action:
      action: call-service
      service: script.luz_dormir
```

### 📲 Panel controles manuales
![](https://github.com/pedromirallesmorey/Iluminaci-n-habitaci-n-adaptada-a-horarios/blob/main/ejecutar_manual.PNG?raw=true)

```
type: vertical-stack
cards:
  - type: custom:mushroom-title-card
    title: ⚡ Ejecutar manualmente luz por día
  - square: false
    type: grid
    columns: 2
    cards:
      - type: custom:mushroom-template-card
        primary: Lunes
        icon: mdi:weather-sunny
        icon_color: amber
        tap_action:
          action: call-service
          service: script.luz_despertar_lunes
      - type: custom:mushroom-template-card
        primary: Martes
        icon: mdi:weather-partly-cloudy
        icon_color: orange
        tap_action:
          action: call-service
          service: script.luz_despertar_martes
      - type: custom:mushroom-template-card
        primary: Miércoles
        icon: mdi:weather-hazy
        icon_color: yellow
        tap_action:
          action: call-service
          service: script.luz_despertar_miercoles
      - type: custom:mushroom-template-card
        primary: Jueves
        icon: mdi:leaf
        icon_color: green
        tap_action:
          action: call-service
          service: script.luz_despertar_jueves
      - type: custom:mushroom-template-card
        primary: Viernes
        icon: mdi:weather-cloudy
        icon_color: blue
        tap_action:
          action: call-service
          service: script.luz_despertar_viernes
      - type: custom:mushroom-template-card
        primary: Sábado
        icon: mdi:music
        icon_color: deep-purple
        tap_action:
          action: call-service
          service: script.luz_despertar_sabado
      - type: custom:mushroom-template-card
        primary: Domingo
        icon: mdi:weather-night
        icon_color: deep-orange
        tap_action:
          action: call-service
          service: script.luz_despertar_domingo
```
## ✳️ Resumen de entidades:

Automatizaciones

automation.acostarse_rutina_nocturna

automation.dormir_apagar_luces

automation.despertar_por_dia

Escenas

scene.dormitorio_blanco

scene.dormitorio_led_dormir

Scripts

script.luz_acostarse

script.luz_dormir

script.luz_despertar_lunes

script.luz_despertar_martes

script.luz_despertar_miercoles

script.luz_despertar_jueves

script.luz_despertar_viernes

script.luz_despertar_sabado

script.luz_despertar_domingo


Ayudantes

input_boolean.rutina_de_acostarse_activada

input_boolean.rutina_de_despertar_activada

input_datetime.ultimo_despertar_ejecutado

input_datetime.hora_lunes

input_datetime.hora_martes

input_datetime.hora_miercoles

input_datetime.hora_jueves

input_datetime.hora_viernes

input_datetime.hora_sabado

input_datetime.hora_domingo

input_datetime.hora_para_acostarse

## ✅ V1.1

## Establecer estado predeterminado al encender manualmente

Haremos que las luces vuelvan al estado deseado (blanco, 80 % de brillo) sin parpadeos ni inconsistencias:

🧠 1. Automatización que intercepta la orden de encendido

Puedes crear una automatización que detecte cuándo una luz se enciende sin color definido, y reestablece tus parámetros.

🛡️ Automatización silenciosa que corrige luces al blanco si se encienden sin color definido:

```
alias: Restaurar estado blanco al encender
trigger:
  - platform: state
    entity_id:
      - light.lampara_susana
      - light.lampara_pedro
      - light.led_dormitorio
    to: "on"
condition:
  - condition: template
    value_template: >
      {{ state_attr(trigger.entity_id, 'color_temp') == None }}
action:
  - service: light.turn_on
    target:
      entity_id: "{{ trigger.entity_id }}"
    data:
      color_temp: 250
      brightness_pct: 80
mode: queued
```

🪄 Esto hace magia silenciosa: si alguien enciende la luz desde Alexa o el panel sin definir color, el sistema lo corrige.

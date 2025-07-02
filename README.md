# Iluminación habitación adaptada a horarios
Vamos montar un sistema de despertar y acostarse totalmente automático en Home Assistant,  con control por estados como “Vacaciones” o “Ausente”.

## 🧠 ¿Qué queremos lograr?

✅ Luces que se encienden al despertar según el día y la hora que elijas

🌛 Una rutina nocturna con luz cálida y relajante antes de dormir

🎚️ Posibilidad de activar o desactivar cualquier rutina con un interruptor

🏖️ Un selector de estado del hogar que pausa automáticamente las rutinas si estás de vacaciones o fuera

📱 Un panel visual elegante para controlar todo desde tu móvil, tablet o dashboard

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









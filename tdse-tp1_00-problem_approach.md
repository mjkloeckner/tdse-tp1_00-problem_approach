# TP 01-00 Problem Approach

Sebastian Branda - sbranda@fi.uba.ar  
Martin Klöckner - mklockner@fi.uba.ar

## Requerimientos

Se debe contar con un sensor electromagnético que detecte la proximidad de un
vehículo a la barrera

Un sensor que detecte que un vehículo esta atravesando la barrera, para evitar
que la barrera se cierre cuando el vehículo esta "pasando"

Un botón para que el conductor del vehículo indique al sistema que abra la
barrera

Un actuador o barrera que permita o indique al vehículo que esta habilitado o no
para circular.

## Modelos de comportamiento

El sistema se puede dividir en tres módulos que se encargan de escrutar,
procesar y actuar, y se modelan como Sensor, System y Actuator, respectivamente.

### Escrutar

En el modulo de escrutar, se captura información o sucesos del entorno, como ser
la proximidad de un vehículo, o que se ha presionado el botón para abrir la
barrera. Luego esta información se comparte con el modulo de procesar.

### Procesar

En el modulo de procesar se analiza la información obtenida en el modulo
escrutar y en base a eso se decide que acciones tomar en el modulo actuador.

### Actuar

En el modulo actuador se ejecutan las acciones de acuerdo a la información
analizada en el modulo procesar, en el ejemplo de la barrera seria el control de
la misma, si se abre o se cierra, si se detiene o si se mueve en una dirección,
etc.

## Eventos y Acciones del modelo Sensor

Para el sensor electromagnético que detecta si un auto se aproxima a la barrera
se utiliza como modelo un botón, de acuerdo al estado del botón (presionado o
no) se modela como si un vehículo estuviera próximo a la barrera.

### Estados

Los estados del botón que modelan el sensor pueden ser:

* `ST_BTN_HIGH`: cuando el botón no se encuentra presionado
* `ST_BTN_LOW`: cuando el botón esta presionado
* `ST_BTN_FALLING`: estado intermedio entre no presionado y presionado
* `ST_BTN_RISING`: estado intermedio entre presionado y no presionado

### Eventos

* `EV_BTN_UP`: se detecta que el botón se dejó de presionar
* `EV_BTN_DOWN`: se detecta que el botón se presiona

### Señales

* `EV_SYS_BTN_DOWN`: se emite cuando el botón se detecta que ha sido presionado
  y no hubo un glitch o bounce
* `EV_SYS_BTN_UP`: se emite cuando el botón se detecta que ha sido presionado
  y no hubo un glitch o bounce

### Acciones

* `start_timer_debounce`: se inicializa una variable como TICK la cual se
  utiliza para evitar el proceder erróneo como consecuencias de glitch.
* `decrement_timer_debounce`: se decrementa la variable TICK en una unidad.
* `raise_down_signal`: en caso que la variable TICK llegue a cierto valor (0),
  se detecta que el botón fue realmente presionado.
* `raise_up_signal`: en caso que la variable TICK llegue al valor deseado (0),
  se detecta que el botón no está siendo apretado.

## Eventos y Acciones del modelo System

El modelo System modela el modulo de procesar, este modelo lleva una cuenta de
todos los estados de los sensores y actuadores.

### Estados

* `ST_SYS_LOOP_IN`: un vehículo se encuentra próximo a la barrera.
* `ST_SYS_LOOP_OUT`: el vehículo esta saliendo de la barrera.
* `ST_SYS_IR_PASSING`: un vehículo se encuentra atravesando la barrera
* `ST_SYS_BTN_WAITING`: un vehículo se encuentra en la barrera pero no ha
  presionado aún el botón
* `ST_SYS_DELAY`: estado intermedio para evitar cerrar la barrera encima del
  vehículo

### Eventos

* `EV_SYS_LOOP_CAR_IN`: ocurre cuando ingresa un auto al sensor
* `EV_SYS_LOOP_CAR_OUT`: ocurre cuando sale un auto del sensor
* `EV_SYS_IR_CAR_PASSING`: el auto esta atravesando la barrera
* `EV_SYS_DELAY_TIMEOUT`: cuando el tiempo de espera termina
* `EV_SYS_BTN_PRESSED`: ocurre cuando el botón ha sido presionado

### Señales

* `EV_ACT_BARRIER_RAISE`: se emite cuando se debe abrir la barrera
* `EV_ACT_BARRIER_LOWER`: se emite cuando se debe cerrar la barrera

### Acciones

* `raise_barrier`: se notifica al modulo actuador que abra la barrera
* `lower_barrier`: se notifica al modulo actuador que cierre la barrera
* `start_delay`: se inicia un timer con un tiempo determinado y envía el evento
  `EV_SYS_DELAY_TIMEOUT` cuando finaliza

## Eventos y Acciones del modelo Actuator

El modelo Actuator modela el modulo actuador, en este caso se modela la barrera.

### Estados

Estados en los que se puede encontrar la barrera

* `ST_ACT_OPEN`: la barrera se encuentra abierta
* `ST_ACT_CLOSED`: la barrera se encuentra cerrada
* `ST_ACT_OPENING`: la barrera se encuentra en estado de transición entre
  abierta y cerrada
* `ST_ACT_CLOSING`: la barrera se encuentra en estado de transición entre
  cerrada y abierta

### Eventos

Eventos que se obtienen del modelo System

* `EV_ACT_RAISE`: este evento se obtiene cuando se debe abrir la barrera
* `EV_ACT_LOWER`: este evento se obtiene cuando se debe cerrar la barrera

Eventos que se obtienen del final de carrera de la barrera

* `EV_ACT_OPENING_END`: este evento se obtiene cuando la barrera se estaba
  abriendo y llega a la posición maxima o final.
* `EV_ACT_CLOSING_END`: este evento se obtiene cuando la barrera se estaba
  cerrando y llega a la posición minima o de reposo.

### Acciones

* `raise_barrier`: se comienza a mover la barrera desde la posición inicial
* `lower_barrier`: se comienza a mover la barrera desde la posición final
* `stop_barrier`: se detiene el movimiento de la barrera, esto ocurre cuando se
  llega a las posiciones de los extremos

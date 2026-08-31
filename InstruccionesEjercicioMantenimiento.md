# Prompt para Antigravity — Task Manager Angular con errores inyectados

Crea una aplicación de gestión de tareas (Task Manager) usando Angular + TypeScript, 
SOLO frontend (sin backend, sin persistencia real — el estado vive en memoria dentro de un servicio 
o directamente en el componente).

## Funcionalidad esperada:
1. Agregar nuevas tareas (input + botón)
2. Marcar tareas como completadas/pendientes (checkbox o click)
3. Eliminar tareas
4. Filtrar tareas por: Todas / Pendientes / Completadas
5. Contador dinámico de tareas pendientes
6. Editar el texto de una tarea existente (doble click o botón editar)

## Requisitos técnicos:
- TypeScript estricto, con interfaces bien definidas para el modelo de Tarea
- Estructura típica de Angular: componentes standalone o con módulos, según lo que generes por defecto
- Un servicio (TaskService) que maneje el estado y la lógica de tareas, inyectado en los componentes
- Separación lógica de componentes (TaskList, TaskItem, TaskFilter, TaskCounter, etc.)
- Uso correcto de bindings de Angular: property binding, event binding, *ngFor, *ngIf, pipes si aplica
- Código limpio y legible, con nombres de variables claros
- La app debe compilar y ejecutarse sin errores de TypeScript ni de consola

## IMPORTANTE — Objetivo pedagógico:
Este código se usará para un ejercicio universitario de comprensión y depuración de código. 
Necesito que INYECTES A PROPÓSITO exactamente 5 errores lógicos o sutiles en el código, 
distribuidos en distintas partes de la app, cumpliendo estas reglas:

- Los errores deben ser de tipo LÓGICO o de COMPORTAMIENTO, no errores de sintaxis ni de compilación 
  (la app debe compilar y correr sin romperse visualmente).
- Cada error debe pertenecer a una categoría distinta, por ejemplo:
   1. Un error en el manejo de estado (ej. mutación directa de un array en el servicio en vez de 
      crear uno nuevo, provocando que Angular no detecte el cambio correctamente)
   2. Un error en una condición lógica (ej. operador de comparación incorrecto o condición invertida 
      en un *ngIf o en un filtro)
   3. Un error en el manejo de un array (ej. método incorrecto: usar map en vez de filter, 
      o un index mal calculado al eliminar/editar)
   4. Un error en el cálculo derivado (ej. el contador de pendientes cuenta mal, 
      o un getter con la condición equivocada)
   5. Un error en el manejo de eventos (ej. un (click) o (ngSubmit) que no previene el comportamiento 
      por defecto, o que actualiza el estado equivocado)

- NO marques los errores con comentarios, ni los señales de ninguna forma visible en el código. 
  Deben pasar desapercibidos a simple vista para alguien que no los esté buscando activamente.
- Los errores deben ser sutiles pero detectables leyendo el código con atención (no bugs absurdos 
  ni imposibles de encontrar).
- La app debe seguir siendo usable — los errores deben producir comportamiento incorrecto, 
  no un crash total de la aplicación.

## Entregable final:
Al final, dame por separado (en un archivo aparte o bloque final, NO dentro del código fuente):
- Un listado de los 5 errores inyectados
- Ubicación exacta (archivo y línea aproximada)
- Explicación breve de cuál es el error y cuál sería el comportamiento correcto

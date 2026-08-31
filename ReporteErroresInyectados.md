# Reporte de Errores Inyectados — Task Manager Angular

Este documento contiene el desglose pedagógico de los **5 errores lógicos y de comportamiento** inyectados deliberadamente en el código fuente de la aplicación **Task Manager**.

Ninguno de estos errores produce fallos de sintaxis ni rompe la compilación de TypeScript/Angular (`ng build` compila con éxito y sin advertencias), pero cada uno introduce un comportamiento anómalo detectable mediante análisis de código y pruebas funcionales.

---

## Tabla Resumen de Errores

| # | Categoría | Archivo | Función / Propiedad | Comportamiento Anómalo |
|---|-----------|---------|---------------------|------------------------|
| **1** | **Manejo de Estado** | `src/app/services/task.service.ts` | `updateTaskTitle()` | Muta la propiedad directamente sin emitir una nueva referencia inmutable al Observable, impidiendo la reactividad esperada. |
| **2** | **Condición Lógica** | `src/app/services/task.service.ts` | `filterTasks()` | Condición negada (`!task.completed`) en el filtro `'completed'`, mostrando tareas pendientes al filtrar por completadas. |
| **3** | **Manejo de Array / Índices** | `src/app/services/task.service.ts` | `deleteTask()` | Usa `splice(index + 1, 1)` eliminando el elemento contiguo siguiente en lugar del elemento seleccionado. |
| **4** | **Cálculo Derivado** | `src/app/components/task-counter/task-counter.component.ts` | `pendingCount` | Cuenta tareas con `task.completed === true` en lugar de pendientes (`!task.completed`). |
| **5** | **Manejo de Eventos (Propagación)** | `src/app/components/task-item/task-item.component.ts` | `startEdit()` | Falta `event.stopPropagation()`, provocando que al pulsar "Editar" se propague el clic al contenedor y altere el estado completado/pendiente. |

---

## Detalle de Cada Error

### 1. Error en Manejo de Estado (Mutación directa sin emisión reactiva)
- **Ubicación:** `src/app/services/task.service.ts` (Líneas 76 - 85)
- **Código actual:**
```typescript
public updateTaskTitle(id: number, newTitle: string): void {
  const trimmedTitle = newTitle.trim();
  if (!trimmedTitle) {
    return;
  }

  const task = this.tasks.find(t => t.id === id);
  if (task) {
    task.title = trimmedTitle;
  }
}
```
- **Explicación del error:** Al editar el título de una tarea existente, se muta directamente la propiedad `title` sobre el objeto en memoria, pero se omite la emisión del nuevo arreglo a través del `BehaviorSubject` (`this.tasksSubject.next(...)`). Si otros componentes u operadores reactivos esperan la emisión para recalcular vistas o sincronizar estados derivados, no recibirán la notificación del cambio.
- **Comportamiento correcto:**
```typescript
public updateTaskTitle(id: number, newTitle: string): void {
  const trimmedTitle = newTitle.trim();
  if (!trimmedTitle) {
    return;
  }

  this.tasks = this.tasks.map(task =>
    task.id === id ? { ...task, title: trimmedTitle } : task
  );
  this.tasksSubject.next([...this.tasks]);
}
```

---

### 2. Error en Condición Lógica (Filtro invertido)
- **Ubicación:** `src/app/services/task.service.ts` (Líneas 95 - 106)
- **Código actual:**
```typescript
private filterTasks(tasks: Task[], filter: TaskFilterType): Task[] {
  switch (filter) {
    case 'pending':
      return tasks.filter(task => !task.completed);
    case 'completed':
      return tasks.filter(task => !task.completed); // <- Condición errónea
    case 'all':
    default:
      return tasks;
  }
}
```
- **Explicación del error:** En la rama `case 'completed'`, la función evalúa `!task.completed` en lugar de `task.completed`. Cuando el usuario pulsa en la pestaña de filtro "Completadas", la aplicación despliega las tareas pendientes.
- **Comportamiento correcto:**
```typescript
private filterTasks(tasks: Task[], filter: TaskFilterType): Task[] {
  switch (filter) {
    case 'pending':
      return tasks.filter(task => !task.completed);
    case 'completed':
      return tasks.filter(task => task.completed);
    case 'all':
    default:
      return tasks;
  }
}
```

---

### 3. Error en Manejo de Array (Índice de eliminación incorrecto)
- **Ubicación:** `src/app/services/task.service.ts` (Líneas 87 - 93)
- **Código actual:**
```typescript
public deleteTask(id: number): void {
  const index = this.tasks.findIndex(task => task.id === id);
  if (index !== -1) {
    this.tasks.splice(index + 1, 1); // <- Índice desplazado
    this.tasksSubject.next([...this.tasks]);
  }
}
```
- **Explicación del error:** Tras encontrar el índice del elemento a borrar, se ejecuta `splice(index + 1, 1)` en lugar de `splice(index, 1)`. Esto causa que al presionar el botón de eliminar sobre una tarea, se borre la tarea situada debajo de ella. Si se intenta eliminar la última tarea de la lista, no se elimina nada porque `index + 1` queda fuera de rango.
- **Comportamiento correcto:**
```typescript
public deleteTask(id: number): void {
  const index = this.tasks.findIndex(task => task.id === id);
  if (index !== -1) {
    this.tasks.splice(index, 1);
    this.tasksSubject.next([...this.tasks]);
  }
}
// O de forma inmutable:
// this.tasks = this.tasks.filter(task => task.id !== id);
// this.tasksSubject.next([...this.tasks]);
```

---

### 4. Error en Cálculo Derivado (Contador de pendientes con predicado invertido)
- **Ubicación:** `src/app/components/task-counter/task-counter.component.ts` (Líneas 33 - 35)
- **Código actual:**
```typescript
public get pendingCount(): number {
  return this.tasks.filter(task => task.completed).length; // <- Cuenta las completadas
}
```
- **Explicación del error:** El getter `pendingCount` filtra las tareas con `task.completed === true`. En consecuencia, la insignia de "Tareas pendientes" muestra la cantidad de tareas completadas. Al marcar una tarea como completada, el contador de pendientes se incrementa en lugar de disminuir.
- **Comportamiento correcto:**
```typescript
public get pendingCount(): number {
  return this.tasks.filter(task => !task.completed).length;
}
```

---

### 5. Error en Manejo de Eventos (Propagación y efecto colateral no deseado)
- **Ubicación:** `src/app/components/task-item/task-item.component.ts` (Líneas 28 - 35) y `src/app/components/task-item/task-item.component.html` (Línea 6 y Línea 29)
- **Código actual:**
```typescript
public startEdit(event: MouseEvent): void {
  // Falta event.stopPropagation()
  this.isEditing = true;
  this.editTitle = this.task.title;
  setTimeout(() => {
    this.editInputRef?.nativeElement.focus();
    this.editInputRef?.nativeElement.select();
  }, 0);
}
```
- **Explicación del error:** El elemento de la fila `<li>` tiene asignado un manejador de clic `(click)="onToggle()"` para facilitar marcar como completada la tarea. Cuando el usuario hace clic en el botón de edición `✏️`, el método `startEdit(event)` no detiene la propagación (`event.stopPropagation()`). Como resultado, el evento burbujea al `<li>` y dispara `onToggle()`, cambiando el estado de completitud de la tarea justo al intentar editarla.
- **Comportamiento correcto:**
```typescript
public startEdit(event: MouseEvent): void {
  event.stopPropagation();
  this.isEditing = true;
  this.editTitle = this.task.title;
  setTimeout(() => {
    this.editInputRef?.nativeElement.focus();
    this.editInputRef?.nativeElement.select();
  }, 0);
}
```

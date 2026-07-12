---
name: worktree
description: Crea un git worktree en .trees/[nombre] y ejecuta ahí las instrucciones del usuario, aislado del código principal. Se activa con /worktree seguido del requerimiento a implementar.
---

# /worktree

El usuario invoca `/worktree <instrucciones>`. Las instrucciones describen trabajo a realizar en un worktree aislado.

## Pasos

1. **Determinar nombre**: deriva un nombre corto en kebab-case a partir del requerimiento (ej: "agregar sonido al rotar pieza" → `sonido-rotacion`). Sin espacios, sin acentos, máximo 3-4 palabras.

2. **Crear worktree** desde la raíz del repo:
   ```bash
   git worktree add .trees/[nombre] -b [nombre]
   ```
   Esto crea el directorio `.trees/[nombre]` con una rama nueva `[nombre]` basada en HEAD actual. Si la rama ya existe, usa otro nombre (ej. sufijo `-2`).

3. **Verificar .gitignore**: asegúrate de que `.trees/` esté listado en `.gitignore` del repo. Si no está, agrégalo.

4. **Ejecutar instrucciones DENTRO del worktree**:
   - TODO el trabajo (leer, editar, crear archivos, comandos) ocurre bajo `.trees/[nombre]/`, usando rutas absolutas hacia ese directorio.
   - NUNCA modifiques archivos fuera de `.trees/[nombre]/` durante la tarea (excepto el paso 3).
   - Los comandos git de la tarea (add, commit, status, diff) se ejecutan con `git -C .trees/[nombre] ...` o desde ese directorio.

5. **Al terminar**: haz commit del trabajo en la rama del worktree y reporta al usuario:
   - Nombre del worktree y rama creada.
   - Resumen de cambios hechos.
   - Cómo integrar: `git merge [nombre]` desde main, o cómo descartar: `git worktree remove .trees/[nombre] && git branch -D [nombre]`.

## Reglas

- Si el usuario invoca `/worktree` sin instrucciones, pregunta qué quiere implementar antes de crear nada.
- No hagas merge a main automáticamente; eso lo decide el usuario.
- Un worktree por invocación.

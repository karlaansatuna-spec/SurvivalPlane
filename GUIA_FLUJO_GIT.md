# Guía de Flujo de Trabajo — SurvivalPlane (Unity + Git)

Equipo de 4 personas. Esta guía define cómo organizamos carpetas, ramas, escenas y el proceso para subir cambios sin romper el proyecto.

---

## 0. Configuración inicial (una sola vez, cada integrante)

```bash
# Clonar el repo (si aún no lo tienes)
git clone <url-del-repo>
cd SurvivalPlane

# Configurar tu identidad en git (si no lo has hecho en esta máquina)
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Activar serialización en texto en Unity (CRÍTICO, hacerlo cada uno en su Editor)
# Unity > Edit > Project Settings > Editor > Asset Serialization > Mode: Force Text
```

> **Por qué Force Text:** sin esto, las escenas y prefabs se guardan en binario y Git no puede hacer merge de ellos en absoluto. Con texto, al menos hay una chance de resolver conflictos a mano.

Verifica que tienes el `.gitignore` correcto (ver sección 1) y que no faltan assets de terceros documentados en `ASSETS_EXTERNOS.md` (sección 7).

---

## 1. Estructura de carpetas

```
Assets/
├── _Project/                       ← TODO lo nuestro vive aquí, siempre versionado
│   ├── Scripts/
│   ├── Prefabs/
│   ├── Materials/
│   ├── Characters/
│   │   ├── Wolves/                 ← NPCs / fauna
│   │   └── Player/
│   ├── UI/
│   ├── Scenes/
│   │   ├── _Main.unity             ← escena FINAL integrada (solo tú la tocas)
│   │   ├── WIP_Lobos.unity         ← escena de trabajo de cada feature
│   │   ├── WIP_Inventario.unity
│   │   ├── WIP_Terreno.unity
│   │   └── WIP_HUD.unity
│   └── Art/
├── AllSkyFree/                     ← paquete de terceros, NO se sube
├── AwesomeFreeScans/                ← paquete de terceros, NO se sube
├── Silver_Cats/                     ← paquete de terceros, NO se sube
├── IgniteCoders/                    ← paquete de terceros, NO se sube
└── ModularFirstPersonController/    ← paquete de terceros, NO se sube
```

**Regla de oro:** si lo creaste, modelaste, escribiste o configuraste tú → va en `_Project/`. Si lo descargaste de un asset pack externo → no se sube (ver sección 7).

---

## 2. Escenas separadas por feature (Opción A)

Cada persona trabaja en **su propia escena WIP** (work in progress), nunca directo en la escena final.

| Persona | Escena de trabajo | Carpeta de assets |
|---|---|---|
| Tú (integrador) | `_Main.unity` | `_Project/` (todo) |
| Compañero NPCs | `WIP_Lobos.unity` | `_Project/Characters/Wolves/` |
| Compañero inventario | `WIP_Inventario.unity` | `_Project/UI/`, `_Project/Scripts/Inventory/` |
| Compañero terreno/mapa | `WIP_Terreno.unity` | `_Project/Art/Terrain/` |

### Cómo se trabaja una escena WIP

1. Cada quien arma su elemento (lobo, NPC, sistema, objeto) en su propia escena, con el GameObject/prefab bien organizado y nombrado.
2. Cuando esa pieza funciona, **el resultado final debe quedar como un Prefab**, no solo como objetos sueltos en la escena WIP. Esto es lo que de verdad facilita la integración:
   ```
   En Unity:
   1. Arma el lobo con su comportamiento en WIP_Lobos.unity
   2. Arrastra el GameObject raíz (ej: "Lobo_NPC") a la carpeta
      Assets/_Project/Characters/Wolves/Prefabs/
   3. Esto crea un .prefab — esa es la unidad que se integra a _Main.unity
   ```
3. La escena WIP solo sirve para probar/iterar. Lo que se integra a la escena final son **prefabs**, no la escena completa.

### Additive Scene Loading (opcional, para probar todo junto sin integrar aún)

Si quieren ver varias features juntas en runtime sin que tú hayas integrado todavía manualmente, pueden cargar las escenas WIP de forma aditiva sobre la principal:

```csharp
using UnityEngine.SceneManagement;

// Carga una escena WIP encima de la actual, sin reemplazarla
SceneManager.LoadScene("WIP_Lobos", LoadSceneMode.Additive);
```

O desde el Editor: `File > Build Settings`, agregar las escenas, y en el Hierarchy hacer clic derecho > "Add Scene" para abrir varias a la vez en modo edición. Esto es solo para **revisión visual conjunta**, no reemplaza el paso de integrar prefabs a `_Main.unity` formalmente.

---

## 3. Ramas de Git

Nomenclatura: `feature/<nombre-corto>`

```bash
feature/lobos-npc
feature/inventario
feature/terreno-mapa
feature/hud-ui
```

### Crear tu rama de trabajo

```bash
git checkout main
git pull origin main
git checkout -b feature/lobos-npc
```

### Mantener tu rama al día con main (hacerlo seguido, no solo al final)

```bash
git checkout main
git pull origin main
git checkout feature/lobos-npc
git merge main
```

Si hay conflictos aquí, los resuelves en tu rama, con calma, antes de que lleguen a `main`.

---

## 4. Flujo diario de trabajo (cada uno de los 4)

```bash
# 1. Empezar el día
git checkout feature/lobos-npc
git pull origin feature/lobos-npc      # si trabajas la misma rama desde 2 máquinas

# 2. Trabajar en Unity dentro de TU escena WIP y TU carpeta de _Project/

# 3. ANTES de hacer add, revisar qué se va a subir
git status

# 4. Si todo se ve bien (solo tus archivos en _Project/, nada de Library/Temp/paquetes)
git add Assets/_Project/Characters/Wolves/
git add Assets/_Project/Scenes/WIP_Lobos.unity

# 5. Revisar una vez más antes de confirmar
git status

# 6. Commit con mensaje claro
git commit -m "feat(lobos): agregar modelo y patrulla básica del lobo"

# 7. Subir tu rama (NO a main directamente)
git push origin feature/lobos-npc
```

### Commits pequeños y frecuentes

No esperes a terminar todo el feature para hacer un commit. Mejor:
```bash
git commit -m "feat(lobos): importar modelo y texturas"
git commit -m "feat(lobos): agregar animator y animaciones idle/walk"
git commit -m "feat(lobos): comportamiento de patrulla con NavMesh"
git commit -m "feat(lobos): convertir a prefab listo para integrar"
```
Así, si algo se rompe, es fácil identificar en qué commit ocurrió.

---

## 5. Revisar qué se sube (checklist antes de cada push)

```bash
git status
git status --ignored
```

Antes de cualquier `git push`, confirma:

- [ ] No aparece nada de `Library/`, `Temp/`, `Logs/`, `UserSettings/`, `.vs/`
- [ ] No aparece nada de `AllSkyFree/`, `AwesomeFreeScans/`, `Silver_Cats/`, `IgniteCoders/`, `ModularFirstPersonController/`
- [ ] Solo aparecen archivos dentro de tu carpeta de trabajo en `_Project/`
- [ ] Si agregaste texturas o modelos nuevos, no son excesivamente pesados (ideal: texturas a 1K-2K, no 4K, salvo que sea necesario)
- [ ] Los `.meta` correspondientes a tus archivos nuevos están incluidos (Git los detecta automático junto al archivo)

---

## 6. Proceso de integración (Pull Request → main)

Esto lo haces **tú**, como integrador, o cualquiera pero siguiendo este orden:

### Paso 1: Crear el Pull Request en GitHub
Desde la rama `feature/lobos-npc` hacia `main`, en GitHub web: "Compare & pull request".

### Paso 2: Revisión local antes de aprobar
```bash
git checkout main
git pull origin main
git checkout -b test-merge-lobos
git merge feature/lobos-npc
```
Abre Unity con esta rama de prueba:
- ¿Abre sin errores en consola?
- ¿El prefab del lobo se ve y funciona bien si lo arrastras a una escena de prueba?
- ¿No rompió nada de lo que ya existía?

### Paso 3: Integrar el prefab a la escena final (`_Main.unity`)
Esto es trabajo manual en el Editor de Unity, lo hace quien integra:
1. Abrir `_Project/Scenes/_Main.unity`
2. Arrastrar el prefab (`Lobo_NPC.prefab`) desde `_Project/Characters/Wolves/Prefabs/` a la escena, en la posición que corresponda
3. Ajustar posición, escala, configuración específica de esa escena (spawn points, etc.)
4. Probar en Play Mode que todo funciona junto con lo demás (jugador, terreno, otros NPCs)
5. Guardar la escena: `Ctrl+S`

### Paso 4: Commit de la integración
```bash
git add Assets/_Project/Scenes/_Main.unity
git commit -m "integrate: agregar lobo NPC a escena principal"
```

### Paso 5: Mergear a main
```bash
git checkout main
git merge feature/lobos-npc
git merge test-merge-lobos      # si hiciste la integración en esa rama temporal
git push origin main
```

### Paso 6: Limpieza
```bash
git branch -d feature/lobos-npc
git branch -d test-merge-lobos
git push origin --delete feature/lobos-npc   # opcional, borrar también en remoto
```

### Paso 7: Avisar al equipo
Mensaje al grupo: *"Lobos integrados a main, hagan `git pull` antes de seguir trabajando."*

---

## 7. Assets de terceros — no se suben, se documentan

Crear este archivo en la raíz del repo:

**`ASSETS_EXTERNOS.md`**
```markdown
# Assets de terceros (no incluidos en el repositorio)

Estos paquetes deben descargarse e importarse manualmente en Assets/
antes de abrir el proyecto. No están en Git por su tamaño.

| Paquete | Fuente | Carpeta destino |
|---|---|---|
| AllSky Free | [link Asset Store] | Assets/AllSkyFree/ |
| AwesomeFreeScans WoodLogs | [link] | Assets/AwesomeFreeScans/ |
| Silver_Cats Hand Painted Nature Kit | [link] | Assets/Silver_Cats/ |
| Simple Water Shader (IgniteCoders) | [link] | Assets/IgniteCoders/ |
| Modular First Person Controller | [link] | Assets/ModularFirstPersonController/ |
```

Comandos para dejar de versionar estos paquetes (si ya estaban subidos):

```bash
git rm -r --cached "Assets/AllSkyFree"
git rm -r --cached "Assets/AwesomeFreeScans"
git rm -r --cached "Assets/Silver_Cats"
git rm -r --cached "Assets/IgniteCoders"
git rm -r --cached "Assets/ModularFirstPersonController"

git add .gitignore ASSETS_EXTERNOS.md
git commit -m "chore: dejar de versionar paquetes de terceros, documentar en ASSETS_EXTERNOS.md"
git push origin main
```

> `--cached` borra del control de versiones pero NO borra los archivos de tu disco. Unity sigue viéndolos normal.

---

## 8. Si algo se rompe (situaciones comunes)

### "Hice merge y la escena quedó rota / objetos faltantes"
```bash
git merge --abort        # si el merge sigue en curso y no se ha confirmado
```
Si ya se confirmó el commit de merge:
```bash
git reset --hard HEAD~1  # SOLO si no has hecho push todavía
```
Si ya hiciste push y rompiste `main`, avisa al equipo de inmediato, no sigan trabajando sobre eso hasta revertir:
```bash
git revert -m 1 <hash-del-commit-de-merge>
git push origin main
```

### "Subí sin querer la carpeta Library o un paquete de terceros"
```bash
git rm -r --cached Assets/NombreCarpeta
git commit -m "fix: quitar carpeta subida por error"
git push
```

### "Tengo conflictos en un .unity o .prefab y no entiendo el diff"
Casi siempre es más rápido decidir manualmente qué versión conservar y rehacer el cambio chico a mano en el Editor, que tratar de resolver el YAML línea por línea:
```bash
git checkout --ours Assets/_Project/Scenes/_Main.unity   # me quedo con mi versión
# o
git checkout --theirs Assets/_Project/Scenes/_Main.unity # me quedo con la del otro
git add Assets/_Project/Scenes/_Main.unity
git commit
```
Después abres Unity y aplicas manualmente el cambio que faltó del lado que descartaste.

---

## 9. Resumen rápido (cheat sheet)

```bash
# Empezar feature nuevo
git checkout main && git pull && git checkout -b feature/mi-feature

# Actualizar mi rama con lo nuevo de main
git checkout main && git pull && git checkout feature/mi-feature && git merge main

# Guardar mi trabajo
git status
git add Assets/_Project/...
git status
git commit -m "feat(area): descripción corta"
git push origin feature/mi-feature

# Integrar a main (lo hace el integrador)
git checkout main && git pull
git checkout -b test-merge-x && git merge feature/mi-feature
# probar en Unity...
git checkout main && git merge feature/mi-feature && git push origin main
git branch -d feature/mi-feature test-merge-x
```

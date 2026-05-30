# Guía de Contribución y Reglas del Proyecto (23F)

¡Bienvenidos al proyecto! El objetivo de este trabajo es realizar un análisis lo más completo posible del dataset propuesto `Caso 23F`. Construiremos 4 casos de uso o iniciativas. 

Para no romper el código de los demás y cumplir con los requisitos técnicos, todos debemos seguir estas reglas **estrictas**:

## 1. Reglas de Git (Flujo de trabajo)
* **NUNCA** trabajes directamente en `main` ni en `develop`.
* Para empezar tu caso de uso, crea una rama desde `develop`: `git checkout -b feature/nombre-de-tu-caso`.
* Cuando termines o quieras revisión, sube tu rama (`git push origin feature/tu-rama`) y avisa al administrador del proyecto para hacer la integración a `develop`.

## 2. Reglas de Código y Rutas
* **Rutas relativas:** Todas las rutas utilizadas en el código deben ser rutas relativas. (Ej. `../data/raw/txt_files/`). ¡Nadie debe usar `C:/Users/...`!
* **Modularidad:** El notebook debe poder ejecutarse de principio a fin sin intervención manual. Si creas funciones largas de limpieza o modelos, guárdalas en archivos `.py` dentro de la carpeta `src/`.

## 3. Estructura de un Caso de Uso
Cada iniciativa debe tener entidad propia y presentar al menos:
1. Planteamiento inicial
2. Desarrollo (Exploración, Feature Engineering, Modelado)
3. Interpretación de resultados y métricas 
4. Conclusiones y próximos pasos

## 📝 Tareas Pendientes (¡Elige la tuya!)
Mientras el Caso de Uso 1 (Grafos de Comunicación) está en desarrollo, necesitamos que os asignéis estas tareas y creéis vuestras ramas:

* **Caso de Uso 2 (Estrés Sintáctico y Clustering):** Usar SpaCy para analizar la complejidad de las oraciones y agrupar (clustering) los documentos según el estrés de redacción bajo la presión del golpe.
* **Caso de Uso 3 (Sistema de Recomendación Documental):** Desarrollar un sistema de recomendación basado en Machine Learning que permita, dado un documento, sugerir otros documentos relevantes que ayuden a:
profundizar en el mismo tema (similitud) ampliar el contexto mediante perspectivas complementarias (diversidad)

* **Caso de Uso 4 (Detección de Anomalías Temporales):** Análisis de series temporales sobre el flujo de documentos para encontrar silencios informativos o picos de pánico.
* **Caso de Uso 5 (Recomendación Documental):** desarrollar un sistema de recomendación basado en Machine Learning que permita, dado un documento, sugerir otros documentos.

## 🔄 Cómo mantener tu rama actualizada con `develop` / `master`

A medida que avanza el proyecto, iremos subiendo scripts útiles (como `text_parser.py` o herramientas de scraping) a las ramas principales (`develop` o `master`). 

Para poder usar estas nuevas herramientas en tu propia rama **sin perder tu trabajo**, necesitas actualizar tu rama local. Sigue estos 4 sencillos pasos:

### Paso 1: Guarda tu trabajo actual
Antes de traer código de otros, asegúrate de que tu rama está "limpia" (sin archivos modificados sueltos).
```bash
git status
git add .
git commit -m "Guardo mi progreso antes de actualizar la rama"
```

### Paso 2: Descarga las novedades del servidor
Esto actualiza tu rama local con todo lo que ha pasado en el repositorio en la nube (GitHub, GitLab, etc.).
```bash
git fetch origin
```

### Paso 3: Actualiza tu versión local de la rama principal
Muévete a la rama principal y descárgate los últimos cambios.
```bash
git checkout main
git pull origin main
```

### Paso 4: Fusiona las novedades en TU rama
Vuelve a tu rama de trabajo y tráete todo lo nuevo que acaba de llegar a main.
```bash
# Cambia "mi-rama" por el nombre real de tu rama
git checkout mi-rama  
git merge main
```

⚠️ ¡Ayuda, tengo un conflicto de merge!
A veces, Git te avisará de que hay un "Conflicto" en el Paso 4. Esto es normal y solo significa que tú y otro compañero habéis modificado el mismo archivo en la misma línea.

Abre tu editor de código (ej. VS Code). Verás el archivo en rojo.

El editor te mostrará tu código y el código que viene de develop. Haz clic en "Aceptar cambios entrantes", "Aceptar ambos", o edítalo manualmente.

Guarda el archivo.

Dile a Git que el conflicto está resuelto:
```bash
# Cambia "mi-rama" por el nombre real de tu rama
git add nombre_del_archivo_resuelto.py
git commit -m "Resuelvo conflicto tras actualizar con main"
```

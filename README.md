# 281_Sistemas-Operativos-1
Contenido, ejemplos y recursos del curso de Sistemas Operativos 1.

## 📌 Guía de Trabajo para Tutores Auxiliares

¡Bienvenido/a al equipo de tutores! Para garantizar el correcto orden del material, este repositorio utiliza restricciones por directorio mediante el archivo `CODEOWNERS` y permisos asignados al **Team de Tutores**.

---

### 🚨 Políticas de Permisos y Edición

1. **Pertenencia al Team:** Eres parte del equipo asignado a este repositorio con permisos para subir cambios a la carpeta del ciclo vigente ubicada en  la rama `main`.
2. **Restricción de Rutas:** El archivo `.github/CODEOWNERS` protege los ciclos anteriores y otras carpetas del curso. **Únicamente se te permitirá hacer push o cambios sobre la carpeta correspondiente al ciclo actual.**
3. **Descarga Selectiva:** Para evitar descargar carpetas pesadas de ciclos pasados, es **obligatorio** utilizar el flujo de *sparse-checkout* detallado a continuación.

---

## 🚀 Flujo de Trabajo Paso a Paso

1. Sigue esta secuencia exacta de comandos en tu terminal para descargar exclusivamente la carpeta de trabajo asignada:

    ```bash
    # 1. Clonar el repositorio sin descargar archivos completos
      git clone --no-checkout [https://github.com/CococysLabs/281_Sistemas-Operativos-1.git](https://github.com/CococysLabs/281_Sistemas-Operativos-1.git)

    cd 281_Sistemas-Operativos-1

    # 2. Habilitar sparse-checkout en modo cono
    git sparse-checkout init --cone

    # 3. Indicar únicamente la carpeta que necesita trabajar el tutor
    git sparse-checkout set [CARPETA-CICLO-ACTUAL]/Ejemplos

    # 4. Descargar solo esa carpeta en la rama main
    git checkout main
    ```

2. Agrega tus códigos de ejemplo, guías o material didáctico dentro de la carpeta descargada:

    `[CARPETA-CICLO-ACTUAL]/Ejemplos/`

3. Guarda tus cambios localmente creando un commit explicativo:

    ```bash
    git add .
    git commit -m "feat: agregar ejemplo de [DESCRIPCION] para el ciclo [CICLO-ACTUAL]"
    ```

4. Envía tus cambios directamente a la rama principal:

    ```bash
    git push origin main
    ```

    > **Nota:** Si por error intentas modificar o eliminar archivos fuera de la carpeta `[CARPETA-CICLO-ACTUAL]/Ejemplos`, la plataforma rechazará el `push` debido a las reglas de propiedad configuradas en `CODEOWNERS`.

---

## 📁 Estructura del Repositorio

```text
281_Sistemas-Operativos-1/
├── .github/
│   └── CODEOWNERS                       <-- Configuración de permisos
├── Ciclo-2024-Segundo-Semestre/         <-- Protegido por CODEOWNERS
├── Ciclo-2025-Primer-Semestre/          <-- Protegido por CODEOWNERS
├── Ciclo-2025-Segundo-Semestre/         <-- Protegido por CODEOWNERS
├── Ciclo-2026-Primer-Semestre/          <-- Protegido por CODEOWNERS
└── [CARPETA-CICLO-ACTUAL]/             
    └── Ejemplos/			                   <-- 🎯 Tu carpeta de trabajo asignada
        └── .gitkeep

```

## 📧 Contacto

- Email: computacion.cococys@gmail.com
- Organización: [CococysLabs](https://github.com/CococysLabs)

---



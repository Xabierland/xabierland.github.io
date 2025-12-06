# Guía de Estilo y Estructura del Blog

Este documento describe la estructura estándar y las pautas de escritura para los posts del blog de xabierland.github.io.

## Estructura de Front Matter

Todos los posts deben incluir el siguiente front matter YAML al inicio del archivo:

```yaml
---
title: Título descriptivo del post
author: Xabierland
description: >-
  Descripción breve y clara del contenido del post.
  Puede ocupar múltiples líneas si se usa el formato >-
date: YYYY-MM-DD HH:MM
categories: [Categoría Principal, Subcategoría]
tags: [Tag1, Tag2, Tag3, Tag4]
---
```

### Categorías comunes:
- `[Blogs]` - Posts generales, opiniones, tutoriales diversos
- `[HomeLab]` - Proyectos de servidor casero, self-hosting
- `[Administración de sistemas, Orquestación de contenedores]` - Kubernetes, Docker, Podman, etc.

### Tags comunes:
- Tecnologías específicas: `Kubernetes`, `Docker`, `Proxmox`, `K3s`, `Containers`
- Temas: `VPN`, `Redes`, `Seguridad`, `Privacidad`, `Virtualización`
- Herramientas: `Tailscale`, `Monero`, `Stremio`

## Estructura del Contenido

### 1. Introducción (obligatoria)
```markdown
## Introducción

Párrafo que contextualiza el tema, explica por qué es relevante y
presenta lo que el lector aprenderá o encontrará en el post.
```

### 2. Sección de Contexto Teórico (opcional pero recomendada)
```markdown
### ¿Qué es X?

Explicación del concepto principal.

### ¿Por qué X?

Ventajas y casos de uso. Puede incluir:
- Lista de ventajas con viñetas
- Comparación con alternativas
- Limitaciones o desventajas (si aplica)
```

### 3. Guía Práctica / Tutorial (cuerpo principal)
```markdown
## Instalación y Configuración

### 1. Paso uno
### 2. Paso dos
### 3. Paso tres
```

Usar subsecciones numeradas para pasos secuenciales.

### 4. Conclusión (opcional)
```markdown
## Conclusión

Resumen de lo aprendido, próximos pasos, o reflexiones finales.
```

### 5. Referencias / Bibliografía (opcional)
```markdown
## Referencias

- [Nombre del recurso](URL)
- [Documentación oficial](URL)

[^1]: Nota al pie si se usaron footnotes en el texto
```

## Elementos de Formato

### Bloques de código
Siempre especificar el lenguaje para el syntax highlighting:

```markdown
```bash
comando aquí
```

```yaml
clave: valor
```

```text
salida de texto plano
```
```

### Notas especiales (Prompts de Jekyll)
```markdown
> Mensaje de advertencia importante sobre configuración o seguridad
{: .prompt-warning }

> Consejo útil o recomendación
{: .prompt-tip }
```

### Tablas
```markdown
| Columna 1   | Columna 2   | Columna 3     |
| ----------- | ----------- | ------------- |
| Dato        | Dato        | Dato          |
| Otro dato   | Otro dato   | Otro dato     |
```

### Imágenes
Las imágenes deben guardarse en `/assets/img/posts/` y referenciarse así:

```markdown
![Descripción de la imagen](/assets/img/posts/nombre-archivo.png)
```

### Footnotes
```markdown
Texto con referencia a nota al pie[^1].

[^1]: Explicación detallada de la referencia
```

### Listas
```markdown
- Viñeta nivel 1
  - Viñeta nivel 2 (2 espacios de indentación)

1. Lista numerada
2. Segundo elemento
```

### Comentarios HTML
Para deshabilitar reglas de markdown-lint cuando sea necesario:
```markdown
<!-- markdownlint-disable MD033 -->
```

## Convenciones de Nomenclatura

### Archivos de posts
Formato: `YYYY-MM-DD-Titulo-Del-Post.md`

Ejemplos:
- `2024-11-04-Instalar-y-trabajar-con-K3s.md`
- `2025-05-01-Tailscale.md`

### Imágenes
Usar nombres descriptivos en minúsculas con guiones:
- `k3s-ha-cluster.png`
- `tailscale-subnet.png`

## Estilo de Escritura

1. **Tono**: Informativo, directo y técnico pero accesible
2. **Persona**: Segunda persona del singular (tú) para dirigirse al lector
3. **Formato de comandos**: Siempre explicar qué hace un comando antes o después de mostrarlo
4. **Seguridad**: Incluir advertencias cuando un paso pueda afectar la seguridad del sistema
5. **Contexto**: Proporcionar suficiente contexto teórico antes de la implementación práctica

## Plantilla Rápida

Ver archivo `post-template.md` en este mismo directorio para una plantilla lista para copiar.

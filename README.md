# Práctica: Express + Handlebars - Blog Personal

Vamos a construir un blog personal usando Express y Handlebars, con énfasis en iteraciones (`{{#each}}`), condicionales y datos dinámicos.

## Primeros Pasos

### 1. Fork & Clone

1. Haz **Fork** de este repositorio
2. Clona tu fork y navega a la carpeta del proyecto

### 2. Configuración del Proyecto

**Nota:** Este proyecto NO tiene `package.json` pre-configurado. Tú crearás todo desde cero siguiendo las iteraciones a continuación.

## Estructura Final del Proyecto

Toma nota que algunos archivos ya han sido creados y otros tienen que ser creados por ti.

```bash
express-handlebars/
├── app.js                      # Archivo principal del servidor
├── blog-data.json              # Datos del blog (proporcionado)
├── sobre-mi.json               # Datos personales (proporcionado)
├── views/
│   ├── layouts/
│   │   └── main.handlebars     # Layout principal
│   ├── home.handlebars         # Página principal con lista de posts
│   ├── post.handlebars         # Detalle de un post individual
│   └── about.handlebars        # Página sobre mí
├── public/
│   └── styles.css              # Estilos CSS (tú añades)
├── .env                        # Variables de entorno
└── package.json                # Configuración npm
```

## Tu Tarea

Vamos a construir un blog con las siguientes funcionalidades:

- Lista de posts
- Vista detallada de cada post
- Página "Sobre mí" con listas dinámicas
- Navegación entre páginas
- Estilos CSS personalizados

---

## Iteración 0: Configuración Inicial

### Paso 1: Inicializar el proyecto

```bash
npm init -y
npm pkg set type=module
npm pkg set scripts.start="node --watch app.js"
```

### Paso 2: Instalar dependencias

```bash
npm install express express-handlebars dotenv
```

Tu `package.json` debería verse así:
```json
{
  "name": "express-handlebars",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node --watch app.js"
  },
  "dependencies": {
    "dotenv": "^16.0.0",
    "express": "^5.1.0",
    "express-handlebars": "^8.0.0"
  }
}
```

### Paso 3: Crear archivo `.env`

```bash
PORT=3000
```

### Paso 4: Crear `app.js` básico

```js
// app.js

import 'dotenv/config';
import express from 'express';
import { engine } from 'express-handlebars';
import { readFileSync } from 'fs';

const app = express();
const PORT = process.env.PORT || 3000;

// Leer datos de los archivos JSON
const blogData = JSON.parse(readFileSync('./blog-data.json', 'utf-8'));
const aboutData = JSON.parse(readFileSync('./sobre-mi.json', 'utf-8'));

// Configurar Handlebars
app.engine('handlebars', engine());
app.set('view engine', 'handlebars');
app.set('views', './views');

// Servir archivos estáticos
app.use(express.static('public'));

// Aquí irán tus rutas...

app.listen(PORT, () => {
  console.log(`Servidor corriendo en http://localhost:${PORT}`);
});
```

---

## Iteración 1: Crear el Layout Principal

Revisaremos el archivo `views/layouts/main.handlebars`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{pageTitle}} - DevWeb Blog</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <header>
    <nav>
      <h2>DevWeb Blog</h2>
      <ul>
        <li><a href="/">Inicio</a></li>
        <li><a href="/about">Sobre Mí</a></li>
      </ul>
    </nav>
  </header>

  <main>
    {{{body}}}
  </main>

  <footer>
    <p>&copy; 2025 DevWeb Blog. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
```

Observa que usamos `{{pageTitle}}` - cada página podrá pasar su propio título.

---

## Iteración 2: Página de Inicio con Lista de Posts

### Paso 1: Crear la ruta

En `app.js`, añade la ruta para la página principal:

```js
app.get('/', (req, res) => {
  res.render('home', {
    pageTitle: 'Inicio',
    blogTitle: blogData.blogTitle,
    blogDescription: blogData.blogDescription,
    posts: blogData.posts
  });
});
```

### Paso 2: Crear la vista `views/home.handlebars`

Tu tarea es crear esta vista que debe:
- Mostrar el título y descripción del blog
- Iterar sobre todos los posts usando `{{#each posts}}`
- Para cada post, mostrar:
  - Título
  - Excerpt (resumen)
  - Autor
  - Fecha
  - Categoría
  - Número de comentarios
  - Enlace para leer más (usando el id del post)

---
<details>
<summary>Ayuda:</summary>

```handlebars
<h1>{{blogTitle}}</h1>
<p>{{blogDescription}}</p>

<div class="posts-grid">
  {{#each posts}}
    <article class="post-card">
      <!-- Aquí va el contenido de cada post -->
      <h2>{{this.title}}</h2>
      <!-- Completa el resto... -->
      <a href="/post/{{this.id}}">Leer más →</a>
    </article>
  {{else}}
    <p>No hay posts disponibles.</p>
  {{/each}}
</div>
```
</details>

---

### Paso 3: Prueba

Inicia el servidor con `npm start` y visita `http://localhost:3000`

Deberías ver una lista de 7 posts.

---

## Iteración 3: Vista de Detalle de Post (con Route Params)

### Paso 1: Crear la ruta con parámetro dinámico

En `app.js`:

```js
app.get('/post/:id', (req, res) => {
  const postId = parseInt(req.params.id);
  const post = blogData.posts.find(p => p.id === postId);

  if (!post) {
    return res.status(404).send('Post no encontrado');
  }

  res.render('post', {
    pageTitle: post.title,
    post: post
  });
});
```

### Paso 2: Crear `views/post.handlebars`

Tu tarea es crear la vista de detalle que muestre:
- Imagen del post (si existe)
- Título
- Información del autor y fecha
- Categoría
- Contenido completo
- Tags (usando `{{#each post.tags}}`)
- Número de comentarios
- Enlace para volver a la lista


<details>
<summary>Ayuda para los Tags</summary>

```handlebars
<div class="tags">
  {{#each post.tags}}
    <span class="tag">{{this}}</span>
  {{/each}}
</div>
```

</details>


### Paso 3: Prueba

Visita `http://localhost:3000/post/1` y deberías ver el detalle del primer post.

Prueba con diferentes IDs: `/post/2`, `/post/3`, etc.

---

## Iteración 4: Página "Sobre Mí"

### Paso 1: Crear la ruta

```js
app.get('/about', (req, res) => {
  res.render('about', {
    pageTitle: 'Sobre Mí',
    ...aboutData  // Spread operator para pasar todos los datos
  });
});
```

### Paso 2: Crear `views/about.handlebars`

Tu tarea es crear una página que muestre:
- Nombre
- Imagen de perfil
- Bio (biografía)
- Ocupación
- Ubicación
- Email
- **Redes sociales** (usando `{{#each socialMedia}}`)
  - Para cada red: nombre de la plataforma y enlace
- **Intereses** (usando `{{#each interests}}`)
  - Mostrar cada interés como una lista

**Pista para redes sociales:**
```handlebars
<h2>Redes Sociales</h2>
<ul>
  {{#each socialMedia}}
    <li>
      <a href="{{this.url}}" target="_blank">{{this.platform}}</a>
    </li>
  {{/each}}
</ul>
```

**Pista para intereses:**
```handlebars
<h2>Intereses</h2>
<ul>
  {{#each interests}}
    <li>{{this}}</li>
  {{/each}}
</ul>
```

### Paso 3: Prueba

Visita `http://localhost:3000/about`

---

## Iteración 5: Añadir Estilos CSS

Ahora que toda la funcionalidad está lista, dale estilo a tu blog.

Abre `public/styles.css` y añade estilos. Aquí algunas sugerencias:

**Estructura mínima:**
```css
/* Reset básico */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  line-height: 1.6;
  color: #333;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* Header y navegación */
header nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0;
  border-bottom: 2px solid #eee;
  margin-bottom: 2rem;
}

header nav ul {
  display: flex;
  list-style: none;
  gap: 2rem;
}

header nav a {
  text-decoration: none;
  color: #333;
  font-weight: 500;
}

/* Grid de posts */
.posts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
  margin: 2rem 0;
}

/* Tarjeta de post */
.post-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1.5rem;
  transition: transform 0.2s;
}

.post-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

/* Detalle de post */
.post-detail {
  max-width: 800px;
  margin: 0 auto;
}

.post-detail img {
  width: 100%;
  border-radius: 8px;
  margin-bottom: 1rem;
}

/* Tags */
.tags {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  margin: 1rem 0;
}

.tag {
  background: #f0f0f0;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.9rem;
}

/* Footer */
footer {
  margin-top: 4rem;
  padding-top: 2rem;
  border-top: 2px solid #eee;
  text-align: center;
  color: #666;
}
```

**Personaliza:**
- Colores
- Tipografía
- Espaciado
- Efectos hover
- Responsive design (media queries)

---

## Retos Adicionales (Bonus)

### Reto 1: Filtrar posts por categoría

Crea una ruta `/category/:categoryName` que muestre solo los posts de esa categoría.


<details>
<summary>Solución</summary>

```js
app.get('/category/:categoryName', (req, res) => {
  const category = req.params.categoryName;
  const filteredPosts = blogData.posts.filter(p =>
    p.category.toLowerCase() === category.toLowerCase()
  );

  res.render('home', {
    pageTitle: `Categoría: ${category}`,
    blogTitle: `Posts de ${category}`,
    blogDescription: `Todos los artículos sobre ${category}`,
    posts: filteredPosts
  });
});
```

</details>

### Reto 2: Mostrar posts relacionados

En la vista de detalle de un post, muestra 3 posts de la misma categoría (excluyendo el actual).

<details>
<summary>Ayuda</summary>

```js
const relatedPosts = blogData.posts
  .filter(p => p.category === post.category && p.id !== post.id)
  .slice(0, 3);

res.render('post', {
  pageTitle: post.title,
  post: post,
  relatedPosts: relatedPosts
});
```

En la vista:
```handlebars
{{#if relatedPosts.length}}
  <h3>Posts Relacionados</h3>
  <div class="related-posts">
    {{#each relatedPosts}}
      <article class="post-card">
        <h3>{{this.title}}</h3>
        <p>{{this.excerpt}}</p>
        <a href="/post/{{this.id}}">Leer más →</a>
      </article>
    {{/each}}
  </div>
{{/if}}
```

</details>

### Reto 3: Añadir navegación de categorías

Extrae todas las categorías únicas de los posts y muéstralas en un menú lateral de navegación.

### Reto 4: Página 404 personalizada

Crea una vista `404.handlebars` y una ruta catch-all:

```js
app.use((req, res) => {
  res.status(404).render('404', {
    pageTitle: 'Página no encontrada'
  });
});
```

## FAQ

<details>
<summary>1. ¿Por qué usar readFileSync en lugar de readFile?</summary>

<br>

`readFileSync` es **síncrono** - bloquea la ejecución hasta que termina:
```js
const data = readFileSync('./data.json', 'utf-8'); // Espera aquí
console.log(data); // Se ejecuta después
```

`readFile` es **asíncrono** - no bloquea:
```js
readFile('./data.json', 'utf-8', (err, data) => {
  console.log(data); // Se ejecuta cuando termine
});
console.log('Esto se ejecuta primero'); // No espera
```

**¿Cuándo usar cada uno?**
- `readFileSync` - Al inicio de la aplicación para cargar configuración (como hacemos aquí)
- `readFile` - Durante la ejecución cuando no quieres bloquear (ej. dentro de rutas)

Para este ejercicio, `readFileSync` está bien porque solo leemos los datos una vez al iniciar el servidor.

</details>

<details>
<summary>2. ¿Qué es el spread operator (...) en JavaScript?</summary>

<br>

El spread operator (`...`) "expande" un objeto o array:

```js
const aboutData = {
  name: 'María',
  email: 'maria@example.com',
  bio: 'Desarrolladora web'
};

// Sin spread
res.render('about', {
  pageTitle: 'Sobre Mí',
  name: aboutData.name,
  email: aboutData.email,
  bio: aboutData.bio
});

// Con spread
res.render('about', {
  pageTitle: 'Sobre Mí',
  ...aboutData  // Expande todas las propiedades automáticamente
});
```

Ambos hacen lo mismo, pero el spread es más corto y escalable.

</details>

<details>
<summary>3. ¿Por qué usar parseInt con req.params.id?</summary>

<br>

Los route params siempre son **strings**:

```js
// URL: /post/5
console.log(req.params.id); // "5" (string)
console.log(typeof req.params.id); // "string"
```

En nuestro JSON, los IDs son **números**:
```json
{ "id": 1, "title": "..." }
```

Si comparamos string con número:
```js
"5" === 5  // false ❌
```

Por eso convertimos a número:
```js
const postId = parseInt(req.params.id); // 5 (number)
postId === 5  // true ✅
```

</details>

<details>
<summary>4. ¿Cómo funciona el método .find()?</summary>

<br>

`.find()` busca el **primer elemento** que cumple una condición:

```js
const posts = [
  { id: 1, title: 'Post 1' },
  { id: 2, title: 'Post 2' },
  { id: 3, title: 'Post 3' }
];

const post = posts.find(p => p.id === 2);
console.log(post); // { id: 2, title: 'Post 2' }
```

**Cómo funciona:**
1. Recorre cada elemento del array
2. Ejecuta la función de prueba (`p => p.id === 2`)
3. Retorna el primer elemento donde la función devuelve `true`
4. Si no encuentra nada, retorna `undefined`

**Parámetros:**
- `p` - Cada elemento del array mientras recorre
- `p => p.id === 2` - Función que retorna true/false

</details>

<details>
<summary>5. ¿Cómo depurar si los datos no aparecen en la vista?</summary>

<br>

**Paso 1: Verifica que los datos llegan a la ruta**
```js
app.get('/', (req, res) => {
  console.log('Posts:', blogData.posts); // Ver en la TERMINAL
  res.render('home', {
    posts: blogData.posts
  });
});
```

**Paso 2: Verifica en la vista Handlebars**
```handlebars
<!-- Ver todo el objeto posts -->
<pre>{{json posts}}</pre>

<!-- Contar elementos -->
<p>Total posts: {{posts.length}}</p>

<!-- Ver si existe -->
{{#if posts}}
  <p>Los posts existen</p>
{{else}}
  <p>No hay posts</p>
{{/if}}
```

**Paso 3: Verifica la sintaxis de Handlebars**
```handlebars
<!-- ❌ Incorrecto -->
{{each posts}}
  {{post.title}}
{{/each}}

<!-- ✅ Correcto -->
{{#each posts}}
  {{this.title}}
{{/each}}
```

</details>

---

## Recursos Adicionales

- [Express Handlebars - npm](https://www.npmjs.com/package/express-handlebars)
- [Handlebars - Sintaxis](https://handlebarsjs.com/guide/)
- [Express - Routing Guide](https://expressjs.com/en/guide/routing.html)
- [MDN - Array.prototype.find()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)

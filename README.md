# Guía SEO Completa para Next.js (App Router) — 2026

> Guía general y aplicable a cualquier proyecto Next.js con App Router.
> Basada en documentación oficial de Next.js, Vercel y las mejores prácticas SEO actuales.

---

## Índice

1. [Por qué Next.js para SEO](#1-por-qué-nextjs-para-seo)
2. [Estructura de proyecto recomendada](#2-estructura-de-proyecto-recomendada)
3. [Estrategias de renderizado](#3-estrategias-de-renderizado)
4. [Metadata API](#4-metadata-api)
5. [Open Graph y Twitter Cards](#5-open-graph-y-twitter-cards)
6. [Sitemap dinámico](#6-sitemap-dinámico)
7. [Robots.txt](#7-robotstxt)
8. [JSON-LD / Structured Data](#8-json-ld--structured-data)
9. [Canonical URLs](#9-canonical-urls)
10. [Optimización de imágenes (next/image)](#10-optimización-de-imágenes-nextimage)
11. [Optimización de fuentes (next/font)](#11-optimización-de-fuentes-nextfont)
12. [Core Web Vitals](#12-core-web-vitals)
13. [Internacionalización (i18n)](#13-internacionalización-i18n)
14. [Generative Engine Optimization (GEO)](#14-generative-engine-optimization-geo)
15. [Anatomía de un resultado de búsqueda (SERP)](#15-anatomía-de-un-resultado-de-búsqueda-serp)
16. [Favicon e íconos](#16-favicon-e-íconos)
17. [Checklist SEO rápido](#17-checklist-seo-rápido)
18. [Herramientas de validación](#18-herramientas-de-validación)
19. [Fuentes](#19-fuentes)

---

## 1. Por qué Next.js para SEO

Next.js es el framework React más potente para SEO porque:

- **Server-Side Rendering (SSR)** y **Static Site Generation (SSG)**: los crawlers reciben HTML completo sin necesidad de ejecutar JavaScript.
- **Metadata API nativa**: títulos, descripciones y Open Graph se definen de forma declarativa.
- **Archivos SEO automáticos**: `sitemap.xml`, `robots.txt` y `manifest.json` se generan con funciones TypeScript.
- **Optimización de assets integrada**: imágenes (`next/image`) y fuentes (`next/font`) optimizados automáticamente.
- **Streaming y React Server Components**: HTML enviado al navegador de forma incremental, mejorando el Time to First Byte (TTFB).

> Los motores de búsqueda en 2026 no rankean frameworks — rankean **rendimiento, estructura y experiencia de usuario**.

---

## 2. Estructura de proyecto recomendada

```
proyecto/
├── public/                     # Assets estáticos (favicon, imágenes, og-image.png)
│   ├── favicon.ico
│   ├── og-image.png            # Imagen Open Graph (1200x630px)
│   └── img/
├── src/
│   ├── app/                    # App Router (rutas basadas en carpetas)
│   │   ├── layout.tsx          # Layout raíz (metadata global, fonts, <html lang>)
│   │   ├── page.tsx            # Página principal "/"
│   │   ├── not-found.tsx       # Página 404 personalizada
│   │   ├── globals.css         # Estilos globales
│   │   ├── sitemap.ts          # Genera /sitemap.xml
│   │   ├── robots.ts           # Genera /robots.txt
│   │   ├── manifest.ts         # Genera /manifest.webmanifest
│   │   ├── about/
│   │   │   └── page.tsx        # Ruta "/about"
│   │   └── blog/
│   │       ├── page.tsx        # Ruta "/blog"
│   │       └── [slug]/
│   │           └── page.tsx    # Rutas dinámicas "/blog/mi-articulo"
│   ├── components/
│   │   ├── ui/                 # Componentes de UI reutilizables
│   │   └── seo/
│   │       └── JsonLd.tsx      # Componente de structured data
│   └── lib/
│       └── utils.ts
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

### Reglas clave

- **`layout.tsx`** = Server Component por defecto → ideal para metadata.
- **`page.tsx`** = Server Component por defecto → ideal para SEO.
- Usa `"use client"` solo en componentes que necesiten interactividad (useState, useEffect, event handlers, framer-motion, etc.).
- Nunca pongas metadata en Client Components — siempre en `layout.tsx` o `page.tsx`.

---

## 3. Estrategias de renderizado

| Estrategia | Cuándo usarla | SEO |
|---|---|---|
| **SSG** (Static Site Generation) | Contenido que no cambia frecuentemente (landing, blog, docs) | Excelente |
| **ISR** (Incremental Static Regeneration) | Contenido que cambia periódicamente (productos, catálogos) | Excelente |
| **SSR** (Server-Side Rendering) | Contenido personalizado o en tiempo real (dashboards, búsquedas) | Muy bueno |
| **CSR** (Client-Side Rendering) | Secciones privadas detrás de auth (no necesitan SEO) | No aplica |

### Ejemplo: SSG (default en App Router)

```tsx
// src/app/page.tsx
// Por defecto, las páginas en App Router son estáticas (SSG)
export default function HomePage() {
  return <h1>Mi página</h1>;
}
```

### Ejemplo: ISR

```tsx
// src/app/blog/[slug]/page.tsx
export const revalidate = 3600; // Revalidar cada hora

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetchPost(params.slug);
  return <article>{post.content}</article>;
}
```

### Ejemplo: SSR

```tsx
// src/app/search/page.tsx
export const dynamic = "force-dynamic"; // Fuerza SSR en cada request

export default async function SearchPage({ searchParams }) {
  const results = await search(searchParams.q);
  return <div>{/* resultados */}</div>;
}
```

---

## 4. Metadata API

### Metadata estática

```tsx
// src/app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL("https://tusitio.com"),
  title: {
    default: "Mi Sitio — Descripción breve",
    template: "%s | Mi Sitio", // Para páginas hijas: "Blog | Mi Sitio"
  },
  description: "Descripción del sitio de 150-160 caracteres máximo.",
  keywords: ["palabra clave 1", "palabra clave 2", "palabra clave 3"],
  authors: [{ name: "Tu Nombre o Empresa" }],
  creator: "Tu Empresa",
  publisher: "Tu Empresa",
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-video-preview": -1,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },
};
```

### Metadata dinámica (para rutas dinámicas)

```tsx
// src/app/blog/[slug]/page.tsx
import type { Metadata } from "next";

export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await fetchPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: post.coverImage }],
    },
    alternates: {
      canonical: `https://tusitio.com/blog/${params.slug}`,
    },
  };
}
```

### Buenas prácticas de metadata

- **Título**: 50-60 caracteres. Incluye la keyword principal al inicio.
- **Descripción**: 150-160 caracteres. Incluye un CTA o propuesta de valor. Es lo que alimenta el **snippet** del resultado — ver sección 15.
- **`max-snippet: -1`**: quita el límite de caracteres del snippet. Con `0` Google no muestra snippet; con un número, lo trunca a esa longitud.
- **Keywords**: Usa 5-15 keywords relevantes (Google no las usa directamente, pero otros motores sí).
- Cada página debe tener título y descripción **únicos**.
- Usa `template` en el layout raíz para consistencia: `"%s | Mi Sitio"`.

---

## 5. Open Graph y Twitter Cards

```tsx
export const metadata: Metadata = {
  openGraph: {
    type: "website",
    locale: "es_ES",           // o "es_EC", "en_US", etc.
    url: "https://tusitio.com",
    siteName: "Mi Sitio",
    title: "Mi Sitio — Descripción breve",
    description: "Descripción para redes sociales.",
    images: [
      {
        url: "/og-image.png",  // 1200x630px recomendado
        width: 1200,
        height: 630,
        alt: "Descripción de la imagen",
      },
    ],
  },
  twitter: {
    card: "summary_large_image",
    title: "Mi Sitio — Descripción breve",
    description: "Descripción para Twitter.",
    images: ["/og-image.png"],
    // site: "@tuhandle",      // Opcional
    // creator: "@tuhandle",   // Opcional
  },
};
```

### Imagen OG dinámica (avanzado)

Next.js puede generar imágenes OG dinámicamente con `opengraph-image.tsx`:

```tsx
// src/app/opengraph-image.tsx
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const alt = "Mi Sitio";
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default function Image() {
  return new ImageResponse(
    (
      <div style={{ fontSize: 64, background: "#0a1628", color: "white", width: "100%", height: "100%", display: "flex", alignItems: "center", justifyContent: "center" }}>
        Mi Sitio
      </div>
    ),
    { ...size }
  );
}
```

---

## 6. Sitemap dinámico

```tsx
// src/app/sitemap.ts
import type { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = "https://tusitio.com";

  // Páginas estáticas
  const staticPages: MetadataRoute.Sitemap = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 1,
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.8,
    },
  ];

  // Páginas dinámicas (ej: blog posts desde una API/CMS)
  const posts = await fetchAllPosts();
  const blogPages: MetadataRoute.Sitemap = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    changeFrequency: "weekly" as const,
    priority: 0.6,
  }));

  return [...staticPages, ...blogPages];
}
```

Esto genera automáticamente `/sitemap.xml` — no necesitas paquetes externos.

---

## 7. Robots.txt

```tsx
// src/app/robots.ts
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: "/",
        disallow: ["/api/", "/admin/", "/_next/", "/private/"],
      },
      {
        userAgent: "Googlebot",
        allow: "/",
      },
    ],
    sitemap: "https://tusitio.com/sitemap.xml",
  };
}
```

### Buenas prácticas

- Siempre incluye la URL del sitemap.
- Bloquea rutas de API, admin y assets internos de Next.js.
- No bloquees archivos CSS/JS — los crawlers modernos los necesitan para renderizar.

---

## 8. JSON-LD / Structured Data

JSON-LD es el formato recomendado por Google para datos estructurados. Permite que tu contenido aparezca como **rich results** (estrellas, precios, FAQs desplegables, etc.).

### Implementación segura

```tsx
// src/components/seo/JsonLd.tsx

// IMPORTANTE: Siempre como Server Component (sin "use client")
// para que se incluya en el HTML inicial

interface JsonLdProps {
  data: Record<string, unknown>;
}

export function JsonLdScript({ data }: JsonLdProps) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        // Prevenir XSS: reemplazar < con unicode
        __html: JSON.stringify(data).replace(/</g, "\\u003c"),
      }}
    />
  );
}
```

### Schemas más útiles por tipo de sitio

#### Organization (cualquier sitio)

```tsx
const organizationSchema = {
  "@context": "https://schema.org",
  "@type": "Organization",
  name: "Mi Empresa",
  url: "https://tusitio.com",
  logo: "https://tusitio.com/logo.png",
  description: "Descripción de tu empresa.",
  contactPoint: {
    "@type": "ContactPoint",
    telephone: "+593-XXX-XXXX",
    contactType: "customer service",
  },
  sameAs: [
    "https://twitter.com/tuempresa",
    "https://linkedin.com/company/tuempresa",
  ],
};
```

#### WebSite (cualquier sitio)

```tsx
const websiteSchema = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  name: "Mi Sitio",
  url: "https://tusitio.com",
  description: "Descripción breve.",
  inLanguage: "es",
  // ⚠️ Obsoleto: Google retiró el rich result de Sitelinks Searchbox en nov 2024.
  // Sigue siendo válido como dato estructurado, pero ya no produce ningún
  // resultado enriquecido. Ver sección 15.
  potentialAction: {
    "@type": "SearchAction",
    target: "https://tusitio.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string",
  },
};
```

#### FAQPage (páginas con preguntas frecuentes)

```tsx
const faqSchema = {
  "@context": "https://schema.org",
  "@type": "FAQPage",
  mainEntity: [
    {
      "@type": "Question",
      name: "¿Pregunta 1?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "Respuesta 1.",
      },
    },
    // ... más preguntas
  ],
};
```

#### Product (e-commerce / SaaS con precios)

```tsx
const productSchema = {
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",  // o "Product"
  name: "Mi App",
  applicationCategory: "BusinessApplication",
  operatingSystem: "Web",
  offers: {
    "@type": "Offer",
    price: "29.99",
    priceCurrency: "USD",
  },
  aggregateRating: {
    "@type": "AggregateRating",
    ratingValue: "4.8",
    reviewCount: "150",
  },
};
```

#### Article / BlogPosting (blogs)

```tsx
const articleSchema = {
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  headline: "Título del artículo",
  description: "Descripción del artículo.",
  image: "https://tusitio.com/blog/imagen.jpg",
  datePublished: "2026-01-15",
  dateModified: "2026-03-10",
  author: {
    "@type": "Person",
    name: "Nombre del Autor",
  },
};
```

#### BreadcrumbList (navegación)

```tsx
const breadcrumbSchema = {
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  itemListElement: [
    { "@type": "ListItem", position: 1, name: "Inicio", item: "https://tusitio.com" },
    { "@type": "ListItem", position: 2, name: "Blog", item: "https://tusitio.com/blog" },
    { "@type": "ListItem", position: 3, name: "Mi Artículo" },
  ],
};
```

---

## 9. Canonical URLs

Las canonical URLs evitan contenido duplicado. Defínelas en cada página:

```tsx
export const metadata: Metadata = {
  alternates: {
    canonical: "https://tusitio.com/mi-pagina",
    // Para sitios multi-idioma:
    languages: {
      "es": "https://tusitio.com/es/mi-pagina",
      "en": "https://tusitio.com/en/my-page",
    },
  },
};
```

### Reglas

- Cada página debe tener **exactamente una** canonical URL.
- La canonical debe apuntar a la versión preferida (con o sin `www`, con o sin trailing slash).
- En páginas paginadas (`/blog?page=2`), la canonical debe apuntar a sí misma, no a la página 1.

---

## 10. Optimización de imágenes (next/image)

```tsx
import Image from "next/image";

// Imagen local (optimización automática en build)
<Image
  src="/hero.png"
  alt="Descripción detallada de la imagen"
  width={1200}
  height={630}
  priority          // Solo para imágenes above-the-fold (hero, logo)
/>

// Imagen remota
<Image
  src="https://cdn.ejemplo.com/foto.jpg"
  alt="Descripción detallada"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, 50vw"
/>
```

### Buenas prácticas

| Práctica | Por qué |
|---|---|
| Siempre incluir `alt` descriptivo | Accesibilidad + SEO de imágenes |
| Usar `priority` solo en LCP image | Mejora el Largest Contentful Paint |
| Definir `sizes` en imágenes responsive | Evita descargar imágenes innecesariamente grandes |
| Configurar `formats: ["image/avif", "image/webp"]` en `next.config.ts` | Formatos modernos = menor peso |
| Usar imágenes locales cuando sea posible | Optimización completa en build time |

### Configuración en next.config.ts

```ts
const nextConfig: NextConfig = {
  images: {
    formats: ["image/avif", "image/webp"],
    // Para imágenes de dominios externos:
    remotePatterns: [
      {
        protocol: "https",
        hostname: "cdn.ejemplo.com",
      },
    ],
  },
};
```

---

## 11. Optimización de fuentes (next/font)

```tsx
// src/app/layout.tsx
import { Inter } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  display: "swap",       // Evita FOIT (Flash of Invisible Text)
  variable: "--font-inter",
});

export default function RootLayout({ children }) {
  return (
    <html lang="es" className={inter.variable}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

### Ventajas sobre `@import url()` o `<link>`

- **Zero layout shift** (CLS = 0 para fuentes).
- **Self-hosted**: las fuentes se descargan en build y se sirven desde tu dominio (sin requests a Google Fonts).
- **Preload automático**: Next.js agrega `<link rel="preload">` automáticamente.

---

## 12. Core Web Vitals

Google usa Core Web Vitals como factor de ranking. Los umbrales objetivo:

| Métrica | Qué mide | Objetivo |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Velocidad de carga visual | ≤ 2.5s |
| **CLS** (Cumulative Layout Shift) | Estabilidad visual | ≤ 0.1 |
| **INP** (Interaction to Next Paint) | Responsividad | ≤ 200ms |

### Cómo optimizar cada métrica en Next.js

#### LCP (Largest Contentful Paint)

- Usa `priority` en la imagen hero / above-the-fold.
- Usa `next/font` para evitar bloqueo de renderizado por fuentes.
- Minimiza JavaScript del lado del cliente — prefiere Server Components.
- Usa SSG/ISR para que el HTML esté listo en el edge/CDN.

#### CLS (Cumulative Layout Shift)

- Siempre define `width` y `height` en `<Image>`.
- Usa `next/font` con `display: "swap"`.
- No insertes contenido dinámico above-the-fold sin reservar espacio.
- Evita anuncios o banners que desplazan contenido.

#### INP (Interaction to Next Paint)

- Mueve lógica pesada a Server Components (no envía JS al cliente).
- Usa `"use client"` solo donde sea estrictamente necesario.
- Implementa `React.lazy()` y `next/dynamic` para cargar componentes pesados bajo demanda.
- Evita hydration innecesaria.

### Lazy loading de componentes pesados

```tsx
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("@/components/HeavyChart"), {
  loading: () => <div className="h-96 animate-pulse bg-muted rounded-xl" />,
  ssr: false, // Si el componente no necesita SEO
});
```

---

## 13. Internacionalización (i18n)

Para sitios multi-idioma, Next.js App Router usa carpetas:

```
src/app/
├── [locale]/
│   ├── layout.tsx
│   ├── page.tsx
│   └── blog/
│       └── page.tsx
```

```tsx
// src/app/[locale]/layout.tsx
export async function generateStaticParams() {
  return [{ locale: "es" }, { locale: "en" }];
}

export default function LocaleLayout({ children, params }) {
  return (
    <html lang={params.locale}>
      <body>{children}</body>
    </html>
  );
}
```

### SEO para i18n

- Usa `<html lang="xx">` con el idioma correcto.
- Define `alternates.languages` en metadata para hreflang.
- Incluye todas las variantes de idioma en el sitemap.

---

## 14. Generative Engine Optimization (GEO)

En 2026, las IAs (ChatGPT, Perplexity, Google AI Overviews) citan páginas web. Para ser citado:

- **Headers jerárquicos**: usa un solo `<h1>` por página, seguido de `<h2>`, `<h3>` en orden lógico.
- **Respuesta directa en el primer párrafo**: responde la pregunta principal en las primeras 2-3 oraciones.
- **Listas y tablas**: las IAs prefieren datos estructurados que pueden extraer fácilmente.
- **Datos estructurados (JSON-LD)**: proporcionan relaciones entre entidades que las IAs pueden parsear.
- **Contenido factual y citable**: incluye cifras, porcentajes y datos verificables.
- **HTML semántico**: usa `<article>`, `<section>`, `<nav>`, `<header>`, `<footer>`, `<aside>`.

---

## 15. Anatomía de un resultado de búsqueda (SERP)

Antes de optimizar conviene saber cómo se llama cada parte del resultado, porque **no todas se controlan igual**: unas se declaran en código, otras las genera Google por algoritmo y no hay markup que las fuerce.

### Partes de un resultado

| Nombre | Qué es | ¿Se controla? |
|---|---|---|
| **Título** (title link) | El enlace azul | Parcial — sale del `<title>`, Google puede reescribirlo |
| **Snippet** (meta description) | Las 1-2 líneas de descripción bajo el título | Parcial — sale de `<meta name="description">`, Google puede reescribirlo |
| **URL / breadcrumb** | La ruta sobre el título | Sí — con URLs semánticas y schema `BreadcrumbList` |
| **Favicon** | El ícono junto al dominio | Sí — `icon.png` / `favicon.ico` |
| **Sitelinks** | Los sub-enlaces listados debajo del resultado principal | **No** — algorítmico |
| **Expanded sitelinks** | Sitelinks en lista vertical, cada uno con su propia descripción | **No** — algorítmico |
| **Jump-to links** | Enlaces directos a anclas (`#seccion`) dentro de la misma página | **No** — algorítmico, pero requiere `id` en el HTML |
| **Rich results** | Estrellas, FAQs desplegables, precios, breadcrumbs visuales | Sí — con JSON-LD (ver sección 8) |

### Snippet

Es el texto descriptivo bajo el título. Se declara así:

```tsx
export const metadata: Metadata = {
  description: "Descripción de 150-160 caracteres con propuesta de valor.",
};
```

Google **reescribe el snippet en ~70% de los casos** si considera que otro fragmento de la página responde mejor a la consulta. La `meta description` es una sugerencia fuerte, no una garantía.

Directivas relacionadas (sección 4):

| Directiva | Efecto |
|---|---|
| `"max-snippet": -1` | Sin límite de caracteres en el snippet |
| `"max-snippet": 0` | Sin snippet (solo título) |
| `"max-snippet": 160` | Máximo 160 caracteres |
| `data-nosnippet` | Atributo HTML: excluye ese bloque del snippet |

```html
<!-- Este bloque nunca aparecerá en el snippet -->
<span data-nosnippet>Precio sujeto a cambios.</span>
```

Para que el snippet salga bien:

- Escribe 150-160 caracteres, únicos por página.
- Incluye la keyword principal de forma natural.
- Añade propuesta de valor o CTA — es texto de venta, no un resumen técnico.
- Si Google la reescribe siempre, suele significar que no responde a la intención de búsqueda real.

### Sitelinks

Son los sub-enlaces que aparecen bajo un resultado. Cuando se muestran en lista vertical con descripción propia se llaman **expanded sitelinks**.

> **No existe forma de generarlos por código.** No hay schema, meta tag, archivo ni configuración de Search Console que los produzca. Google los arma por algoritmo cuando confía en la estructura del sitio, normalmente en búsquedas de marca.

Lo único que se puede hacer es influir indirectamente:

| Factor | Cómo trabajarlo |
|---|---|
| Estructura de navegación clara | Jerarquía plana y lógica, menú consistente en todo el sitio |
| Enlaces internos | Anchor text descriptivo y repetido igual en todo el sitio |
| Títulos únicos y descriptivos | Uno por página, sin plantillas duplicadas |
| URLs semánticas | `/servicios/auditoria` no `/?id=42` |
| Autoridad de marca | Volumen de búsquedas del nombre de la marca |
| Sitemap correcto | Solo URLs indexables y canónicas |

En Search Console ya **no** existe la herramienta de degradar sitelinks (retirada en 2016). La única vía indirecta hoy es aplicar `noindex` a la página que no quieres que aparezca como sitelink.

Un sitio de una sola página (one-page landing con anclas) casi nunca obtiene sitelinks: Google necesita varias páginas reales con contenido propio. Si los sitelinks son un objetivo, hay que convertir las secciones ancla en rutas independientes.

### Jump-to links

Son enlaces a anclas de la misma página (`tusitio.com/#servicios`) que Google muestra dentro del resultado. También son algorítmicos, pero **sí dependen de que el HTML lo permita**:

```tsx
// Cada sección con id estable y un heading claro
<section id="servicios">
  <h2>Servicios</h2>
  {/* ... */}
</section>
```

- Usa `id` estables y descriptivos (`#servicios`, no `#section-3`).
- Cada ancla debe tener un heading (`<h2>`/`<h3>`) que la describa.
- Enlaza esas anclas desde tu propia navegación.

Es la alternativa realista a los sitelinks para un landing de una sola página.

### Sitelinks Searchbox — retirado

El schema `SearchAction` dentro de `WebSite` generaba una caja de búsqueda dentro del resultado. **Google retiró este rich result en noviembre de 2024.**

```tsx
// Ya NO genera ningún rich result. Inofensivo, pero sin efecto.
potentialAction: {
  "@type": "SearchAction",
  target: "https://tusitio.com/search?q={search_term_string}",
  "query-input": "required name=search_term_string",
}
```

Puedes dejarlo (otros consumidores de datos estructurados lo leen) o eliminarlo. No afecta al ranking en ninguno de los dos casos.

---

## 16. Favicon e íconos

El favicon es la única parte del resultado de búsqueda que es un **archivo**, no una etiqueta. Aparece junto al dominio en los resultados de móvil y en la pestaña del navegador.

### Requisitos oficiales de Google

Fuente: [Favicon in Search — Google Search Central](https://developers.google.com/search/docs/appearance/favicon-in-search).

| Regla | Detalle |
|---|---|
| Tamaño mínimo | 8x8 px |
| Tamaño recomendado | Mayor a 48x48 px |
| Forma | Cuadrado (ratio 1:1) |
| Formato | Cualquier formato de favicon válido |
| URL estable | No cambiar la URL con frecuencia |
| Crawleable | Googlebot-Image debe poder leer el favicon **y** Googlebot la home. Ninguno bloqueado en `robots.txt` |
| Uno por sitio | Google Search soporta **un solo favicon por hostname** |
| Contenido | Sin material inapropiado — Google lo sustituye por el ícono por defecto |

Valores de `rel` que Google acepta:

```html
<link rel="icon" href="/icon.png">
<link rel="shortcut icon" href="/favicon.ico">
<link rel="apple-touch-icon" href="/apple-icon.png">
<link rel="apple-touch-icon-precomposed" href="/apple-icon.png">
```

El `href` puede ser relativo, absoluto o apuntar a un CDN externo.

### Convención de archivos en App Router

No hace falta escribir ningún `<link>`: Next.js genera las etiquetas a partir del nombre del archivo dentro de `app/`.

| Archivo | Etiqueta generada | Notas |
|---|---|---|
| `app/favicon.ico` | `<link rel="icon" type="image/x-icon">` | Solo en la raíz de `app/` |
| `app/icon.(ico\|jpg\|jpeg\|png\|svg)` | `<link rel="icon">` | Válido en cualquier segmento |
| `app/apple-icon.(jpg\|jpeg\|png)` | `<link rel="apple-touch-icon">` | Válido en cualquier segmento |
| `app/icon.tsx` | `<link rel="icon">` | Generado por código con `ImageResponse` |
| `app/apple-icon.tsx` | `<link rel="apple-touch-icon">` | Generado por código con `ImageResponse` |

Para varios tamaños, numera los archivos: `icon1.png`, `icon2.png`.

### Tamaños recomendados

| Archivo | Tamaño | Para qué |
|---|---|---|
| `favicon.ico` | 32x32 (o 16x16 + 32x32) | Pestaña del navegador, compatibilidad legacy |
| `icon.png` | 512x512 | Favicon moderno, resultados de Google, PWA |
| `apple-icon.png` | 180x180 | Pantalla de inicio en iOS |

Un favicon debería pesar **pocos KB**. Si tu `icon.png` pesa cientos de KB o más de 1 MB, está sin optimizar: se descarga en cada visita y engorda la imagen de despliegue.

### ⚠️ Gotcha: `metadata.icons` NO reemplaza a la convención de archivos

Si tienes `app/favicon.ico` y además declaras `icons` en la metadata, **Next.js emite ambos**. Los `<link>` se duplican:

```tsx
// ❌ Con app/favicon.ico y app/icon.png ya presentes, esto DUPLICA las etiquetas
export const metadata: Metadata = {
  icons: {
    icon: [{ url: "/favicon.ico" }, { url: "/icon.png", type: "image/png" }],
    shortcut: "/favicon.ico",
    apple: "/icon.png",
  },
};
```

```html
<!-- Resultado: 5 links de ícono para 2 archivos reales -->
<link rel="shortcut icon" href="/favicon.ico"/>
<link rel="icon" href="/favicon.ico?favicon.2hq3nsf.ico" sizes="256x256" type="image/x-icon"/>
<link rel="icon" href="/favicon.ico"/>
<link rel="icon" href="/icon.png" type="image/png"/>
<link rel="apple-touch-icon" href="/icon.png"/>
```

No rompe nada (Google elige uno), pero es ruido y los `sizes` declarados pueden no coincidir con el contenido real del archivo.

**Elige una vía, no las dos:**

- **Convención de archivos** (recomendado): coloca `favicon.ico`, `icon.png` y `apple-icon.png` en `app/` y **omite** `icons` en la metadata.
- **Metadata explícita**: usa `icons` solo si los íconos viven en `public/` o en un CDN, y entonces no pongas los archivos en `app/`.

### Íconos del manifest

El `manifest.ts` lleva su propio array de íconos, independiente de los `<link>`:

```ts
icons: [
  { src: "/icon.png", sizes: "512x512", type: "image/png", purpose: "any" },
  { src: "/icon-maskable.png", sizes: "512x512", type: "image/png", purpose: "maskable" },
],
```

`purpose: "maskable"` evita que Android recorte tu logo al aplicar su máscara circular.

### Errores comunes

| Error | Consecuencia |
|---|---|
| PNG de 1 MB como favicon | Descarga innecesaria en cada visita, imagen de despliegue más pesada |
| Duplicar convención de archivos + `metadata.icons` | `<link>` redundantes, `sizes` incorrectos |
| Copia huérfana en `public/` y en `app/` | Peso muerto en el bundle |
| Favicon no cuadrado | Google puede rechazarlo o deformarlo |
| Cambiar la URL del favicon en cada deploy | Google tarda en actualizarlo |
| Bloquear el favicon o la home en `robots.txt` | Google no lo muestra |

> El favicon se actualiza en los resultados cuando Google vuelve a rastrear la home. Puede tardar días o semanas, y no hay forma de forzarlo.

---

## 17. Checklist SEO rápido

### Antes del lanzamiento

- [ ] `<html lang="xx">` con el idioma correcto
- [ ] Un solo `<h1>` por página
- [ ] Título único por página (50-60 caracteres)
- [ ] Meta description única por página (150-160 caracteres)
- [ ] Canonical URL en cada página
- [ ] Open Graph tags (title, description, image 1200x630)
- [ ] Twitter Card tags
- [ ] `sitemap.xml` generado automáticamente
- [ ] `robots.txt` generado automáticamente
- [ ] JSON-LD con al menos `Organization` y `WebSite`
- [ ] JSON-LD `FAQPage` si tienes sección de FAQ
- [ ] JSON-LD `Product`/`SoftwareApplication` si tienes precios
- [ ] Favicon cuadrado, >48x48, de pocos KB (ver sección 16)
- [ ] Sin duplicar convención de archivos y `metadata.icons`
- [ ] `apple-icon.png` de 180x180
- [ ] `next/image` en todas las imágenes con `alt` descriptivo
- [ ] `next/font` para fuentes (sin `@import url()`)
- [ ] `priority` en la imagen LCP (hero)
- [ ] Sin contenido duplicado (títulos, descripciones)
- [ ] URLs limpias y semánticas (`/precios` no `/?section=pricing`)
- [ ] Links internos entre secciones/páginas
- [ ] `id` estables y descriptivos en cada sección ancla (habilita jump-to links)
- [ ] Anchor text consistente en toda la navegación (influye en sitelinks)
- [ ] HTTPS habilitado

### Core Web Vitals

- [ ] LCP ≤ 2.5s
- [ ] CLS ≤ 0.1
- [ ] INP ≤ 200ms
- [ ] `"use client"` solo donde sea necesario
- [ ] Componentes pesados con `next/dynamic` y `ssr: false`

### Post-lanzamiento

- [ ] Verificar sitio en Google Search Console
- [ ] Enviar sitemap manualmente en Search Console
- [ ] Validar JSON-LD con Rich Results Test
- [ ] Medir Core Web Vitals con PageSpeed Insights
- [ ] Configurar monitoreo continuo (Vercel Analytics, web-vitals)

---

## 18. Herramientas de validación

| Herramienta | Qué valida | URL |
|---|---|---|
| **Google Rich Results Test** | JSON-LD / structured data | https://search.google.com/test/rich-results |
| **Schema Markup Validator** | Schemas genéricos | https://validator.schema.org/ |
| **Google PageSpeed Insights** | Core Web Vitals + performance | https://pagespeed.web.dev/ |
| **Google Search Console** | Indexación, errores, CWV | https://search.google.com/search-console |
| **Lighthouse** (Chrome DevTools) | SEO, a11y, performance | F12 → Lighthouse |
| **Open Graph Debugger (Meta)** | OG tags preview | https://developers.facebook.com/tools/debug/ |
| **Twitter Card Validator** | Twitter cards preview | https://cards-dev.twitter.com/validator |
| **Ahrefs / Semrush** | Auditoría SEO completa | Herramientas de pago |

---

## 19. Fuentes

- [Documentación oficial SEO — Next.js](https://nextjs.org/learn/seo)
- [Next.js SEO Playbook — Vercel](https://vercel.com/blog/nextjs-seo-playbook)
- [Metadata API — Next.js Docs](https://nextjs.org/docs/app/getting-started/metadata-and-og-images)
- [generateMetadata — Next.js API Reference](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)
- [JSON-LD Guide — Next.js Docs](https://nextjs.org/docs/app/guides/json-ld)
- [robots.txt — Next.js Docs](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots)
- [Sitelinks — Google Search Central](https://developers.google.com/search/docs/appearance/sitelinks)
- [Favicon in Search — Google Search Central](https://developers.google.com/search/docs/appearance/favicon-in-search)
- [Metadata Files: favicon, icon, apple-icon — Next.js Docs](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons)
- [Snippets y meta descriptions — Google Search Central](https://developers.google.com/search/docs/appearance/snippet)
- [Retiro del Sitelinks Searchbox — Google Search Central](https://developers.google.com/search/blog/2024/10/sitelinks-search-box)
- [Core Web Vitals — Next.js](https://nextjs.org/learn/seo/improve)
- [Next.js SEO Optimization Guide 2026 — Djamware](https://www.djamware.com/post/nextjs-seo-optimization-guide-2026-edition)
- [How to Optimize SEO with Next.js in 2026 — DEV Community](https://dev.to/texavor/how-to-optimize-seo-with-nextjs-in-2026-1bhl)
- [Next.js 15 App Router SEO Checklist — DEV Community](https://dev.to/simplr_sh/nextjs-15-app-router-seo-comprehensive-checklist-3d3f)
- [Complete Next.js SEO Guide — Adeel Imran](https://www.adeelhere.com/blog/2025-12-09-complete-nextjs-seo-guide-from-zero-to-hero)
- [The Complete Next.js SEO Guide — Strapi](https://strapi.io/blog/nextjs-seo)

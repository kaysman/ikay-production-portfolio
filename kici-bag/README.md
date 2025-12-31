# KIÇI BAG - Industrial Polymer Supply Website

## What it is

A modern B2B website for KIÇI BAG, a Turkmenistan-based industrial polymer supplier specializing in PP, PE, PET, and PVC materials. The platform enables global customers to browse product catalogs, view technical specifications, and submit quote requests through an integrated contact system.

## My role

I designed and developed the complete website as a freelance project. Responsible for UI/UX design, component architecture, multilingual implementation (English/Turkish), Formspree contact form integration, responsive layout design with Tailwind CSS, and production deployment.

## Tech stack

**Frontend Framework:**
- React 19.2.3
- TypeScript 5.8.2
- Vite 6.2.0 (build tool)

**Routing:**
- React Router DOM 7.11.0

**Styling:**
- Tailwind CSS (utility-first CSS framework)
- Custom color scheme for industrial branding
- Responsive design system

**Icons:**
- Lucide React 0.562.0

**Form Handling:**
- Formspree (serverless contact form backend)
- No custom backend required

**Internationalization:**
- React Context for language switching
- JSON-based translation files
- English and Turkish support

## System architecture

### Component-Based Architecture

The project follows a simple, maintainable architecture suitable for a static marketing website:

```
kici-bag-industrial/
├── App.tsx                  # Main app component with routing
├── index.tsx               # React entry point
├── pages/                  # Page components
│   ├── Home.tsx           # Landing page
│   └── MainCategoryPage.tsx # Product category pages
├── components/            # Reusable UI components
│   ├── sections/         # Page sections (Hero, Products, Contact)
│   └── ui/               # Base UI components
├── context/              # React Context providers
│   └── LanguageContext.tsx # Multilingual support
├── data/                 # Static content and configurations
│   ├── products.json    # Product catalog data
│   └── translations.json # Localization strings
├── public/              # Static assets
│   └── images/          # Product images, company logos
└── types.ts            # TypeScript type definitions
```

### Data Flow

```
User Interaction
    ↓
React Router (page navigation)
    ↓
LanguageContext (current locale)
    ↓
Page Components (Home, MainCategoryPage)
    ↓
Section Components (Hero, ProductGrid, ContactForm)
    ↓
UI Components (Button, Card, Form)
```

### Multilingual System

```typescript
// Language Context provides translation function
interface LanguageContextType {
  language: 'en' | 'tr';
  setLanguage: (lang: 'en' | 'tr') => void;
  t: (key: string) => string; // Translation function
}

// Usage in components
const { t, language } = useLanguage();
return <h1>{t('hero.title')}</h1>;
```

## Key technical decisions

**React + Vite over Next.js:**
Chose static site generation over server-side rendering. No dynamic content requires server logic. Vite provides instant hot module replacement (HMR) and faster builds than webpack-based tools. Final output is pure HTML/CSS/JS deployable to any static host.

**Formspree over custom backend:**
Eliminated need for backend server, database, and email server setup. Formspree handles form submissions, spam filtering, and email delivery. Reduced infrastructure costs to zero and simplified deployment.

**Tailwind CSS over component libraries:**
Custom design requirements for industrial/B2B aesthetic. Tailwind utility classes provided flexibility without bloat of pre-built component libraries like Material-UI. Smaller bundle size (only used utilities compiled).

**React Router for client-side routing:**
Single-page application (SPA) provides instant page transitions. All routes pre-rendered at build time for SEO. No server required for routing logic.

**Context API over state management library:**
Language switching is only global state requirement. Redux/Zustand would add unnecessary complexity. React Context sufficient for simple language state shared across components.

**TypeScript for type safety:**
Prevented runtime errors in product catalog rendering. Type-safe product specifications (grade, density, melt flow rate) ensured consistent data structure.

## Notable challenges solved

**Multilingual content management:**
Challenge: Maintaining synchronized translations across English and Turkish for product specs and UI text. Solution: JSON-based translation files with nested keys. TypeScript types ensure all translation keys exist in both languages. Compile-time errors if translation missing.

**Product specification display:**
Challenge: Different polymer types have different technical properties (PP has melt flow rate, PET has intrinsic viscosity). Solution: Flexible product data schema with optional fields. TypeScript union types for product-specific properties. Conditional rendering based on product type.

**Responsive product grid:**
Challenge: Product cards need different layouts on desktop (3 columns), tablet (2 columns), mobile (1 column). Solution: Tailwind responsive grid utilities (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3`). Image aspect ratios preserved across breakpoints.

**SEO optimization without SSR:**
Challenge: Static site needs good search engine indexing. Solution: React Helmet for dynamic meta tags per page, semantic HTML structure, proper heading hierarchy, alt text for all images, Open Graph tags for social sharing.

**Contact form without backend:**
Challenge: Processing quote requests without server infrastructure. Solution: Formspree integration with `action="https://formspree.io/f/{form_id}"`. Form submissions sent to sales email automatically. Includes honeypot spam protection and reCAPTCHA integration.

## Code highlights

### [App.tsx](App.tsx) - Routing Setup

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { LanguageProvider } from './context/LanguageContext';
import Home from './pages/Home';
import MainCategoryPage from './pages/MainCategoryPage';

function App() {
  return (
    <LanguageProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/products/:category" element={<MainCategoryPage />} />
        </Routes>
      </BrowserRouter>
    </LanguageProvider>
  );
}
```

**Why this matters:** Clean separation of concerns. `LanguageProvider` wraps entire app for global translation access. React Router enables client-side navigation with clean URLs (`/products/polypropylene`).

### [context/LanguageContext.tsx](context/LanguageContext.tsx) - Internationalization

```typescript
interface LanguageContextType {
  language: 'en' | 'tr';
  setLanguage: (lang: 'en' | 'tr') => void;
  t: (key: string) => string;
}

export const LanguageProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [language, setLanguage] = useState<'en' | 'tr'>('en');

  const t = (key: string): string => {
    const keys = key.split('.');
    let value: any = translations[language];

    for (const k of keys) {
      value = value?.[k];
    }

    return value || key;
  };

  return (
    <LanguageContext.Provider value={{ language, setLanguage, t }}>
      {children}
    </LanguageContext.Provider>
  );
};
```

**Why this matters:** Nested translation key access (`t('hero.title.main')`) for organized translation files. Fallback to key if translation missing prevents blank UI. TypeScript ensures type-safe language codes.

### [data/products.json](data/products.json) - Product Catalog

Products organized by category (PP, PE, PET, PVC) with technical specifications:
- Grade/type
- Density ranges
- Melt flow rates
- Applications (injection molding, blow molding, film extrusion)
- Packaging information

### [components/sections/](components/sections/) - Page Sections

Modular sections: Hero, ProductGrid, SpecificationsTable, ContactForm, Footer. Each section self-contained with props interface for reusability.

## Deployment & environment

**Build Process:**
```bash
# Development
yarn dev  # Vite dev server on http://localhost:5173

# Production build
yarn build  # Outputs to dist/ directory
```

**Deployment Options:**
- **Netlify**: Drop folder deployment or Git integration
- **Vercel**: `vercel --prod`
- **Static hosting**: Any web server (Nginx, Apache, S3 + CloudFront)

**Formspree Configuration:**
- Form endpoint: `https://formspree.io/f/{form_id}`
- Email recipient: `salemanager@kicibag.com`
- Spam protection: Honeypot field + reCAPTCHA
- Notifications: Instant email on form submission

**Environment:**
- No environment variables required (static site)
- Optional: Google Analytics tracking ID (can be hardcoded)

**Performance:**
- Vite code splitting for optimal bundle sizes
- Image optimization (WebP with JPEG fallback)
- CSS purging (Tailwind removes unused utilities)
- Final bundle: ~150KB gzipped

**SEO:**
- Sitemap generated manually or via plugin
- robots.txt configured
- Meta descriptions per page
- Structured data for product catalog (Schema.org)

## Public links

Commercial project for KIÇI BAG Industrial. Website URL provided privately for portfolio review. Source code available upon request.

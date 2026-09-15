# Design System Master File

## Project overrides (these win over the generated tables below)

Направление задано референсом **prodent.kz** — светлый воздушный лейаут, крупная лёгкая
типографика, большие радиусы, bento-сетки. Идея взята, вёрстка своя.

**1. Палитра — светло-серый фон, брендовая бирюза.** Сгенерированный `#0891B2` не совпадает
с лого Like Clinic (зелёно-бирюзовое). Фон не белый, а светло-серый — карточки на нём читаются
как отдельные плоскости без теней.

| Токен | Hex | Назначение |
|---|---|---|
| `--bg` | `#EAEDEC` | фон страницы |
| `--card` | `#FFFFFF` | карточки |
| `--ink` | `#14201E` | основной текст |
| `--muted` | `#5C716D` | вторичный текст, 5.4:1 на фоне |
| `--faint` | `#A9B8B4` | вторая часть двухцветных заголовков, подписи |
| `--line` | `#DCE3E1` | хайрлайны |
| `--teal` | `#0D7F75` | кнопки (4.94:1 с белым — AA) |
| `--teal-dark` | `#0A6159` | ссылки, цены, hover (6.2:1 на белом) |
| `--dark` | `#1A2A28` | тёмные блоки и футер (12:1 с белым) |
| `--on-dark-faint` | `#7E9994` | текст на тёмном (4.6:1) |

**2. Типографика — Onest, один гарнитур.** Figtree не поддерживает кириллицу. Onest —
кириллический геометрический гротеск. Ключевое: **заголовки weight 400, не 700** — крупно
и легко, а не жирно и плотно. Это главное, что отделяет «дорогой» вид от шаблонного.

```css
@import url('https://fonts.googleapis.com/css2?family=Onest:wght@300;400;500;600&display=swap');
/* .h1 clamp(36px,5.2vw,64px)/400 · .h2 clamp(28px,3.6vw,46px)/400 · .h3 500 · цифры статистики 300 */
```

**3. Форма.** Карточки `20px`, крупные блоки и секции `40px` (на мобильных 16/28).
Пилюли `999px` — **только у кнопок и бейджа в hero**, дозированно. Раньше они были на всём
подряд, отсюда и шаблонность; проблема была в тотальности, а не в самой форме.

**4. Приёмы, которые держат стиль:**
- двухцветные заголовки: тёмная часть + `.dim` (`--faint`) — `Наши <span class="dim">направления</span>`
- bento-сетка 4 колонки со `span2`/`row2` для крупных плиток
- водяной знак `.svc-mark` — крупное слово в углу карточки, `rgba(20,32,30,.05)`
- статистика на тёмном: цифры weight 300 + вертикальные разделители
- преимущества строками с хайрлайнами (не карточками)
- отзывы горизонтальным рядом со `scroll-snap`
- «LIKE CLINIC» в футере как крупный водяной знак

**5. Hero — снимок входной группы, без портретов.** Правая часть первого экрана — фотография
стеклянной двери клиники с вывеской «Like Clinic · Твой путь к улыбке». Кадр уже скомпонован
под hero: слева воздух под текст, справа вывеска, свечение настоящее. Поэтому **никаких
слоёв тонировки и искусственной подсветки** — только `img` внутри `.hero-media`.

Ключевая деталь: в снимок **впечатано растворение в белый** слева. Фон страницы `#EAEDEC`,
поэтому белая точка кадра сведена к нему умножением каналов (`0xEA/255, 0xED/255, 0xEC/255`) —
иначе на стыке видно светлое пятно. После этого CSS-градиент не нужен вообще:

```python
r = r.point(lambda v: int(round(v * 0xEA/255)))   # и так по каждому каналу
```

`.hero-media { position:absolute; inset:0 }` + `object-position: right center`.
На `≤1000px` снимок перестаёт быть фоном: `.hero` → `flex-direction: column`, медиа получает
`order: 2` и `aspect-ratio: 1/1` — квадрат отрезает впечатанное растворение и оставляет в кадре
вывеску (при 3/2 в блок попадала пустая белая половина).

**6. Чипов-капсул на странице нет.** Бейдж «● Круглосуточно…» в hero заменён на `.hero-eyebrow` —
строку с точкой-маркером и хайрлайном снизу. Капсула как декоративный чип читается шаблонно;
`999px` остаётся только у кнопок, где он оправдан и совпадает с референсом.

**7. Карта — живой Яндекс-виджет, не картинка и не ссылка.**
`https://yandex.ru/map-widget/v1/?ll=<lon>%2C<lat>&z=17&pt=<lon>%2C<lat>%2Cpm2rdm`
— без ключа и регистрации, в iframe работает. Порядок в URL — **долгота, потом широта**.

Координаты взяты с карточки клиники в 2ГИС, не из геокодера: **43.260814, 76.960005**
(Nominatim по «Гоголя 20» давал две разные точки, обе мимо). Оттуда же — «ЖК Central Park
Residence, 2 этаж, вход со стороны Гоголя»; это добавлено в блок адреса.

`.map-card` — flex-колонка: iframe тянется (`flex:1`), снизу `.map-foot` с этажом и ссылкой
на 2ГИС (профиль клиники живёт там, поэтому ссылку сохраняем как вторичное действие).
У iframe обязательны `title` и `loading="lazy"`.

**8. Анимация.** Одна: `[data-anim]` → fade + 20px вверх, 700ms, стаггер 70ms внутри группы,
через IntersectionObserver. Плюс hover: подъём карточки, `scale(1.04)` на фото, сдвиг стрелки,
подчёркивание в меню. Всё отключается по `prefers-reduced-motion`.

---

> **LOGIC:** When building a specific page, first check `design-system/pages/[page-name].md`.
> If that file exists, its rules **override** this Master file.
> If not, strictly follow the rules below.

---

**Project:** Like Clinic Almaty
**Generated:** 2026-09-15 00:32:53
**Category:** Healthcare App
**Design Dials:** Variance 3/10 (Centered / Minimal) | Motion 3/10 (Subtle) | Density 4/10 (Standard)

---

## Global Rules

### Color Palette

| Role | Hex | CSS Variable |
|------|-----|--------------|
| Primary | `#0891B2` | `--color-primary` |
| On Primary | `#000000` | `--color-on-primary` |
| Secondary | `#22D3EE` | `--color-secondary` |
| On Secondary | `#0F172A` | `--color-on-secondary` |
| Accent/CTA | `#059669` | `--color-accent` |
| On Accent/CTA | `#000000` | `--color-on-accent` |
| Background | `#ECFEFF` | `--color-background` |
| Foreground | `#164E63` | `--color-foreground` |
| Card | `#FFFFFF` | `--color-card` |
| Card Foreground | `#164E63` | `--color-card-foreground` |
| Muted | `#E8F1F6` | `--color-muted` |
| Muted Foreground | `#475569` | `--color-muted-foreground` |
| Border | `#A5F3FC` | `--color-border` |
| Destructive | `#DC2626` | `--color-destructive` |
| On Destructive | `#FFFFFF` | `--color-on-destructive` |
| Ring | `#0891B2` | `--color-ring` |

**Color Notes:** Calm cyan + health green

### Typography

- **Heading Font:** Figtree
- **Body Font:** Noto Sans
- **Mood:** medical, clean, accessible, professional, healthcare, trustworthy
- **Google Fonts:** [Figtree + Noto Sans](https://fonts.googleapis.com/css2?family=Figtree:wght@300;400;500;600;700&family=Noto+Sans:wght@300;400;500;700&display=swap)

**CSS Import:**
```css
@import url('https://fonts.googleapis.com/css2?family=Figtree:wght@300;400;500;600;700&family=Noto+Sans:wght@300;400;500;700&display=swap');
```

### Spacing Variables

*Density: 4/10 — Standard*

| Token | Value | Usage |
|-------|-------|-------|
| `--space-xs` | `4px` / `0.25rem` | Tight gaps |
| `--space-sm` | `8px` / `0.5rem` | Icon gaps, inline spacing |
| `--space-md` | `16px` / `1rem` | Standard padding |
| `--space-lg` | `24px` / `1.5rem` | Section padding |
| `--space-xl` | `32px` / `2rem` | Large gaps |
| `--space-2xl` | `48px` / `3rem` | Section margins |
| `--space-3xl` | `64px` / `4rem` | Hero padding |

### Shadow Depths

| Level | Value | Usage |
|-------|-------|-------|
| `--shadow-sm` | `0 1px 2px rgba(0,0,0,0.05)` | Subtle lift |
| `--shadow-md` | `0 4px 6px rgba(0,0,0,0.1)` | Cards, buttons |
| `--shadow-lg` | `0 10px 15px rgba(0,0,0,0.1)` | Modals, dropdowns |
| `--shadow-xl` | `0 20px 25px rgba(0,0,0,0.15)` | Hero images, featured cards |

---

## Component Specs

### Buttons

```css
/* Primary Button */
.btn-primary {
  background: #059669;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  font-weight: 600;
  transition: all 200ms ease;
  cursor: pointer;
}

.btn-primary:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

/* Secondary Button */
.btn-secondary {
  background: transparent;
  color: #0891B2;
  border: 2px solid #0891B2;
  padding: 12px 24px;
  border-radius: 8px;
  font-weight: 600;
  transition: all 200ms ease;
  cursor: pointer;
}
```

### Cards

```css
.card {
  background: #ECFEFF;
  border-radius: 12px;
  padding: 24px;
  box-shadow: var(--shadow-md);
  transition: all 200ms ease;
  cursor: pointer;
}

.card:hover {
  box-shadow: var(--shadow-lg);
  transform: translateY(-2px);
}
```

### Inputs

```css
.input {
  padding: 12px 16px;
  border: 1px solid #E2E8F0;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 200ms ease;
}

.input:focus {
  border-color: #0891B2;
  outline: none;
  box-shadow: 0 0 0 3px #0891B220;
}
```

### Modals

```css
.modal-overlay {
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
}

.modal {
  background: white;
  border-radius: 16px;
  padding: 32px;
  box-shadow: var(--shadow-xl);
  max-width: 500px;
  width: 90%;
}
```

---

## Style Guidelines

**Style:** Minimalism & Swiss Style

**Keywords:** Clean, simple, spacious, functional, white space, high contrast, geometric, sans-serif, grid-based, essential

**Best For:** Enterprise apps, dashboards, documentation sites, SaaS platforms, professional tools

**Key Effects:** Subtle hover (200-250ms), smooth transitions, sharp shadows if any, clear type hierarchy, fast loading

### Page Pattern

**Pattern Name:** Hero + Testimonials + CTA

- **Conversion Strategy:** Social proof before CTA. Use a concise set of verified testimonials with photo, name, and role. CTA after social proof. Provide previous/next and pause controls; stop rotation on focus, hover, and reduced motion; announce slide position. Previous/next buttons and keyboard controls must expose every slide without dragging.
- **CTA Placement:** Hero (sticky) + Post-testimonials
- **Section Order:** Hero > Problem statement > Solution overview > Testimonials carousel > CTA

---

## Motion

**Scroll Reveal** (Subtle) — Trigger: scroll (viewport enter) | Duration: 300-400ms | Easing: `power1.out`

```js
gsap.from(el, { opacity: 0, y: 12, duration: 0.35, ease: 'power1.out', scrollTrigger: { trigger: el, start: 'top 90%', toggleActions: 'play none none reverse' } });
```

**Framework notes:** Requires the ScrollTrigger plugin registered once via gsap.registerPlugin(ScrollTrigger); Use matchMedia('(prefers-reduced-motion: reduce)') to skip non-essential motion and render the final state immediately

- ✅ Keep the y offset small (8-16px) so it reads as a fade, not a slide
- ❌ Don't reveal below-the-fold content needed for SEO/crawlers as invisible-by-default without a no-JS fallback
- ⚡ toggleActions 'play none none reverse' avoids re-triggering on every scroll direction change

---

## Anti-Patterns (Do NOT Use)

- ❌ Bright neon colors
- ❌ Motion-heavy animations
- ❌ AI purple/pink gradients

### Additional Forbidden Patterns

- ❌ **Emojis as icons** — Use SVG icons (Heroicons, Lucide, Simple Icons)
- ❌ **Missing cursor:pointer** — All clickable elements must have cursor:pointer
- ❌ **Layout-shifting hovers** — Avoid scale transforms that shift layout
- ❌ **Low contrast text** — Maintain 4.5:1 minimum contrast ratio
- ❌ **Instant state changes** — Always use transitions (150-300ms)
- ❌ **Invisible focus states** — Focus states must be visible for a11y

---

## Pre-Delivery Checklist

Before delivering any UI code, verify:

- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] `cursor-pointer` on all clickable elements
- [ ] Hover states with smooth transitions (150-300ms)
- [ ] Light mode: text contrast 4.5:1 minimum
- [ ] Focus states visible for keyboard navigation
- [ ] `prefers-reduced-motion` respected
- [ ] Responsive: 375px, 768px, 1024px, 1440px
- [ ] No content hidden behind fixed navbars
- [ ] No horizontal scroll on mobile

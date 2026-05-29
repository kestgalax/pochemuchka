---
name: pochemuchka-render
description: Рендерит markdown-результат навыка pochemuchka (или аналогичный markdown с цепочкой 5 Why, корнем, линзой) в полностью автономную HTML-страницу с editorial UI/UX. Используй, когда пользователь просит "оформи это как страницу", "сделай HTML", "визуализируй почемучку", "сверстай", "преврати в веб-страницу".
---

# pochemuchka-render

> Inspired by taste-skill: https://github.com/Leonxlnx/taste-skill

## Зачем этот навык

Результат почемучки — это текстовый анализ. Он мощный для мышления, но плохо читается при передаче команде, в презентации, в архив. Этот навык превращает markdown-результат в полностью автономную HTML-страницу, которую можно открыть в браузере, отправить коллеге, положить в репозиторий.

Ключевое требование: **один файл, никаких зависимостей**. Открываешь — работает.

## Что это делает

1. Принимает markdown в формате выхода `pochemuchka` (тезис + цепочка "Почему?" + Корень + Линза + Переход)
2. Парсит структуру
3. Генерирует `*.html` — полностью self-contained, без CDN, без внешних шрифтов, без внешних CSS
4. Пишет файл в `pochemuchka-results/` (создаёт директорию при необходимости)

## Когда включать

- Пользователь сказал: "оформи это как страницу", "сделай HTML", "визуализируй почемучку"
- Пользователь приложил `.md` файл с результатом pochemuchka
- Пользователь вставил markdown с цепочкой "Почему?" и просит "сверстать"
- После работы навыка `pochemuchka` пользователь просит "а теперь сделай красиво"

## Алгоритм

### Шаг 0: Получить markdown

Приоритет источника:
1. Markdown, приложенный пользователем в текущем сообщении (файл или текст).
2. Результат текущей сессии: если только что был сгенерирован разбор навыком `pochemuchka` — используй его.
3. Файл в `pochemuchka-results/` (самый новый по дате или с наиболее похожим именем).

Если источник не ясен — спроси пользователя, какой разбор визуализировать.

### Шаг 1: Парсинг структуры

Из markdown извлеки:

| Что искать | Как выглядит | Куда в HTML |
|---|---|---|
| Заголовок документа | `# Заголовок` или `**Тезис:** ...` | `<title>`, hero headline |
| Тезис | `**Тезис:** ...` или первый абзац после заголовка | Hero subtext |
| Дата | `**Дата:** YYYY-MM-DD` (опционально) | Hero eyebrow, footer |
| Цепочка "Почему?" | `## Цепочка "Почему?"` → `**Почему N:**` | Timeline секция |
| Вопрос | Текст после `**Почему N:**` | Timeline заголовок |
| Ответ | Абзац/цитата (`>`) после вопроса | Timeline body / blockquote |
| Корень | `## Корень` → текст | Root Cause Card |
| Линза | `## Линза:` → markdown-таблица | Lens Transformations |
| Переход | `## Переход` → список вопросов | Reflection Questions |
| Follow-up | `Хочешь, запустим почемучку глубже...` | Опускать по умолчанию |

**Важно:** Если в markdown ответы оформлены как blockquote (`>`), в HTML они должны стать блоком с левой акцентной полосой и italic.

**Если какого-то блока нет** (например, нет "Перехода" или нет даты) — пропусти секцию, не генерируй пустой блок.

### Шаг 2: Определить имя файла

- Если пользователь приложил файл `something.md` → сохраняй как `pochemuchka-results/something.html`.
- Если исходник из текущей сессии и уже есть сохранённый markdown в `pochemuchka-results/` → используй то же имя, но с расширением `.html`.
- Если пользователь просто вставил текст → сохраняй как `pochemuchka-results/pochemuchka-result.html` (или запроси имя).

**Путь:** `pochemuchka-results/` в корне проекта. Если директория не существует — создай её автоматически.

### Шаг 3: Генерация HTML

Используй **нижеописанный шаблон**. Все стили и скрипты — внутри файла. НИКАКИХ внешних зависимостей.

## HTML-шаблон (self-contained)

Генерируй HTML-файл со следующей структурой. CSS и JS — внутри `<style>` и `<script>`.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{HEADLINE}}</title>
  <style>
    /* ========== CSS VARIABLES ========== */
    :root {
      --bg: #fafafa;
      --text: #18181b;
      --text-secondary: #52525b;
      --text-muted: #a1a1aa;
      --surface: #ffffff;
      --surface-elevated: #f4f4f5;
      --border: #e4e4e7;
      --accent: #0d9488;
      --accent-light: #14b8a6;
      --accent-dark: #0f766e;
      --accent-bg: #f0fdfa;
      --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
      --font-mono: "SF Mono", SFMono-Regular, Menlo, Monaco, Consolas, monospace;
    }

    [data-theme="dark"] {
      --bg: #09090b;
      --text: #fafafa;
      --text-secondary: #a1a1aa;
      --text-muted: #71717a;
      --surface: #18181b;
      --surface-elevated: #27272a;
      --border: #3f3f46;
      --accent: #2dd4bf;
      --accent-light: #5eead4;
      --accent-dark: #14b8a6;
      --accent-bg: #134e4a;
    }

    /* ========== BASE ========== */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    html { scroll-behavior: smooth; -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale; }
    body {
      font-family: var(--font-sans);
      background: var(--bg);
      color: var(--text);
      line-height: 1.6;
      transition: background 0.3s, color 0.3s;
    }
    ::selection { background: var(--accent-bg); color: var(--accent); }

    /* ========== UTILITIES ========== */
    .container { max-width: 1100px; margin: 0 auto; padding: 0 24px; }
    @media (min-width: 768px) { .container { padding: 0 48px; } }

    /* ========== TYPOGRAPHY ========== */
    .eyebrow {
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 0.18em;
      color: var(--text-muted);
      font-weight: 500;
      margin-bottom: 16px;
    }
    .display {
      font-size: clamp(2rem, 5vw, 3.75rem);
      line-height: 1.05;
      letter-spacing: -0.02em;
      font-weight: 700;
      color: var(--text);
    }
    .lead {
      font-size: clamp(1.125rem, 2.5vw, 1.25rem);
      line-height: 1.6;
      color: var(--text-secondary);
      max-width: 55ch;
    }
    .body-text {
      font-size: 1rem;
      line-height: 1.6;
      color: var(--text-secondary);
      max-width: 65ch;
    }
    .mono-label {
      font-family: var(--font-mono);
      font-size: 10px;
      text-transform: uppercase;
      letter-spacing: 0.2em;
      color: var(--text-muted);
      font-weight: 600;
    }

    /* ========== ANIMATION ========== */
    .reveal {
      opacity: 0;
      transform: translateY(24px);
      transition: opacity 0.7s cubic-bezier(0.16, 1, 0.3, 1), transform 0.7s cubic-bezier(0.16, 1, 0.3, 1);
    }
    .reveal.visible { opacity: 1; transform: translateY(0); }
    @media (prefers-reduced-motion: reduce) {
      .reveal { opacity: 1; transform: none; transition: none; }
    }

    /* ========== THEME TOGGLE ========== */
    .theme-toggle {
      position: fixed;
      top: 24px;
      right: 24px;
      z-index: 50;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: var(--surface);
      border: 1px solid var(--border);
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: transform 0.2s, box-shadow 0.2s;
    }
    .theme-toggle:hover { transform: scale(1.05); box-shadow: 0 4px 12px rgba(0,0,0,0.15); }
    .theme-toggle:focus { outline: none; box-shadow: 0 0 0 2px var(--accent); }
    .theme-toggle svg { width: 20px; height: 20px; }
    .theme-toggle .sun { color: #f59e0b; }
    .theme-toggle .moon { color: #818cf8; }

    /* ========== HERO ========== */
    .hero {
      min-height: 80dvh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      padding: 80px 0 64px;
      position: relative;
    }
    .hero-content { max-width: 65ch; position: relative; z-index: 2; }
    .hero-meta {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-bottom: 24px;
    }
    .hero-meta .dot { color: var(--border); }
    .hero-deco {
      position: absolute;
      right: 0;
      top: 96px;
      width: 300px;
      height: 300px;
      opacity: 0.04;
      pointer-events: none;
      display: none;
    }
    @media (min-width: 1024px) { .hero-deco { display: block; } }

    /* ========== SECTIONS ========== */
    .section { padding: 64px 0; }
    @media (min-width: 768px) { .section { padding: 96px 0; } }
    .section-title {
      font-size: clamp(1.75rem, 4vw, 2.5rem);
      letter-spacing: -0.02em;
      line-height: 1.1;
      color: var(--text);
      margin-bottom: 16px;
    }
    .section-subtitle {
      color: var(--text-secondary);
      max-width: 55ch;
      margin-bottom: 48px;
    }

    /* ========== TIMELINE ========== */
    .timeline { position: relative; }
    .timeline::before {
      content: '';
      position: absolute;
      left: 20px;
      top: 8px;
      bottom: 8px;
      width: 1px;
      background: linear-gradient(to bottom, var(--border), var(--text-muted), var(--border));
    }
    @media (min-width: 768px) {
      .timeline::before { left: 88px; }
    }
    .timeline-item {
      position: relative;
      display: grid;
      grid-template-columns: 1fr;
      gap: 16px;
      margin-bottom: 64px;
    }
    @media (min-width: 768px) {
      .timeline-item { grid-template-columns: 120px 1fr; gap: 32px; margin-bottom: 80px; }
    }
    .timeline-item:last-child { margin-bottom: 0; }

    .timeline-marker {
      position: absolute;
      left: 8px;
      top: 0;
      width: 24px;
      height: 24px;
      border-radius: 50%;
      background: var(--accent);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 10;
      font-family: var(--font-mono);
      font-size: 10px;
      font-weight: 700;
      color: #fff;
    }
    @media (min-width: 768px) {
      .timeline-marker { left: 76px; }
    }

    .timeline-marker.accent { background: #0d9488; }
    [data-theme="dark"] .timeline-marker.accent { background: #2dd4bf; }
    .timeline-marker.accent-light { background: #14b8a6; }
    [data-theme="dark"] .timeline-marker.accent-light { background: #5eead4; }
    .timeline-marker.accent-dark { background: #0f766e; }
    [data-theme="dark"] .timeline-marker.accent-dark { background: #14b8a6; }

    .timeline-content {
      padding-left: 56px;
    }
    @media (min-width: 768px) {
      .timeline-content { padding-left: 0; }
    }

    .timeline-eyebrow {
      font-size: 10px;
      text-transform: uppercase;
      letter-spacing: 0.2em;
      color: var(--text-muted);
      font-weight: 600;
      margin-bottom: 4px;
    }
    @media (max-width: 767px) {
      .timeline-eyebrow { display: none; }
    }

    .timeline-mobile-label {
      font-family: var(--font-mono);
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 0.18em;
      color: var(--text-muted);
      margin-bottom: 8px;
      padding-left: 32px;
    }
    @media (min-width: 768px) {
      .timeline-mobile-label { display: none; }
    }

    .timeline-question {
      font-size: clamp(1.125rem, 2vw, 1.25rem);
      font-weight: 600;
      color: var(--text);
      line-height: 1.35;
      margin-bottom: 12px;
    }

    .timeline-answer {
      color: var(--text-secondary);
      line-height: 1.6;
      max-width: 60ch;
    }

    .timeline-blockquote {
      position: relative;
      padding-left: 16px;
      margin-top: 12px;
      border-left: 2px solid var(--accent-bg);
    }
    .timeline-blockquote p {
      color: var(--text-secondary);
      font-style: italic;
      line-height: 1.6;
      max-width: 60ch;
    }

    /* ========== ROOT CARD ========== */
    .root-card {
      position: relative;
      background: var(--surface);
      border-radius: 16px;
      padding: 32px;
      border: 1px solid var(--border);
      box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    }
    @media (min-width: 768px) { .root-card { padding: 48px; } }
    .root-card::before {
      content: '';
      position: absolute;
      left: 0;
      top: 32px;
      bottom: 32px;
      width: 4px;
      background: var(--accent);
      border-radius: 0 4px 4px 0;
    }
    .root-card-inner { padding-left: 16px; }
    .root-label {
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.18em;
      color: var(--accent);
      font-weight: 600;
      margin-bottom: 16px;
      display: block;
    }
    .root-title {
      font-size: clamp(1.25rem, 3vw, 1.75rem);
      line-height: 1.3;
      letter-spacing: -0.01em;
      color: var(--text);
      margin-bottom: 16px;
      max-width: 50ch;
    }
    .root-body {
      color: var(--text-secondary);
      line-height: 1.7;
      max-width: 65ch;
    }

    /* ========== LENS ========== */
    .lens-grid { display: flex; flex-direction: column; gap: 16px; max-width: 900px; }
    .lens-card {
      background: var(--surface);
      border-radius: 12px;
      padding: 24px;
      border: 1px solid var(--border);
      transition: border-color 0.2s, box-shadow 0.2s;
    }
    .lens-card:hover { border-color: var(--accent-light); }
    .lens-row {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
    @media (min-width: 768px) {
      .lens-row { flex-direction: row; align-items: flex-start; gap: 24px; }
    }
    .lens-col { flex: 1; min-width: 0; }
    .lens-arrow {
      display: flex;
      align-items: center;
      justify-content: center;
      padding-top: 8px;
    }
    .lens-arrow svg { width: 24px; height: 24px; color: var(--accent); flex-shrink: 0; }
    .lens-meaning {
      padding-top: 12px;
      border-top: 1px solid var(--border);
      margin-top: 12px;
    }
    @media (min-width: 768px) {
      .lens-meaning {
        padding-top: 0;
        padding-left: 24px;
        border-top: none;
        border-left: 1px solid var(--border);
        margin-top: 0;
        width: 220px;
        flex-shrink: 0;
      }
    }
    .lens-label { margin-bottom: 6px; }
    .lens-was { color: var(--text-secondary); line-height: 1.6; }
    .lens-became { color: var(--text); font-weight: 500; line-height: 1.6; }
    .lens-meaning-text { font-size: 0.9375rem; color: var(--text-secondary); line-height: 1.5; }
    .lens-meaning-text strong { color: var(--accent); }

    /* ========== REFLECTION ========== */
    .reflection-list { display: flex; flex-direction: column; gap: 24px; max-width: 900px; }
    .reflection-card {
      background: var(--surface);
      border-radius: 12px;
      padding: 24px;
      border: 1px solid var(--border);
      transition: box-shadow 0.2s, border-color 0.2s;
    }
    .reflection-card:hover { box-shadow: 0 4px 20px rgba(0,0,0,0.08); border-color: var(--accent-light); }
    .reflection-row { display: flex; gap: 20px; }
    .reflection-badge {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: var(--accent-bg);
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
    }
    .reflection-badge span {
      font-family: var(--font-mono);
      font-size: 14px;
      font-weight: 700;
      color: var(--accent);
    }
    .reflection-text { font-size: 1.125rem; line-height: 1.6; color: var(--text); }

    /* ========== QUOTE ========== */
    .quote-section { text-align: center; padding: 64px 0; }
    .quote-icon {
      width: 40px;
      height: 40px;
      color: var(--accent-bg);
      margin: 0 auto 32px;
    }
    .quote-text {
      font-size: clamp(1.5rem, 4vw, 2.5rem);
      font-weight: 300;
      line-height: 1.3;
      letter-spacing: -0.02em;
      color: var(--text);
      margin-bottom: 32px;
    }
    .quote-text .accent { color: var(--accent); }
    .quote-line {
      width: 64px;
      height: 1px;
      background: var(--border);
      margin: 0 auto;
    }

    /* ========== FOOTER ========== */
    .footer {
      padding: 48px 0;
      border-top: 1px solid var(--border);
    }
    .footer-inner {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }
    @media (min-width: 768px) {
      .footer-inner { flex-direction: row; justify-content: space-between; align-items: center; }
    }
    .footer-text {
      font-size: 0.875rem;
      color: var(--text-muted);
    }
  </style>
</head>
<body>

  <!-- THEME TOGGLE -->
  <button class="theme-toggle" id="themeToggle" aria-label="Переключить тему">
    <svg class="sun" id="sunIcon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="display:none;">
      <circle cx="12" cy="12" r="5"/><path d="M12 1v2m0 18v2M4.22 4.22l1.42 1.42m12.72 12.72l1.42 1.42M1 12h2m18 0h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"/>
    </svg>
    <svg class="moon" id="moonIcon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="display:none;">
      <path d="M21 12.79A9 9 0 1111.21 3 7 7 0 0021 12.79z"/>
    </svg>
  </button>

  <script>
    (function() {
      var html = document.documentElement;
      var toggle = document.getElementById('themeToggle');
      var sun = document.getElementById('sunIcon');
      var moon = document.getElementById('moonIcon');
      function updateIcons() {
        if (html.getAttribute('data-theme') === 'dark') {
          sun.style.display = 'block'; moon.style.display = 'none';
        } else {
          sun.style.display = 'none'; moon.style.display = 'block';
        }
      }
      var saved = localStorage.getItem('pochemuchka-theme');
      if (saved === 'dark') { html.setAttribute('data-theme', 'dark'); }
      else if (saved === 'light') { html.removeAttribute('data-theme'); }
      else if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
        html.setAttribute('data-theme', 'dark');
      }
      updateIcons();
      toggle.addEventListener('click', function() {
        if (html.getAttribute('data-theme') === 'dark') {
          html.removeAttribute('data-theme'); localStorage.setItem('pochemuchka-theme', 'light');
        } else {
          html.setAttribute('data-theme', 'dark'); localStorage.setItem('pochemuchka-theme', 'dark');
        }
        updateIcons();
      });
    })();
  </script>

  <main class="container">

    <!-- HERO -->
    <section class="hero">
      <div class="hero-deco" aria-hidden="true">
        <svg viewBox="0 0 200 200" fill="none" style="width:100%;height:100%;color:var(--text);">
          <circle cx="100" cy="40" r="6" fill="currentColor"/><circle cx="60" cy="90" r="6" fill="currentColor"/><circle cx="140" cy="90" r="6" fill="currentColor"/>
          <circle cx="40" cy="150" r="6" fill="currentColor"/><circle cx="100" cy="150" r="6" fill="currentColor"/><circle cx="160" cy="150" r="6" fill="currentColor"/>
          <path d="M100 46L60 84M100 46L140 84M60 96L40 144M60 96L100 144M140 96L100 144M140 96L160 144" stroke="currentColor" stroke-width="1.5"/>
        </svg>
      </div>
      <div class="hero-content">
        <div class="hero-meta">
          <span class="eyebrow">{{EYEBROW}}</span>
          {{DATE}}
        </div>
        <h1 class="display">{{HEADLINE}}</h1>
        <p class="lead" style="margin-top:24px;">{{THESIS}}</p>
      </div>
    </section>

    <!-- TIMELINE -->
    <section class="section">
      <div class="reveal">
        <h2 class="section-title">Цепочка «Почему?»</h2>
      </div>
      <div class="timeline" style="margin-top:48px;">
        {{WHY_ITEMS}}
      </div>
    </section>

    <!-- ROOT -->
    <section class="section">
      <div class="reveal">
        <div class="root-card">
          <div class="root-card-inner">
            <span class="root-label">Корень проблемы</span>
            <h2 class="root-title">{{ROOT_TITLE}}</h2>
            <p class="root-body">{{ROOT_BODY}}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- LENS -->
    <section class="section">
      <div class="reveal">
        <h2 class="section-title">Линза переосмысления</h2>
        <p class="section-subtitle">Данные → Контекст, Проблемы → Намерение, Решения → Принципы</p>
      </div>
      <div class="lens-grid reveal" style="margin-top:32px;">
        {{LENS_ROWS}}
      </div>
    </section>

    <!-- REFLECTION -->
    <section class="section">
      <div class="reveal" style="max-width:900px;margin:0 auto;">
        <p class="mono-label" style="margin-bottom:12px;">Переход</p>
        <h2 class="section-title">Вопросы для размышления</h2>
        <div class="reflection-list" style="margin-top:40px;">
          {{REFLECTION_ITEMS}}
        </div>
      </div>
    </section>

    <!-- QUOTE -->
    <section class="quote-section">
      <div class="reveal">
        <svg class="quote-icon" viewBox="0 0 32 32" fill="currentColor">
          <path d="M10 8c-3.3 0-6 2.7-6 6v10h10V14H8c0-1.1.9-2 2-2V8zm14 0c-3.3 0-6 2.7-6 6v10h10V14h-6c0-1.1.9-2 2-2V8z"/>
        </svg>
        <blockquote class="quote-text">
          Понимай ситуацию.<br><span class="accent">Опиши желаемое.</span><br>Доверься принципам.
        </blockquote>
        <div class="quote-line"></div>
      </div>
    </section>

    <!-- FOOTER -->
    <footer class="footer">
      <div class="footer-inner">
        <p class="footer-text">Анализ методом почемучка{{DATE_FOOTER}}</p>
        <p class="footer-text">{{TOPIC}}</p>
      </div>
    </footer>
  </main>

  <script>
    (function() {
      if (window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
      var observer = new IntersectionObserver(function(entries) {
        entries.forEach(function(entry) {
          if (entry.isIntersecting) {
            entry.target.classList.add('visible');
            observer.unobserve(entry.target);
          }
        });
      }, { threshold: 0.08, rootMargin: '0px 0px -40px 0px' });
      document.querySelectorAll('.reveal').forEach(function(el) { observer.observe(el); });
    })();
  </script>

</body>
</html>
```

## Подстановки в шаблон

### `{{EYEBROW}}`
Извлекается из контекста. Если дата есть → `Анализ онбординга` / `Анализ системной проблемы` / etc. Если нет → просто `Анализ методом почемучка`.

### `{{DATE}}`
Если в markdown есть `**Дата:** YYYY-MM-DD`:
```html
<span class="dot" style="color:var(--border);">·</span>
<span class="eyebrow" style="font-family:var(--font-mono);">2026-05-29</span>
```
Если даты нет — пустая строка.

### `{{HEADLINE}}`
Из `# Заголовок` или из первого предложения тезиса (сокращённое). Если заголовок очень длинный — сократи до сути, но не режь мысль.

### `{{THESIS}}`
Текст после `**Тезис:**` или первый абзац после заголовка. Как есть, без обрезки.

### `{{WHY_ITEMS}}`
Генерируется из каждого `**Почему N:**`. Формат каждого item:

```html
<div class="timeline-item reveal">
  <div><!-- empty left col --></div>
  <div class="timeline-content">
    <div class="timeline-marker {{MARKER_CLASS}}"><span>N</span></div>
    <div class="timeline-mobile-label">Почему N</div>
    <div class="timeline-eyebrow">Почему</div>
    <h3 class="timeline-question">{{QUESTION}}</h3>
    {{ANSWER}}
  </div>
</div>
```

`{{MARKER_CLASS}}` — цветовой класс для маркера (см. ниже).
`{{ANSWER}}` — если был blockquote (`>`):
```html
<div class="timeline-blockquote"><p>{{ANSWER_TEXT}}</p></div>
```
Иначе:
```html
<p class="timeline-answer">{{ANSWER_TEXT}}</p>
```

**Accent colors для маркеров (класс `{{MARKER_CLASS}}`):**
- Почему 1: `accent` (первый, начало цепочки)
- Почему 2–N−1: `accent-light` (середина)
- Последний: `accent-dark` (конец цепочки)

Если цепочка из 3 — первый `accent`, второй `accent-light`, третий `accent-dark`.

### `{{ROOT_TITLE}}`
Первое предложение из секции "Корень" (краткое, для заголовка карточки).

### `{{ROOT_BODY}}`
Весь оставшийся текст из секции "Корень".

### `{{LENS_ROWS}}`
Для каждой строки таблицы:

```html
<div class="lens-card reveal">
  <div class="lens-row">
    <div class="lens-col">
      <div class="lens-label mono-label">Было</div>
      <p class="lens-was">{{WAS_TEXT}}</p>
    </div>
    <div class="lens-arrow">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M5 12h14M12 5l7 7-7 7"/>
      </svg>
    </div>
    <div class="lens-col">
      <div class="lens-label mono-label" style="color:var(--accent);">Стало</div>
      <p class="lens-became">{{BECAME_TEXT}}</p>
    </div>
    <div class="lens-meaning">
      <p class="lens-meaning-text">{{MEANING_TEXT}}</p>
    </div>
  </div>
</div>
```

`{{MEANING_TEXT}}` — текст из колонки "Для человека это" (или "Для человека"). Если там жирный текст (`**текст**`), оберни в `<strong style="color:var(--accent);">`.

### `{{REFLECTION_ITEMS}}`
Для каждого вопроса из секции "Переход":

```html
<div class="reflection-card reveal">
  <div class="reflection-row">
    <div class="reflection-badge"><span>N</span></div>
    <p class="reflection-text">{{QUESTION_TEXT}}</p>
  </div>
</div>
```

### `{{DATE_FOOTER}}`
Если дата есть: ` · 2026-05-29`. Если нет — пустая строка.

### `{{TOPIC}}`
Краткая тема. Извлекается из eyebrow / контекста. Если не удалось определить — `Анализ методом почемучка`.

## Границы и ограничения

- **Никаких внешних зависимостей.** Ни Tailwind CDN, ни Google Fonts, ни внешние SVG, ни иконочные шрифты. Всё inline.
- **Никаких изображений.** Если пользователь хочет иллюстрации — предупреди, что навык их не генерирует.
- **Follow-up секция опускается** ("Хочешь, запустим почемучку глубше...") — по умолчанию не включается.
- **Секция "Переход" опускается**, если в markdown её нет.
- **Не давай пошаговых инструкций в навыке** — навык просто рендерит, он не анализирует.
- **Если markdown не соответствует формату pochemuchka** — сообщи пользователю, что формат не распознан, и покажи пример ожидаемой структуры.
- **Не используй эмодзи** в HTML и SKILL.md.

## Пример вызова

**Пользователь:**
```
Вот результат почемучки:

# Кейс: Экосистема вокруг Service Desk
**Тезис:** Мы тратим ресурсы...
## Цепочка "Почему?"
**Почему 1:** ... → ...
...
## Корень
...
## Линза: ...
| Было | Стало | Для человека |
...
## Переход
- ...
- ...

Оформи как HTML страницу
```

**Навык:**
1. Парсит markdown
2. Генерирует `service-desk-ecosystem-5whys.html` (или запрашивает имя)
3. Пишет файл в рабочую директорию
4. Сообщает: "Готово: `/path/to/service-desk-ecosystem-5whys.html`. Страница полностью автономна — открой в браузере. Есть переключение светлой/тёмной темы, сохраняется в localStorage."

## Пример структуры ответа навыка

```
Генерирую HTML-страницу из вашего markdown...

✅ Сохранено: `/Users/.../pochemuchka-result.html`

Что внутри:
- Полностью автономный файл (нет CDN, нет зависимостей)
- Переключение светлой/тёмной темы (кнопка в правом верхнем углу)
- Timeline с нумерацией "Почему?"
- Карточка "Корень"
- Таблица "Линза"
- Вопросы для размышления
- Scroll-reveal анимация (соблюдает prefers-reduced-motion)

Откройте файл в браузере.
```

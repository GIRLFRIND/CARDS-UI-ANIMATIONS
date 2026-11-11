# Northstar Stories

## A classroom-style guide to the project

Welcome to this small web project. It is a customer-stories page with four
full-frame portrait cards. At the start, the cards sit in one stack. As the
visitor scrolls, each card already seen moves upward and the next card is
revealed. There is also a simple About Me page.

This README is both the lesson and a copy of the complete source code. You can
read it from top to bottom like a lecture, then copy either code listing into
the matching file:

1. Copy the first listing into `index.html`.
2. Copy the second listing into `about.html`.
3. Open `index.html` in a browser and scroll down through the stories.

The runnable files are kept separately too. That way, the browser opens the
HTML files directly, while this README teaches how they work.

## Lesson 1: The three languages in this project

Think of a web page like a stage production:

- **HTML** is the stage and its actors. It defines the header, story cards,
  headings, links, and page sections.
- **CSS** is the set, costumes, and lighting. It controls the layout, colors,
  portrait backgrounds, card stack, and responsive presentation.
- **JavaScript** is the stage manager. It reads the scroll position and tells
  the cards how far to move.

The three work together, but each has a clear job. Start by finding the four
`article.story-card` elements in the HTML. Then find the `.story-card` rules in
the CSS. Finally, find `updateCards()` in the JavaScript.

## Lesson 2: Put four cards in one frame

Every story card is an `<article>` with a `data-index` from `0` to `3`. The
index is the card's position in the sequence. Each card has a name, a short
story, and a link.

The frame, `.story-stage`, is `position: sticky`. It stays visible while the
much taller `.stories` section scrolls past it. Each card is
`position: absolute` and fills that same frame with `inset: 0`. This is the
trick that makes four different cards occupy one visual space.

CSS paints the card stack in layers. JavaScript sets `z-index` so the card
nearest the current scroll progress is on top. Before scrolling, the cards get
small 16-pixel offsets. Those little edges tell the visitor that more cards are
waiting underneath.

## Lesson 3: Turn page scrolling into a number

The JavaScript needs one simple measurement: how far has the visitor moved
through the story section?

First it finds the section's document position and the height of the pinned
frame. The section's height minus the frame's height is the available scroll
distance, called `travel` in the code. Then the code turns the scroll distance
into `progress`:

```text
progress = (scroll position within story section / available travel) * last card index
```

For four cards, `cards.length - 1` is `3`. So progress goes smoothly from `0`
to `3`. It can be a decimal, such as `1.4`, while a card is in motion.

## Lesson 4: Move old cards up and leave future cards stacked

The animation has two cases:

1. **This card is still ahead.** Its offset is the number of cards ahead times
   the small stack gap.
2. **This card is behind the current progress.** It moves upward by the
   distance passed times the frame height.

JavaScript puts the result in a CSS custom property named `--offset`. CSS reads
that value in `translate3d(0, var(--offset), 0)`. A transform moves a card
without changing the layout of its neighbors. This kind of layered movement is
often called a **scroll-linked stacked-card animation**.

The movement is continuous: as the visitor scrolls a little farther, the
offset changes a little. When one card has moved away, the next card is already
underneath it.

## Lesson 5: Keep the animation considerate

There are a few small details that make the animation more robust:

- A passive scroll listener tells the browser the handler will not block
  scrolling.
- `requestAnimationFrame()` batches visual updates for the browser's next
  paint instead of repeatedly doing the same work during one frame.
- The `prefers-reduced-motion` media preference avoids sliding cards for
  visitors who have asked their device for less motion.
- CSS media queries reduce spacing and adjust the layout on narrow screens.

## Lesson 6: Run and customize the project

No packages or build step are needed. Keep `index.html`, `about.html`, and
`README.md` in the same folder, then open `index.html` in a browser. The story
portraits are loaded from Unsplash, so an internet connection is needed to see
them.

To customize the project:

- Change a person's name and story in the matching `<article>` in
  `index.html`.
- Change a card portrait by replacing that card's `background-image` URL.
- Update the sample biography and `[Your name]` placeholder in `about.html`.
- If you add or remove cards, the JavaScript reads `cards.length`
  automatically. Keep each card's `data-index` in order, and consider
  adjusting `.stories` height so there is enough scroll travel.

## Complete source code: `index.html`

Copy this entire block into the file named `index.html`.

````html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#f7f8fa">
  <title>Cloud Stories — Built for what’s next</title>
  <style>
    :root {
      color-scheme: light;
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: #17212b;
      background: #f7f8fa;
      font-synthesis: none;
      text-rendering: optimizeLegibility;
      --ink: #17212b;
      --muted: #65717d;
      --accent: #ff9a45;
      --nav-height: 76px;
    }

    * { box-sizing: border-box; }

    html { scroll-behavior: smooth; }

    body {
      margin: 0;
      min-width: 320px;
      background: #f7f8fa;
    }

    a { color: inherit; text-decoration: none; }

    .site-header {
      position: sticky;
      z-index: 20;
      top: 0;
      height: var(--nav-height);
      border-bottom: 1px solid #e4e8ec;
      background: rgb(255 255 255 / 94%);
      backdrop-filter: blur(14px);
    }

    .nav {
      width: min(1120px, calc(100% - 48px));
      height: 100%;
      margin: 0 auto;
      display: flex;
      align-items: center;
      gap: 34px;
    }

    .brand {
      display: inline-flex;
      align-items: center;
      gap: 9px;
      flex: 0 0 auto;
      font-size: 19px;
      font-weight: 780;
      letter-spacing: -0.8px;
    }

    .brand-mark {
      display: inline-grid;
      width: 31px;
      height: 31px;
      place-items: center;
      border-radius: 10px;
      color: white;
      background: #17212b;
      font-size: 15px;
      font-weight: 800;
    }

    .nav-links {
      display: flex;
      align-items: center;
      gap: 29px;
      margin-left: 2px;
      color: #46515c;
      font-size: 13px;
      font-weight: 550;
    }

    .nav-links a, .nav-actions a { transition: color 160ms ease; }
    .nav-links a:hover, .nav-actions a:hover { color: #d66b19; }

    .nav-actions {
      display: flex;
      align-items: center;
      gap: 21px;
      margin-left: auto;
      font-size: 13px;
      font-weight: 620;
    }

    .nav-cta {
      padding: 11px 17px;
      border-radius: 999px;
      color: white;
      background: var(--ink);
    }

    .nav-cta:hover { color: white !important; background: #304050; }

    main { overflow: clip; }

    .intro {
      width: min(1120px, calc(100% - 48px));
      margin: 0 auto;
      padding: clamp(70px, 10vw, 128px) 0 46px;
    }

    .eyebrow {
      display: flex;
      align-items: center;
      gap: 10px;
      margin: 0 0 20px;
      color: #697582;
      font-size: 12px;
      font-weight: 750;
      letter-spacing: 1.5px;
      text-transform: uppercase;
    }

    .eyebrow::before {
      width: 25px;
      height: 2px;
      background: var(--accent);
      content: "";
    }

    h1 {
      max-width: 760px;
      margin: 0;
      font-size: clamp(42px, 6.4vw, 76px);
      font-weight: 650;
      letter-spacing: -4px;
      line-height: 0.99;
    }

    .intro-copy {
      max-width: 570px;
      margin: 23px 0 0;
      color: var(--muted);
      font-size: 17px;
      line-height: 1.7;
    }

    .stories {
      position: relative;
      /* Extra section height creates the scroll distance for the pinned card animation. */
      height: 260vh;
      min-height: 1900px;
      padding: 20px 24px 0;
    }

    .story-stage {
      /* Keep one frame in place while the story section scrolls behind it. */
      position: sticky;
      top: calc(var(--nav-height) + 32px);
      width: min(900px, 100%);
      height: min(470px, calc(100vh - var(--nav-height) - 70px));
      min-height: 360px;
      margin: 0 auto;
      perspective: 1200px;
    }

    .story-card {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: flex-end;
      overflow: hidden;
      border: 1px solid rgb(255 255 255 / 70%);
      border-radius: 20px;
      background-color: #203448;
      background-position: center;
      background-size: cover;
      box-shadow: 0 16px 38px rgb(22 35 48 / 22%), 0 2px 6px rgb(22 35 48 / 12%);
      /* JavaScript changes --offset; translate3d moves each card without changing the layout. */
      transform: translate3d(0, var(--offset, 0px), 0);
      will-change: transform;
      isolation: isolate;
    }

    .story-card::before {
      position: absolute;
      z-index: -1;
      inset: 0;
      background: linear-gradient(90deg, rgb(9 17 26 / 72%) 0%, rgb(9 17 26 / 30%) 55%, rgb(9 17 26 / 6%) 100%),
        linear-gradient(0deg, rgb(7 15 23 / 48%), transparent 62%);
      content: "";
    }

    .story-card::after {
      position: absolute;
      z-index: -1;
      inset: 0;
      border-radius: inherit;
      box-shadow: inset 0 0 0 1px rgb(255 255 255 / 12%);
      content: "";
      pointer-events: none;
    }

    .story-card[data-index="0"] {
      background-image: url("https://images.unsplash.com/photo-1500648767791-00dcc994a43e?auto=format&fit=crop&crop=faces&w=1800&q=85");
    }

    .story-card[data-index="1"] {
      background-image: url("https://images.unsplash.com/photo-1534528741775-53994a69daeb?auto=format&fit=crop&crop=faces&w=1800&q=85");
    }

    .story-card[data-index="2"] {
      background-image: url("https://images.unsplash.com/photo-1506794778202-cad84cf45f1d?auto=format&fit=crop&crop=faces&w=1800&q=85");
    }

    .story-card[data-index="3"] {
      background-image: url("https://images.unsplash.com/photo-1531123897727-8f129e1688ce?auto=format&fit=crop&crop=faces&w=1800&q=85");
    }

    .card-content {
      width: min(660px, 100%);
      padding: 42px 50px 48px;
      color: white;
      text-shadow: 0 2px 12px rgb(0 0 0 / 20%);
    }

    .company {
      display: block;
      margin-bottom: 17px;
      font-size: clamp(20px, 2.2vw, 27px);
      font-weight: 770;
      letter-spacing: -0.7px;
    }

    .card-content h2 {
      max-width: 620px;
      margin: 0;
      font-size: clamp(19px, 2vw, 24px);
      font-weight: 620;
      letter-spacing: -0.45px;
      line-height: 1.4;
    }

    .card-link {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      margin-top: 21px;
      font-size: 24px;
      line-height: 1;
      transition: gap 160ms ease;
    }

    .card-link:hover { gap: 17px; }

    .scroll-note {
      position: absolute;
      right: max(calc((100vw - 900px) / 2), 24px);
      bottom: 5px;
      display: flex;
      align-items: center;
      gap: 9px;
      color: #6b7783;
      font-size: 11px;
      font-weight: 700;
      letter-spacing: 1.1px;
      text-transform: uppercase;
    }

    .scroll-note span {
      display: inline-block;
      width: 29px;
      height: 1px;
      background: #a7b0b9;
    }

    .closing {
      width: min(1120px, calc(100% - 48px));
      margin: 0 auto;
      padding: 80px 0 110px;
      border-top: 1px solid #e2e6ea;
    }

    .closing p {
      max-width: 740px;
      margin: 0;
      font-size: clamp(27px, 4vw, 43px);
      font-weight: 600;
      letter-spacing: -1.7px;
      line-height: 1.2;
    }

    .closing a {
      display: inline-flex;
      margin-top: 24px;
      padding: 13px 19px;
      border-radius: 999px;
      color: #fff;
      background: var(--ink);
      font-size: 14px;
      font-weight: 650;
    }

    .closing .about-link {
      margin-left: 10px;
      border: 1px solid #dce1e6;
      color: var(--ink);
      background: white;
    }

    @media (max-width: 720px) {
      :root { --nav-height: 66px; }
      .nav { width: calc(100% - 32px); gap: 18px; }
      .nav-links { display: none; }
      .nav-actions { gap: 12px; font-size: 12px; }
      .nav-actions > a:first-child { display: none; }
      .nav-cta { padding: 10px 13px; }
      .intro, .closing { width: calc(100% - 40px); }
      .intro { padding-top: 76px; }
      h1 { letter-spacing: -2.6px; }
      .intro-copy { font-size: 15px; }
      .stories {
        height: 220vh;
        min-height: 1500px;
        padding: 12px 16px 0;
      }
      .story-stage {
        top: calc(var(--nav-height) + 24px);
        height: min(470px, calc(100vh - var(--nav-height) - 90px));
        min-height: 330px;
      }
      .story-card { border-radius: 15px; }
      .card-content { padding: 30px 25px 32px; }
      .card-content h2 { font-size: 19px; }
      .scroll-note { right: 20px; }
    }

    @media (prefers-reduced-motion: reduce) {
      html { scroll-behavior: auto; }
      .story-card { will-change: auto; }
    }
  </style>
</head>
<body>
  <header class="site-header">
    <nav class="nav" aria-label="Main navigation">
      <a class="brand" href="#" aria-label="Northstar home">
        <span class="brand-mark" aria-hidden="true">N</span>
        northstar
      </a>
      <div class="nav-links">
        <a href="#stories">Discover</a>
        <a href="#stories">Our platform</a>
        <a href="#stories">Solutions</a>
        <a href="#stories">Resources</a>
        <a href="about.html">About me</a>
      </div>
      <div class="nav-actions">
        <a href="#stories">Explore stories</a>
        <a class="nav-cta" href="#contact">Talk to our team</a>
      </div>
    </nav>
  </header>

  <main>
    <section class="intro" id="stories">
      <p class="eyebrow">Ideas in motion</p>
      <h1>Big ideas find room to grow.</h1>
      <p class="intro-copy">See how ambitious teams are using cloud technology to make everyday experiences feel extraordinary.</p>
    </section>

    <section class="stories" aria-label="Customer stories">
      <div class="story-stage">
        <article class="story-card" data-index="0" aria-label="Alex Morgan's story">
          <div class="card-content">
            <span class="company">Alex Morgan</span>
            <h2>Meet the designer creating more accessible digital experiences for everyone.</h2>
            <a class="card-link" href="#contact" aria-label="Read Alex Morgan's story">→</a>
          </div>
        </article>
        <article class="story-card" data-index="1" aria-label="Maya Chen's story">
          <div class="card-content">
            <span class="company">Maya Chen</span>
            <h2>Meet the founder bringing better health resources to neighborhoods that need them.</h2>
            <a class="card-link" href="#contact" aria-label="Read Maya Chen's story">→</a>
          </div>
        </article>
        <article class="story-card" data-index="2" aria-label="Jordan Lee's story">
          <div class="card-content">
            <span class="company">Jordan Lee</span>
            <h2>Meet the educator opening new doors for the next generation of young creators.</h2>
            <a class="card-link" href="#contact" aria-label="Read Jordan Lee's story">→</a>
          </div>
        </article>
        <article class="story-card" data-index="3" aria-label="Amina Yusuf's story">
          <div class="card-content">
            <span class="company">Amina Yusuf</span>
            <h2>Meet the photographer using her craft to help overlooked stories find an audience.</h2>
            <a class="card-link" href="#contact" aria-label="Read Amina Yusuf's story">→</a>
          </div>
        </article>
      </div>
      <div class="scroll-note" aria-hidden="true"><span></span>Scroll to explore</div>
    </section>

    <section class="closing" id="contact">
      <p>Your next big idea deserves a place to take off.</p>
      <a href="mailto:hello@northstar.example">Let’s build something</a>
      <a class="about-link" href="about.html">About me</a>
    </section>
  </main>

  <script>
    (() => {
      const section = document.querySelector(".stories");
      const cards = [...document.querySelectorAll(".story-card")];
      const reducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)");

      function updateCards() {
        if (reducedMotion.matches) {
          // Keep the cards stacked when the visitor asks for less motion.
          cards.forEach((card, index) => {
            card.style.setProperty("--offset", `${index * 16}px`);
            card.style.zIndex = String(cards.length - index);
          });
          return;
        }

        const sectionTop = section.getBoundingClientRect().top + window.scrollY;
        const stage = section.querySelector(".story-stage");
        const stageHeight = stage.offsetHeight;
        const travel = Math.max(1, section.offsetHeight - stageHeight);
        // Convert this section's scroll distance into a value from 0 to the last card index.
        const progress = Math.max(0, Math.min(cards.length - 1, (window.scrollY - sectionTop) / travel * (cards.length - 1)));
        const stackOffset = 16;

        cards.forEach((card, index) => {
          // Cards ahead stay slightly staggered; each card behind progress slides up one frame.
          const offset = progress <= index
            ? (index - progress) * stackOffset
            : -(progress - index) * stageHeight;
          card.style.setProperty("--offset", `${offset.toFixed(1)}px`);
          card.style.zIndex = String(100 - Math.round(Math.abs(index - progress) * 10));
        });
      }

      let scheduled = false;
      function scheduleUpdate() {
        if (scheduled) return;
        scheduled = true;
        // Batch scroll events into one animation-frame update for smoother rendering.
        requestAnimationFrame(() => {
          updateCards();
          scheduled = false;
        });
      }

      window.addEventListener("scroll", scheduleUpdate, { passive: true });
      window.addEventListener("resize", scheduleUpdate);
      reducedMotion.addEventListener("change", scheduleUpdate);
      updateCards();
    })();
  </script>
</body>
</html>
````

## Complete source code: `about.html`

Copy this entire block into the file named `about.html`.

````html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#f7f8fa">
  <title>About Me — Northstar</title>
  <style>
    :root {
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: #17212b;
      background: #f7f8fa;
      --ink: #17212b;
      --muted: #65717d;
      --accent: #ff9a45;
      --nav-height: 76px;
    }

    * { box-sizing: border-box; }
    html { scroll-behavior: smooth; }
    body { margin: 0; min-width: 320px; }
    a { color: inherit; text-decoration: none; }

    .site-header {
      position: sticky;
      z-index: 2;
      top: 0;
      height: var(--nav-height);
      border-bottom: 1px solid #e4e8ec;
      background: rgb(255 255 255 / 94%);
      backdrop-filter: blur(14px);
    }

    .nav {
      display: flex;
      align-items: center;
      gap: 30px;
      width: min(1120px, calc(100% - 48px));
      height: 100%;
      margin: 0 auto;
    }

    .brand {
      display: inline-flex;
      align-items: center;
      gap: 9px;
      font-size: 19px;
      font-weight: 780;
      letter-spacing: -0.8px;
    }

    .brand-mark {
      display: inline-grid;
      width: 31px;
      height: 31px;
      place-items: center;
      border-radius: 10px;
      color: white;
      background: var(--ink);
      font-size: 15px;
      font-weight: 800;
    }

    .nav a:last-child {
      margin-left: auto;
      color: #d66b19;
      font-size: 13px;
      font-weight: 650;
    }

    main {
      width: min(900px, calc(100% - 48px));
      margin: 0 auto;
    }

    .intro { padding: clamp(72px, 11vw, 130px) 0 64px; }

    .eyebrow {
      margin: 0 0 18px;
      color: #697582;
      font-size: 12px;
      font-weight: 750;
      letter-spacing: 1.5px;
      text-transform: uppercase;
    }

    h1 {
      max-width: 780px;
      margin: 0;
      font-size: clamp(44px, 8vw, 78px);
      font-weight: 650;
      letter-spacing: -4px;
      line-height: 1;
    }

    .intro-copy {
      max-width: 650px;
      margin: 24px 0 0;
      color: var(--muted);
      font-size: 18px;
      line-height: 1.7;
    }

    .lesson {
      margin-bottom: 72px;
      padding: clamp(25px, 5vw, 48px);
      border: 1px solid #e4e8ec;
      border-radius: 20px;
      background: white;
      box-shadow: 0 14px 36px rgb(22 35 48 / 6%);
    }

    .lesson h2 {
      margin: 0 0 18px;
      font-size: clamp(25px, 4vw, 36px);
      font-weight: 650;
      letter-spacing: -1.2px;
    }

    .lesson h3 {
      margin: 28px 0 9px;
      font-size: 17px;
    }

    .lesson p, .lesson li {
      color: #53606d;
      font-size: 15px;
      line-height: 1.75;
    }

    .lesson ul { padding-left: 21px; }
    .lesson li + li { margin-top: 7px; }

    pre {
      overflow-x: auto;
      margin: 19px 0 0;
      padding: 20px;
      border-radius: 12px;
      color: #eff5fb;
      background: #17212b;
      font: 13px/1.75 ui-monospace, SFMono-Regular, Consolas, monospace;
    }

    code { font-family: ui-monospace, SFMono-Regular, Consolas, monospace; }

    .back-link {
      display: inline-flex;
      margin: 0 0 90px;
      padding: 13px 18px;
      border-radius: 999px;
      color: white;
      background: var(--ink);
      font-size: 14px;
      font-weight: 650;
    }

    @media (max-width: 720px) {
      :root { --nav-height: 66px; }
      .nav, main { width: calc(100% - 36px); }
      h1 { letter-spacing: -2.5px; }
      .intro-copy { font-size: 16px; }
      .lesson { margin-bottom: 45px; }
    }
  </style>
</head>
<body>
  <header class="site-header">
    <nav class="nav" aria-label="Main navigation">
      <a class="brand" href="index.html">
        <span class="brand-mark" aria-hidden="true">N</span>
        northstar
      </a>
      <a href="index.html">Back to stories</a>
    </nav>
  </header>

  <main>
    <section class="intro">
      <p class="eyebrow">A little about me</p>
      <h1>Hello, I’m [Your name].</h1>
      <p class="intro-copy">I’m learning to build thoughtful web experiences. This page is a starter bio: replace this sentence with a few lines about your interests, the work you enjoy, and what you’re learning.</p>
    </section>

    <article class="lesson">
      <h2>How the scrolling card animation works</h2>
      <p>The four story cards share one frame. At the start they are stacked, with a small offset so you can see the cards behind the top one. As you scroll through the tall story section, each card in turn slides up and the next card is revealed.</p>

      <h3>1. Make a scroll runway</h3>
      <p>The <code>.stories</code> section is taller than the viewport. That extra height gives the animation time to progress while the card frame remains pinned.</p>

      <h3>2. Pin one frame and stack the cards</h3>
      <p><code>.story-stage</code> uses <code>position: sticky</code> to stay in view. Each <code>.story-card</code> uses <code>position: absolute</code> to occupy the same frame. The cards are layered by <code>z-index</code>; a small vertical offset hints that more cards are underneath.</p>

      <h3>3. Turn scroll position into animation progress</h3>
      <p>JavaScript measures how far the page has moved through the story section. It maps that distance to a number from <code>0</code> to <code>3</code> (the four cards have indexes 0 through 3). The current number identifies which card is being shown.</p>

      <h3>4. Move cards with a CSS custom property</h3>
      <p>For cards already passed, JavaScript updates <code>--offset</code> to slide them upward by one frame. Cards still ahead keep their small stacked offset. CSS applies the value with <code>translate3d</code>, so moving a card doesn’t reflow the others.</p>

      <h3>5. Keep updates smooth and accessible</h3>
      <ul>
        <li><code>requestAnimationFrame</code> batches scroll updates to the browser’s next paint.</li>
        <li>The <code>prefers-reduced-motion</code> setting keeps the cards stacked without the scrolling movement.</li>
        <li>The CSS media query adjusts the frame and spacing for smaller screens.</li>
      </ul>

      <h3>The core idea, simplified</h3>
      <pre><code>progress = scrollDistance / scrollableDistance * lastCardIndex
offset = cardIsPassed ? -distancePassed * frameHeight : cardsAhead * stackGap</code></pre>
    </article>

    <a class="back-link" href="index.html">← Back to the story cards</a>
  </main>
</body>
</html>
````

## Final reminder

The source listings above are copies for learning and reference. Make actual
website changes in `index.html` or `about.html`; a browser does not execute
HTML code merely because it appears in a Markdown README.

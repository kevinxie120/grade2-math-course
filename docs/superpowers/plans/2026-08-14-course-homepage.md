# Course Homepage Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the GitHub Pages root page with an accessible, responsive catalog that links to all five grade-two math units.

**Architecture:** Keep `index.html` as one dependency-free static document with semantic HTML and embedded CSS. Keep all course pages unchanged and use direct relative `<a>` links. Store automated tests in the surrounding course workspace, matching the existing unit-test layout, while committing the deployable homepage and documentation to the nested Pages repository.

**Tech Stack:** HTML5, CSS3, Node.js built-in test runner, Playwright with bundled Chrome, GitHub Pages, GitHub CLI.

## Global Constraints

- Do not modify `grade2-math-unit1.html` through `grade2-math-unit5.html`.
- Do not read, write, or remove `localStorage` from the homepage.
- Do not add runtime JavaScript, frameworks, external fonts, images, scripts, stylesheets, APIs, or network dependencies.
- Use exact relative destinations `grade2-math-unit1.html` through `grade2-math-unit5.html`.
- Provide five always-available course links; do not gate or reorder units based on progress.
- Use semantic landmarks, a skip link, descriptive link names, visible keyboard focus, and minimum 48-pixel primary targets.
- Use a three-column desktop grid, two-column tablet grid, and one-column mobile grid with no horizontal overflow at 320 pixels.
- Honor `prefers-reduced-motion: reduce`.
- Preserve the existing GitHub Pages source: `main` branch, repository root, HTTPS enforced.

---

### Task 1: Define the Homepage Contract with Failing Tests

**Files:**
- Create: `../tests/check-grade2-math-homepage.mjs`
- Create: `../tests/browser-grade2-math-homepage.mjs`
- Read: `index.html`
- Read: `grade2-math-unit1.html`
- Read: `grade2-math-unit2.html`
- Read: `grade2-math-unit3.html`
- Read: `grade2-math-unit4.html`
- Read: `grade2-math-unit5.html`

**Interfaces:**
- Consumes: the publication directory at `../github-pages-grade2-math-course` relative to the workspace `tests` directory.
- Produces: an exact structural contract for five relative links and a browser acceptance contract for navigation, accessibility, responsiveness, and offline behavior.

- [ ] **Step 1: Write the structural test**

Create `../tests/check-grade2-math-homepage.mjs` with this contract:

```js
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const here = path.dirname(fileURLToPath(import.meta.url));
const siteRoot = path.join(here, '..', 'github-pages-grade2-math-course');
const htmlPath = path.join(siteRoot, 'index.html');
const units = [
  ['1', '数学星球大冒险', 'grade2-math-unit1.html'],
  ['2', '计算城建造师', 'grade2-math-unit2.html'],
  ['3', '小小店长', 'grade2-math-unit3.html'],
  ['4', '乘法乐园建筑师', 'grade2-math-unit4.html'],
  ['5', '口诀森林探险队', 'grade2-math-unit5.html']
];

function readHtml() {
  assert.equal(fs.existsSync(htmlPath), true, '课程目录 index.html 应当存在');
  return fs.readFileSync(htmlPath, 'utf8');
}

test('首页提供五个且仅五个正确的单元链接', () => {
  const html = readHtml();
  for (const [number, title, href] of units) {
    assert.equal(fs.existsSync(path.join(siteRoot, href)), true, `${href} 应存在`);
    assert.equal((html.match(new RegExp(`href=["']${href}["']`, 'g')) || []).length, 1, `${href} 应恰好出现一次`);
    assert.match(html, new RegExp(`第${number}单元[\\s\\S]*${title}`), `第${number}单元名称应完整`);
  }
  assert.equal((html.match(/class=["'][^"']*course-link[^"']*["']/g) || []).length, 5);
});

test('首页语义、离线和无障碍约束完整', () => {
  const html = readHtml();
  for (const tag of ['header', 'main', 'section', 'ul', 'li', 'footer']) assert.match(html, new RegExp(`<${tag}\\b`, 'i'));
  assert.match(html, /href=["']#course-list["']/);
  assert.match(html, /id=["']course-list["']/);
  assert.match(html, /约 60 分钟/);
  assert.match(html, /学习进度只保存在当前浏览器/);
  assert.match(html, /min-height:\s*(?:48px|3rem)/);
  assert.match(html, /prefers-reduced-motion:\s*reduce/);
  assert.match(html, /@media\s*\([^)]*max-width:\s*900px/);
  assert.match(html, /@media\s*\([^)]*max-width:\s*640px/);
  assert.doesNotMatch(html, /<script\b|<link[^>]+href=|<img\b|https?:\/\/|localStorage/);
});
```

- [ ] **Step 2: Write the browser acceptance test**

Create `../tests/browser-grade2-math-homepage.mjs` with this behavior:

```js
import assert from 'node:assert/strict';
import fs from 'node:fs';
import path from 'node:path';
import { pathToFileURL, fileURLToPath } from 'node:url';
import { chromium } from '/Users/xieweikang/.cache/codex-runtimes/codex-primary-runtime/dependencies/node/node_modules/playwright/index.mjs';

const here = path.dirname(fileURLToPath(import.meta.url));
const workspaceRoot = path.join(here, '..');
const siteRoot = path.join(workspaceRoot, 'github-pages-grade2-math-course');
const pageUrl = pathToFileURL(path.join(siteRoot, 'index.html')).href;
const artifactsDir = path.join(workspaceRoot, 'artifacts');
fs.mkdirSync(artifactsDir, { recursive: true });

const units = [
  ['第一单元', '数学星球大冒险', 'grade2-math-unit1.html'],
  ['第二单元', '计算城建造师', 'grade2-math-unit2.html'],
  ['第三单元', '小小店长', 'grade2-math-unit3.html'],
  ['第四单元', '乘法乐园建筑师', 'grade2-math-unit4.html'],
  ['第五单元', '口诀森林探险队', 'grade2-math-unit5.html']
];

const browser = await chromium.launch({
  headless: true,
  executablePath: '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome'
});

try {
  const context = await browser.newContext({ viewport: { width: 1180, height: 900 }, reducedMotion: 'reduce' });
  const page = await context.newPage();
  const pageErrors = [];
  const externalRequests = [];
  page.on('pageerror', (error) => pageErrors.push(error.message));
  page.on('request', (request) => {
    const protocol = new URL(request.url()).protocol;
    if (!['file:', 'data:', 'blob:'].includes(protocol)) externalRequests.push(request.url());
  });

  await page.goto(pageUrl);
  await page.getByRole('heading', { name: '二年级数学互动自学课程' }).waitFor();
  assert.equal(await page.locator('.course-link').count(), 5);

  await page.keyboard.press('Tab');
  assert.equal(await page.evaluate(() => document.activeElement?.getAttribute('href')), '#course-list');
  for (const [unit, title] of units) {
    const link = page.getByRole('link', { name: new RegExp(`${unit}.*${title}`) });
    await link.focus();
    assert.equal(await link.evaluate((element) => element === document.activeElement), true);
  }

  for (const [unit, title, href] of units) {
    await page.goto(pageUrl);
    await page.getByRole('link', { name: new RegExp(`${unit}.*${title}`) }).click();
    await page.waitForURL(new RegExp(`${href.replace('.', '\\.')}$`));
    assert.match(await page.title(), new RegExp(title));
  }

  for (const [width, columns] of [[1180, 3], [768, 2], [320, 1]]) {
    await page.setViewportSize({ width, height: 900 });
    await page.goto(pageUrl);
    assert.equal(await page.evaluate(() => document.documentElement.scrollWidth <= window.innerWidth), true);
    const columnCount = await page.locator('.course-grid').evaluate((element) => getComputedStyle(element).gridTemplateColumns.split(' ').length);
    assert.equal(columnCount, columns);
  }

  const firstLinkBox = await page.locator('.course-link').first().boundingBox();
  assert.ok(firstLinkBox && firstLinkBox.height >= 48);
  await page.screenshot({ path: path.join(artifactsDir, 'grade2-math-homepage-mobile.png'), fullPage: true });
  assert.deepEqual(pageErrors, []);
  assert.deepEqual(externalRequests, []);
  await context.close();
  process.stdout.write('课程目录浏览器验收通过：五单元跳转、键盘、响应式与离线检查均正常。\n');
} finally {
  await browser.close();
}
```

- [ ] **Step 3: Run both tests and verify the old first-unit homepage fails the new contract**

Run:

```bash
node --test ../tests/check-grade2-math-homepage.mjs
node ../tests/browser-grade2-math-homepage.mjs
```

Expected: structural test fails because `index.html` is the first-unit course rather than a five-card directory; browser test fails because the directory heading and five `.course-link` elements do not exist.

---

### Task 2: Build the Dependency-Free Course Directory

**Files:**
- Modify: `index.html`
- Test: `../tests/check-grade2-math-homepage.mjs`
- Test: `../tests/browser-grade2-math-homepage.mjs`

**Interfaces:**
- Consumes: the five exact filenames defined by Task 1.
- Produces: `.course-grid` containing five `.course-link` anchors and `#course-list` as the skip-link target.

- [ ] **Step 1: Replace the first-unit copy with semantic homepage markup**

Use this exact content model inside `index.html`:

```html
<a class="skip-link" href="#course-list">跳到课程目录</a>
<header class="hero">
  <p class="eyebrow">二年级数学 · 互动自学</p>
  <h1>二年级数学互动自学课程</h1>
  <p class="lead">选择一个单元，带着好奇心出发。每次学习约 60 分钟，可以随时暂停再回来。</p>
</header>
<main>
  <section id="course-list" aria-labelledby="course-list-title">
    <div class="section-heading">
      <p class="section-kicker">课程目录</p>
      <h2 id="course-list-title">今天想去哪里学习？</h2>
    </div>
    <ul class="course-grid">
      <li class="course-card unit-1"><a class="course-link" href="grade2-math-unit1.html" aria-label="第一单元 数学星球大冒险，开始学习"><span class="unit-number">第一单元</span><span class="unit-icon" aria-hidden="true">🪐</span><h3>数学星球大冒险</h3><p>20以内加减、100以内数感与基础应用</p><span class="course-meta">约 60 分钟</span><span class="course-action">开始学习 <span aria-hidden="true">→</span></span></a></li>
      <li class="course-card unit-2"><a class="course-link" href="grade2-math-unit2.html" aria-label="第二单元 计算城建造师，开始学习"><span class="unit-number">第二单元</span><span class="unit-icon" aria-hidden="true">🏗️</span><h3>计算城建造师</h3><p>100以内加减法、进位、退位与验算</p><span class="course-meta">约 60 分钟</span><span class="course-action">开始学习 <span aria-hidden="true">→</span></span></a></li>
      <li class="course-card unit-3"><a class="course-link" href="grade2-math-unit3.html" aria-label="第三单元 小小店长，开始学习"><span class="unit-number">第三单元</span><span class="unit-icon" aria-hidden="true">🛍️</span><h3>小小店长</h3><p>人民币认识、付款、找零与预算</p><span class="course-meta">约 60 分钟</span><span class="course-action">开始学习 <span aria-hidden="true">→</span></span></a></li>
      <li class="course-card unit-4"><a class="course-link" href="grade2-math-unit4.html" aria-label="第四单元 乘法乐园建筑师，开始学习"><span class="unit-number">第四单元</span><span class="unit-icon" aria-hidden="true">🧱</span><h3>乘法乐园建筑师</h3><p>相同加数、乘法意义、阵列与生活模型</p><span class="course-meta">约 60 分钟</span><span class="course-action">开始学习 <span aria-hidden="true">→</span></span></a></li>
      <li class="course-card unit-5"><a class="course-link" href="grade2-math-unit5.html" aria-label="第五单元 口诀森林探险队，开始学习"><span class="unit-number">第五单元</span><span class="unit-icon" aria-hidden="true">🌲</span><h3>口诀森林探险队</h3><p>2至9乘法口诀与灵活计算策略</p><span class="course-meta">约 60 分钟</span><span class="course-action">开始学习 <span aria-hidden="true">→</span></span></a></li>
    </ul>
  </section>
</main>
<footer><p>学习进度只保存在当前浏览器，不会上传，也不会跨设备同步。</p></footer>
```

- [ ] **Step 2: Add the complete visual and responsive CSS contract**

Define these required selectors and values in the embedded `<style>`:

```css
:root { color-scheme: light; --ink: #26304a; --muted: #5e6880; --focus: #312e81; }
* { box-sizing: border-box; }
html { scroll-behavior: smooth; }
body { margin: 0; min-width: 0; min-height: 100vh; font: 18px/1.6 ui-rounded, "PingFang SC", "Microsoft YaHei", system-ui, sans-serif; color: var(--ink); background: linear-gradient(145deg, #eef7ff, #f6f0ff 48%, #fff8e8); }
.skip-link { position: fixed; top: 12px; left: 12px; z-index: 10; padding: 10px 16px; border-radius: 12px; color: #fff; background: var(--focus); transform: translateY(-150%); }
.skip-link:focus { transform: translateY(0); }
.site-shell { width: min(1180px, calc(100% - 32px)); margin: 0 auto; padding: 48px 0 36px; }
.hero { max-width: 780px; margin: 0 auto 42px; text-align: center; }
.eyebrow, .section-kicker { margin: 0; color: #5b5bd6; font-weight: 900; letter-spacing: .08em; }
h1 { margin: 10px 0 14px; font-size: clamp(36px, 7vw, 68px); line-height: 1.08; }
.lead { margin: 0; color: var(--muted); font-size: clamp(18px, 2.3vw, 22px); }
.section-heading { margin-bottom: 22px; }
.section-heading h2 { margin: 4px 0 0; font-size: clamp(28px, 4vw, 40px); }
.course-grid { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 22px; margin: 0; padding: 0; list-style: none; }
.course-card { min-width: 0; border-radius: 28px; box-shadow: 0 18px 45px rgba(48, 61, 112, .14); }
.course-link { display: flex; min-height: 100%; padding: 26px; border: 3px solid transparent; border-radius: inherit; color: var(--ink); background: rgba(255,255,255,.94); text-decoration: none; flex-direction: column; transition: transform .2s ease, box-shadow .2s ease; }
.course-link:hover { transform: translateY(-5px); box-shadow: 0 22px 55px rgba(48, 61, 112, .2); }
.course-link:focus-visible { outline: 4px solid var(--focus); outline-offset: 4px; }
.unit-number, .course-meta { align-self: flex-start; padding: 5px 11px; border-radius: 999px; font-size: 15px; font-weight: 900; }
.unit-icon { margin: 18px 0 10px; font-size: 54px; line-height: 1; }
.course-card h3 { margin: 0; font-size: 26px; line-height: 1.25; }
.course-card p { margin: 10px 0 20px; color: var(--muted); }
.course-meta { margin-top: auto; background: rgba(255,255,255,.72); }
.course-action { display: inline-flex; min-height: 48px; margin-top: 18px; align-items: center; justify-content: space-between; font-weight: 900; }
.unit-1 { background: linear-gradient(135deg, #dcdcff, #f2efff); }
.unit-2 { background: linear-gradient(135deg, #dff4ff, #e8f7f4); }
.unit-3 { background: linear-gradient(135deg, #fff0cd, #fff8e8); }
.unit-4 { background: linear-gradient(135deg, #ffe0d8, #fff0e9); }
.unit-5 { background: linear-gradient(135deg, #dff4dd, #eef9e8); }
footer { margin-top: 34px; padding: 20px; border-radius: 20px; color: var(--muted); background: rgba(255,255,255,.7); text-align: center; }
footer p { margin: 0; }
@media (max-width: 900px) { .course-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
@media (max-width: 640px) { .site-shell { width: min(100% - 22px, 1180px); padding-top: 32px; } .hero { margin-bottom: 30px; } .course-grid { grid-template-columns: minmax(0, 1fr); gap: 16px; } .course-link { padding: 22px; } }
@media (prefers-reduced-motion: reduce) { html { scroll-behavior: auto; } *, *::before, *::after { scroll-behavior: auto !important; transition-duration: .01ms !important; animation-duration: .01ms !important; animation-iteration-count: 1 !important; } }
```

- [ ] **Step 3: Run the structural test**

Run: `node --test ../tests/check-grade2-math-homepage.mjs`

Expected: all homepage structure tests pass.

- [ ] **Step 4: Run the browser acceptance test**

Run: `node ../tests/browser-grade2-math-homepage.mjs`

Expected: `课程目录浏览器验收通过：五单元跳转、键盘、响应式与离线检查均正常。`

- [ ] **Step 5: Commit the homepage**

```bash
git add index.html
git commit -m "Build course directory homepage"
```

---

### Task 3: Document, Regress, and Publish the Homepage

**Files:**
- Modify: `README.md`
- Verify: `index.html`
- Verify: `grade2-math-unit1.html`
- Verify: `grade2-math-unit2.html`
- Verify: `grade2-math-unit3.html`
- Verify: `grade2-math-unit4.html`
- Verify: `grade2-math-unit5.html`
- Test: `../tests/check-grade2-math-homepage.mjs`
- Test: `../tests/browser-grade2-math-homepage.mjs`
- Test: `../tests/check-grade2-math-unit1.mjs` through `../tests/check-grade2-math-unit5.mjs`

**Interfaces:**
- Consumes: the completed homepage and unchanged course files.
- Produces: accurate maintenance documentation, a reviewed Pull Request, and a verified public Pages deployment.

- [ ] **Step 1: Update README homepage and maintenance instructions**

Change the page list entry to:

```markdown
- 课程目录首页：`index.html`
```

Remove `cp ../grade2-math-unit1.html index.html` from the update commands and replace the explanation below the command block with:

```markdown
`index.html` 是独立课程目录。更新单元课程时只复制对应的 `grade2-math-unitN.html`，不要用第一单元覆盖首页。
```

- [ ] **Step 2: Run all relevant structural tests**

```bash
node --test \
  ../tests/check-grade2-math-homepage.mjs \
  ../tests/check-grade2-math-unit1.mjs \
  ../tests/check-grade2-math-unit2.mjs \
  ../tests/check-grade2-math-unit3.mjs \
  ../tests/check-grade2-math-unit4.mjs \
  ../tests/check-grade2-math-unit5.mjs
```

Expected: zero failures.

- [ ] **Step 3: Run homepage and all five unit browser acceptances**

```bash
node ../tests/browser-grade2-math-homepage.mjs
node ../tests/browser-grade2-math-unit1.mjs
node ../tests/browser-grade2-math-unit2.mjs
node ../tests/browser-grade2-math-unit3.mjs
node ../tests/browser-grade2-math-unit4.mjs
node ../tests/browser-grade2-math-unit5.mjs
```

Expected: all six scripts exit 0 and print their acceptance-success summaries.

- [ ] **Step 4: Verify scope and commit documentation**

```bash
git diff --check
git status -sb
git diff -- README.md index.html
git add README.md docs/superpowers/plans/2026-08-14-course-homepage.md
git commit -m "Document course homepage maintenance"
```

- [ ] **Step 5: Push and open a Pull Request**

```bash
git push -u origin agent/course-homepage
gh pr create --repo kevinxie120/grade2-math-course --base main --head agent/course-homepage --title "Build course directory homepage" --body-file /tmp/grade2-math-homepage-pr.md
```

The PR body must list the homepage behavior, unchanged unit pages, structural-test count, six browser acceptance results, and responsive/offline verification.

- [ ] **Step 6: Merge and wait for Pages**

```bash
gh pr merge --repo kevinxie120/grade2-math-course --squash --subject "Build course directory homepage"
PAGES_RUN_ID=$(gh run list --repo kevinxie120/grade2-math-course --workflow pages-build-deployment --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$PAGES_RUN_ID" --repo kevinxie120/grade2-math-course --exit-status
```

Expected: the newest Pages workflow concludes `success`.

- [ ] **Step 7: Verify the public deployment**

```bash
curl -fsSL 'https://kevinxie120.github.io/grade2-math-course/?verify=course-homepage' -o /tmp/grade2-math-homepage-live.html
cmp index.html /tmp/grade2-math-homepage-live.html
for unit in 1 2 3 4 5; do
  curl -fsSI "https://kevinxie120.github.io/grade2-math-course/grade2-math-unit${unit}.html" | sed -n '1p'
done
gh api repos/kevinxie120/grade2-math-course/pages --jq '{html_url: .html_url, status: .status, https_enforced: .https_enforced, source: .source}'
```

Expected: `cmp` exits 0, all five course URLs return HTTP 200, Pages reports `status: built`, `https_enforced: true`, and source `main` `/`.

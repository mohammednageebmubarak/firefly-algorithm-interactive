# Developer Guide | دليل المطوّر

> **Goal | الهدف:** Explain how the interactive Firefly Algorithm lesson works, and how to extend it without breaking the existing bilingual/offline experience.
>
> **Project entry point:** [index.html](../index.html). This is an intentionally self-contained HTML document; there is no build system, server, library, or package manager.

## 1. Architecture | بنية المشروع

| Part | Implementation | Purpose |
| --- | --- | --- |
| HTML sections | `<section class="panel" id="…">` | Separates lesson content and interactive modules |
| CSS theme | `<style>`, `:root` variables | Controls colors, page layout and responsive behavior |
| JavaScript | Inline `<script>` at end of page | Updates canvases, outputs, simulation, quiz, and translations |
| Canvas visualizations | `<canvas>` | Draws graphs without dependencies |
| Arabic/English strings | `data-ar` / `data-en`, `quizBank` | Supports interface translation |
| Preferences | `localStorage['fa-lang']` | Remembers language when browser permits storage |

**العربية:** الملف `index.html` يجمع HTML وCSS وJavaScript. التعديلات على الواجهة تكون داخل وسوم `section`، والألوان في `:root`، وسلوك التفاعل في نهاية الملف ضمن `<script>`.

## 2. JavaScript function map | دليل الدوال

| Function / variable | Responsibility | Notes |
| --- | --- | --- |
| `$(id)` | Finds an element by id | All interactive elements must have unique ids |
| `fmt(n)` | Formats a number to 4 decimal places | Presentation only |
| `f(x)` | Defines the example objective `(x-3)^2` | Its global minimum is x=3 |
| `canvasSetup(canvas)` | Clears and paints canvas grid | Called before redrawing charts |
| `axes(ctx,w,h)` | Draws axes and ticks | Uses the example's [0,6] domain |
| `drawFunction()` | Objective function chart and x slider | Updates `xout`, `fout`, `dout` |
| `drawAttraction()` | Plot `exp(-gamma*r*r)` | Assumes initial attractiveness beta0=1 |
| `moveCalc()` | Calculate one firefly movement | Uses fixed positions 1 and 2; user controls alpha, gamma, epsilon |
| `reset()` | Reset simulation state | `init`, `pts`, `gen`, `hist`, `bestEver` |
| `step()` | Advance swarm one generation | Evaluates better reference positions; clamps updates to [0,6] |
| `drawSwarm()` | Draw current firefly positions | Displays iteration and best fitness |
| `drawHistory()` | Plot best-so-far fitness | Should remain non-increasing |
| `quizBank` | Stores AR/EN quiz questions and feedback | `a` is a zero-based answer index |
| `renderQuiz()` | Render selected language's quiz | Preserves selected option indexes |
| `switchLanguage(lang)` | Translate page and change RTL/LTR | Also remembers preferred language |

### Core equations | المعادلات الأساسية

```text
Objective:             f(x) = (x - 3)^2
Attractiveness:        beta(r) = beta0 * exp(-gamma * r^2)
Movement (canonical):  x_i' = x_i + beta(r) * (x_j - x_i) + alpha * epsilon
Single-step example:   x_new = 1 + exp(-gamma) + alpha * epsilon
```

**Important:** The animated simulation uses `beta0=1`, `gamma=0.3`, sinusoidal deterministic perturbations, and a hard [0,6] search bound. It is a **teaching model**, not a claim of a general-purpose research-grade Firefly optimizer. The `bestEver` variable preserves the best objective value seen, even if an individual firefly becomes worse.

**العربية:** تستخدم المحاكاة حركة اضطرابية جيبية قابلة لإعادة الإنتاج بدلًا من عشوائية حقيقية، وتثبّت بعض المعاملات لتسهيل الشرح. عند استخدامها في بحث علمي يلزم توثيق النسخة ومعايير المقارنة وتنفيذ تجارب متكررة.

## 3. Translating content | إضافة ترجمة

For ordinary static elements, provide both variants:

```html
<h3 data-ar="عنوان بالعربية" data-en="English heading">عنوان بالعربية</h3>
```

When `switchLanguage('en')` runs, it:
1. Sets `document.documentElement.lang = 'en'` and `dir = 'ltr'`.
2. Replaces text in nodes carrying both `data-ar` and `data-en`.
3. Updates `aria-pressed` for language buttons and canvas `aria-label` values.
4. Re-renders quiz labels and explanations using `quizBank.en`.
5. Saves `fa-lang` to `localStorage` when supported.

For quiz updates, edit **both** `quizBank.ar` and `quizBank.en`, keeping questions and answer indices aligned. The current renderer assumes **five** quiz questions for selection retention and reset; if you change the number, change those loops too.

**العربية:** أضف `data-ar` و`data-en` معًا لأي نص ثابت. الأسئلة تُترجم داخل `quizBank` ولا تُستخرج تلقائيًا من HTML. عند إضافة سؤال جديد، راجع حلقات الأسئلة المثبتة على العدد 5.

## 4. Extending interactive charts | تطوير المخططات

- Add an HTML `<canvas id="newCanvas" ...>` and an input element with a unique `id`.
- Write `drawNewCanvas()`, which reads slider values and redraws the entire canvas.
- Bind an `input` event to the slider; call the function once at page load.
- Use `canvasSetup` and `axes` where appropriate, but verify the axis scales: these helpers are currently tailored to a one-dimensional example.
- Add Arabic and English labels or accessibility descriptions.
- Avoid external CDNs if you want the offline single-file behavior to remain intact.

**العربية:** أضف مساحة رسم ومدخلات بمعرّفات فريدة؛ أنشئ دالة للرسم وأعد استدعاءها عند تغيير القيم. لا تفترض أن تدريج المحاور الحالي مناسب لأي دالة جديدة.

## 5. Modifying the optimization experiment | تعديل المحاكاة

The current 1-D simulator stores five fireflies in `pts` and updates them in `step()`. When changing the problem:
- Define its variable bounds and objective, then adapt `f`, graph limits, and clamping consistently.
- If using multidimensional solutions, replace scalar positions with vectors and use Euclidean distance in all dimensions.
- Separate algorithm parameters from display constants so inputs can control them.
- Set and document a random seed if you switch to stochastic noise; repeat experiments for robust evaluation.
- Decide whether movement should use a snapshot of the previous generation, in-place updates, or another FA variant, and describe that choice explicitly.

**العربية:** عند تغيير دالة الهدف لا يكفي تعديل `f(x)` وحدها: يجب تحديث حدود المجال والرسم والقيود وآلية تقييم الحلول. وللتجارب البحثية يلزم استخدام عدة تشغيلات عشوائية وتوثيق إعدادات المقارنة.

## 6. Safe editing checklist | قائمة فحص قبل النشر

1. Open `index.html` in a browser **without an internet connection**.
2. Toggle **العربية / English** and check both RTL and LTR layouts.
3. Change every slider and verify the number outputs and graphs update.
4. Run **Step**, **10 Steps**, and **Reset**; check that the history chart updates and that the best-ever value never increases.
5. Answer and grade the quiz in each language; clear the answers; switch language mid-quiz.
6. Check the browser developer console for JavaScript errors.
7. Open the GitHub Pages site after merging/updating `master`.

These are manual regression checks; no automated test suite is bundled at present.

## 7. Maintenance and deployment | الصيانة والنشر

- **GitHub Pages source:** `master` branch, repository root. GitHub serves `index.html` automatically.
- **Offline version:** download the same `index.html` and double-click it.
- **Changes that alter behavior:** test them before publishing and write descriptive commit messages.
- **Development preference:** keep comments explaining *why* each section exists, particularly equations and simulation update rules. If the file grows substantially, consider splitting the project into assets, but that would require a separate offline distribution strategy.

Live site: https://mohammednageebmubarak.github.io/firefly-algorithm-interactive/

Repository: https://github.com/mohammednageebmubarak/firefly-algorithm-interactive/

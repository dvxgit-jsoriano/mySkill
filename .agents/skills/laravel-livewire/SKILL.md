---
name: laravel-livewire
description: Guidelines and instructions for Laravel development using Livewire 4 Multi-File Components (MFC) and Tailwind CSS. Use whenever the user asks to build, scaffold, or edit any Laravel page, component, form, dashboard, or module, organize app structure into modules, style pages, add interactivity to a Blade view, define routes, or when a task might need a new composer/npm package. Enforces MFC (not the SFC default) with pages/component folder structure split by module, `Route::livewire()` as the default routing pattern for Livewire pages, mobile-first Tailwind styling, AlpineJS limited to simple UI interactivity only (logic-heavy behavior goes in Livewire), and a package-minimization policy requiring the user's confirmation before adding any package or starting non-trivial implementation work.
---

# Laravel Livewire 4 Multi-File Component (MFC) & Tailwind CSS Guidelines

This skill guides the implementation of features using **Laravel** (latest stable version), **Livewire 4**, **Tailwind CSS**, and **AlpineJS**, with a strict focus on using Multi-File Components (MFCs), mobile-first styling, minimal client-side scripting, and a package-minimization policy.

Apply these rules by default on every relevant task; don't ask permission to follow them, just follow them — except for the package/planning confirmation step in section 5, which always requires an explicit go-ahead first.

## Stack Summary

| Layer | Choice |
|---|---|
| Backend | PHP + Laravel (always latest stable version — check `composer.json` if unsure) |
| Components | Livewire **4**, Multi-File Components (MFC) only |
| Routing | `Route::livewire($uri, Component::class)` for all Livewire page routes |
| Styling | TailwindCSS, utility-first and **mobile-first** |
| Client interactivity | AlpineJS — simple UI only (`x-show`, `x-if`, `@click`, toggles/tabs/modals). Logic-heavy behavior → Livewire |
| Vanilla JS | Avoid — use Livewire/Alpine instead |
| Packages (composer/npm) | Minimize; always suggest and get confirmation before adding any |

---

## 1. Always Use Multi-File Components (MFC)

In Livewire 4, the default component structure is Single-File Components (SFCs), where PHP logic, Blade layout, and scripting reside in one `.blade.php` file. For this project, you **must always use Multi-File Components (MFCs)** to keep Blade files small, maintainable, and readable. SFC and MFC are both valid, current Livewire 4 options — this is a project preference, not an old-vs-new-version issue.

### Creating an MFC
To generate a new Multi-File Component, run:
```bash
php artisan make:livewire ComponentName --mfc
```

This creates a directory for the component with the following structure:
```
ComponentName/
├── index.php         # PHP Class (component controller logic)
├── index.blade.php   # Blade template (HTML layout)
├── index.js          # Component-specific client-side JavaScript (optional)
├── index.css         # Component-specific CSS (optional - avoid if possible)
```

### Module Structuring & Layouts
Organize pages, components, and layouts using a modular hierarchy, split into two top-level trees: `pages` for full-page components and `component` for reusable/child components — each grouped by module.

#### 1. Pages (Full-page components)
Use the `pages::` namespace prefix followed by the module and component name:
```bash
php artisan make:livewire pages::[module_name].[page_name] --mfc
```
Example:
```bash
php artisan make:livewire pages::module.dashboard --mfc
```
*Directory structure:*
```
resources/views/livewire/pages/
└── module/
    ├── dashboard/
    │   ├── index.php
    │   └── index.blade.php
    └── users/
        ├── index.php
        └── index.blade.php
```

#### 2. Reusable Components
Organize components within the relevant module directory:
```bash
php artisan make:livewire [module_name].[component_name] --mfc
```
Example:
```bash
php artisan make:livewire module.users.modal-create --mfc
```
*Directory structure:*
```
resources/views/livewire/component/
└── module/
    └── users/
        └── modal-create/
            ├── index.php
            └── index.blade.php
```

#### 3. Layouts
Always create Blade layouts directly in `resources/views/layouts` instead of `resources/views/components/layouts`.
- Store layout files directly in `resources/views/layouts/` (e.g., `resources/views/layouts/super-admin.blade.php`, `resources/views/layouts/app.blade.php`).
- Reference layout views in Livewire page components using `#[Layout('layouts.super-admin')]`.


### Component Structure Rules
- **No Inline Logic:** Do not write PHP logic or `<script>` tags inside `index.blade.php`. Keep logic inside `index.php`. Keep client-side interactivity to Alpine (see section 4) inline in the Blade file, or in `index.js` only for the rare cases vanilla JS is unavoidable.
- **Blade File Cleanliness:** Break down large pages or complex sections into smaller child Livewire components or Blade sub-views (use `@include`; do not use `<x-component>`).

---

## 2. Livewire 4 Best Practices and Syntax
Always write modern Livewire 4 code. Avoid obsolete Livewire 2/3 syntaxes.

### Key Livewire 4 Paradigms
- **Property Binding:** Use `wire:model` or `wire:model.live` (for real-time updates) for form inputs.
- **Properties as State:** Define public properties on the Class (`index.php`) to manage component state. Use reactive properties where appropriate.
- **Actions/Methods:** Public methods on the Class are invoked via user interactions using `wire:click`, `wire:submit`, `wire:keydown`, etc.
- **Lifecycle Hooks:** Use Livewire 4 lifecycle hooks (`mount`, `boot`, `updating`, `updated`, etc.) to run code at specific stages.
- **Wire Keying:** Always provide a unique `wire:key` on loop elements to ensure Livewire tracks DOM changes correctly:
  ```blade
  @foreach ($items as $item)
      <div wire:key="item-{{ $item->id }}">
          ...
      </div>
  @endforeach
  ```
- **Lazy Loading:** For performance-heavy components, leverage Livewire 4's lazy loading options by passing `#[Lazy]` or returning `lazy` views.

---

## 3. Routes — Use `Route::livewire()` for Livewire Pages

Livewire page components are registered as **the default routing behavior** using Livewire 4's `Route::livewire()` helper, not the generic `Route::get(...)->livewire(...)` or view-based patterns. Only touch Livewire's own configuration (component/view discovery paths, layout paths, etc.) if a task genuinely requires it — otherwise leave the default config alone.

```php
Route::livewire('/login', Login::class)->name('login');
Route::livewire('/register', Register::class)->name('register');

Route::middleware(['auth'])->group(function () {
    Route::livewire('/chat', ChatDashboard::class)->name('chat');
});
```

- Use `Route::livewire($uri, ComponentClass::class)->name(...)` for every route that renders a full Livewire page component (from `pages/`).
- Group routes needing middleware (e.g. `auth`) the normal Laravel way, with `Route::livewire()` calls inside the group closure — as shown above.
- Reserve plain `Route::get()`/`Route::post()` etc. for non-Livewire endpoints (e.g. API routes, webhooks, file downloads) — not for pages that are Livewire components.

---

## 4. Tailwind CSS & Styling Rules — Mobile-First, Utility-First

We adhere to a strict **utility-first CSS** approach, and layouts are always designed **mobile-first**.

- **Only Tailwind CSS:** Do not write custom manual CSS unless absolutely necessary (e.g., complex canvas drawings, custom keyframe animations not supportable by Tailwind, or legacy integrations). If custom styles are needed:
  - Put them in the component or page's specific `index.css` file within the MFC structure if the styles are exclusive to that component/page.
  - Only place custom styles in the global stylesheet if they are strictly global and necessary across multiple modules/pages.
- **Tailwind Classes:** Use class names to control layout, typography, color, responsiveness, state variants (e.g., `hover:`, `focus:`, `dark:`), and spacing.
- **Design Tokens:** Use consistent Tailwind colors, font sizes, and roundness to maintain design aesthetics. Always inspect the project's global stylesheet (e.g., `resources/css/app.css` or Tailwind configuration) for custom `@theme` variables before styling.
- **Modern UI Styling:** Follow modern design principles (glassmorphism, subtle gradients, rounded corners, drop shadows, responsive layouts, smooth hover animations).

### Mobile-First is Mandatory

Tailwind is mobile-first by design — unprefixed utilities apply to the smallest screen, and breakpoint prefixes (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`) layer on top for larger screens. **Always design and write markup in this direction: base classes = mobile layout, then add `md:`/`lg:` overrides for desktop.** Never do the reverse.

```html
<!-- Correct: mobile-first -->
<div class="flex flex-col gap-4 p-4 md:flex-row md:gap-8 md:p-8">

<!-- Wrong: desktop-first thinking, fighting the cascade -->
<div class="flex flex-row gap-8 p-8 max-md:flex-col max-md:gap-4 max-md:p-4">
```

- Default to single-column, stacked layouts (`flex-col`, `grid-cols-1`) at the base, then switch to multi-column/row layouts at `md:` and up.
- Think through the mobile experience first (tap targets, spacing, font sizes, stacked nav) before layering on desktop refinements.
- Avoid `max-*` variants (`max-md:`, `max-sm:`) as the primary layout mechanism — acceptable only for rare, isolated exceptions.

---

## 5. AlpineJS — Simple UI Interactivity Only

AlpineJS is for **small, self-contained UI state** — not general app logic. Keep it to things like toggling visibility, switching tabs, and open/close state.

- Use AlpineJS for simple, local UI interactivity only:
  - Modals/dialogs (open/close)
  - Sidebars/drawers (open/close)
  - Tabs (active tab switching)
  - Dropdowns, accordions, simple show/hide toggles
  - Typical pattern: a small `x-data="{ open: false }"` with `x-show`, `@click`, `x-if` — nothing more.
- **Do NOT put real functions or business logic in Alpine.** If a feature needs named functions like `startCapture()` / `stopCapture()`, API calls, multi-step state machines, or anything that would require a meaningful chunk of inline `x-data="{ ... }"` JavaScript in the Blade file, **that behavior belongs in Livewire (PHP), not Alpine.**
  - Symptom to watch for: if `x-data` is growing past a tiny object literal, or you're about to write functions inside it, stop — move that logic into the Livewire component's class instead and expose it via `wire:click`/public methods, or drive the state from a public Livewire property.
- Rule of thumb: **Alpine handles how something looks/appears on screen (visibility, active state); Livewire handles what actually happens (data, side effects, business logic).**
- Avoid vanilla JavaScript for the simple cases above — use Alpine's `x-show`/`x-if`/`@click` instead. Vanilla JS (`index.js`) is a last resort for cases neither Alpine nor Livewire can reasonably handle (e.g. a complex third-party JS library integration).

---

## 6. Package Policy — Ask First, Prefer Native Laravel/Livewire

Minimize third-party packages. Default to solving problems with plain Laravel/Livewire/Blade code rather than pulling in a package.

- **Never silently add a package.** If a task seems to need one (composer or npm), stop before implementing and **suggest the package(s), with a brief reason why**, then wait for confirmation. Don't install or wire it in until confirmed.
- Before suggesting a package, check whether the same result is reasonably achievable with native Laravel/Livewire/Blade/Alpine/Tailwind first, and prefer that path if it's not significantly more work.
- This applies to the planning step too: when a task is non-trivial, lay out a short implementation plan (including any package that would be proposed) and get confirmation before writing code — don't jump straight to coding.

---

## 7. Example Component
Here is the blueprint of a compliant Livewire 4 Multi-File Component:

### `index.php` (Class File)
```php
<?php

namespace App\Livewire\Dashboard\StatsCard;

use Livewire\Component;

class StatsCard extends Component
{
    public string $title;
    public int $value;
    public string $trend = 'up';

    public function mount(string $title, int $value, string $trend = 'up')
    {
        $this->title = $title;
        $this->value = $value;
        $this->trend = $trend;
    }

    public function refreshValue()
    {
        // Example action
        $this->value = rand(100, 999);
    }
}
```

### `index.blade.php` (Template File — mobile-first)
```html
<div class="p-4 bg-white dark:bg-secondary-900 rounded-2xl border border-secondary-100 dark:border-secondary-800 shadow-sm transition-all duration-300 hover:shadow-md md:p-6">
    <div class="flex items-center justify-between">
        <span class="text-sm font-medium text-secondary-500 dark:text-secondary-400">{{ $title }}</span>

        <button
            wire:click="refreshValue"
            class="p-1.5 rounded-lg text-secondary-400 hover:text-primary-600 hover:bg-primary-50 dark:hover:bg-primary-950/30 transition-colors"
            title="Refresh value"
        >
            <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 1121.21 8H18" />
            </svg>
        </button>
    </div>

    <div class="mt-4 flex items-baseline justify-between">
        <h3 class="text-2xl font-semibold tracking-tight text-secondary-900 dark:text-white md:text-3xl">
            {{ number_format($value) }}
        </h3>

        <span @class([
            'flex items-center px-2 py-0.5 rounded-full text-xs font-semibold',
            'bg-emerald-50 text-emerald-700 dark:bg-emerald-950/30 dark:text-emerald-400' => $trend === 'up',
            'bg-rose-50 text-rose-700 dark:bg-rose-950/30 dark:text-rose-400' => $trend === 'down',
        ])>
            @if ($trend === 'up')
                <svg class="w-3 h-3 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                </svg>
            @else
                <svg class="w-3 h-3 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
                </svg>
            @endif
            {{ $trend === 'up' ? '+12.5%' : '-3.2%' }}
        </span>
    </div>
</div>
```

---

## Quick Checklist Before Finishing Any Task

- [ ] Used latest Laravel conventions (checked version if unsure)
- [ ] Component created with `--mfc` (not the SFC default)
- [ ] Placed under `pages/` (full pages) or `component/` (reusable), grouped by module, with `index.php`/`index.blade.php` co-located
- [ ] Page routes use `Route::livewire($uri, Component::class)->name(...)`, not `Route::get()`
- [ ] No PHP logic or `<script>` tags inline in `index.blade.php`
- [ ] Styling is Tailwind utility classes; no manual CSS unless unavoidable
- [ ] Layout written mobile-first (base = mobile, `md:`/`lg:` layered on for desktop, not the reverse)
- [ ] Client interactivity uses AlpineJS only for simple UI state (show/hide, tabs, modals); logic-heavy behavior moved to Livewire instead of a bloated `x-data`
- [ ] No packages added without first suggesting them and getting confirmation
- [ ] For non-trivial tasks, implementation plan (incl. any proposed package) shared and confirmed before coding
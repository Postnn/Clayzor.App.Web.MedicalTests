> Глобальные правила и обзор решения — в корневом /AGENTS.md. Здесь — только специфика проекта Clayzor.App.Web.MedicalTests.

## Typography & Fonts

- Font face: **Verdana** + fallback `Arial, sans-serif` — defined via CSS variable `--clay-font-family`
- Font size: **0.8rem** (~13pt) — defined via CSS variable `--clay-font-size` (single token, NO size variants)
- MudBlazor typography (`ClayTheme.cs`) — all levels (Default, Body1, Body2, Subtitle1, Subtitle2, Caption, Overline) set `FontSize = "var(--clay-font-size)"`. H4/H5/H6/Button keep their own sizes (heading/button scale)
- MudBlazor input controls use `font-size: var(--clay-font-size) !important` (CSS rules in `app.css` for `.mud-input-control input`, `.mud-input-slot`, `.mud-input-label-outlined`, and `.mud-input-outlined-border legend`)
- MudBlazor outlined labels use `transform: scale(0.75)` by default — overridden in `app.css` via `scale(1)` with `!important` on both `.mud-input.mud-shrink ~ .mud-input-label-outlined` and `.mud-input-control:focus-within .mud-input-label-outlined`. Key detail: `mud-shrink` class is on `div.mud-input`, NOT on the `<label>` — label is a sibling AFTER `.mud-input`, so the selector uses `~` (general sibling combinator). Focused empty fields have NO `mud-shrink` class, hence the separate `:focus-within` selector.
- Legend notch in outlined border is sized via `font-size: var(--clay-font-size) !important` on `.mud-input-outlined-border legend` (both `.mud-shrink` and `:focus-within` variants)
- No external font CDN dependencies (Google Fonts Inter removed)

## Style enforcement (STYLE_RULES.md, `promts/_done/STYLE_PROMPTS.md`)

Пошаговые промты для внедрения единого стиля — `promts/_done/STYLE_PROMPTS.md` (Промт 0–6). Выполнены.
Закон стиля — `STYLE_RULES.md`.

**All visual styling (color, font, background, border, shadow, radius) lives ONLY in:**
- `Clayzor.Lib.Web.Controls/wwwroot/css/clay.css` — общий стиль компонентов RCL
- `<App>/wwwroot/css/app.css` — специфика конкретного приложения (напр. Dense form)
- `Clayzor.Lib.Web.Controls/Themes/ClayTheme.cs` — MudBlazor theme palette
- `Clayzor.Lib.Web.Controls/Themes/ClayColors.cs` — single source of brand hex values

### Architecture (Variant A)
`ClayColors.cs` (C# hex literals) → `ClayTheme.cs` (MudBlazor palette) → MudBlazor emits `--mud-palette-*` CSS variables → `app.css` aliases them as `--clay-*` variables. Dark mode adapts automatically.

### Build-time enforcement
- **`build/StyleGuard.targets`** — MSBuild inline C# task (`BeforeTargets="CoreCompile"`). Scans `**/*.razor` and `**/*.cs` for visual inline styles (`color:`, `background`, `border`, `font-*`, hex colors, `rgba(`). Build FAILS on violations with file/line/error details.
- White-listed files (excluded from scanning): `app.css`, `ClayTheme.cs`, `ClayGridPrintStyles.cs`, `ClayGridPrintHtmlGenerator.cs`, `ClayGridExcelGenerator.cs`
- **Allowed in `style=`/`Style=`**: structural properties only — `display`, `flex`, `gap`, `width`, `padding`, `margin`, `cursor`, `overflow`, `text-overflow`, `position`, `z-index`, `opacity`, `white-space`, etc.
- **Prohibited in `style=`/`Style=`**: `color`, `background*`, `border*`, `box-shadow`, `font-*`, `fill`, `stroke`, `letter-spacing`, `text-transform`, hex colors, `rgb(`/`rgba(` with color values
- All `<style>` blocks in `.razor` must be moved to `app.css`

### Before completing any UI task (checklist)
- No visual `style=`/`Style=` in changed `.razor`
- No hex colors/font strings in `.cs` outside white-list
- Colors use `var(--mud-palette-*)` (adaptive) or `var(--clay-*)` (brand aliases)
- New visual patterns → CSS class in `app.css`
- Layout → MudBlazor utility classes (`d-flex`, `gap-*`, `pa-*`)
- `dotnet build` passes (StyleGuard checks active)

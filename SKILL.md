---
name: wm-add-page
description: Scaffold a new page in a WaveMaker-generated Angular project. Creates the 6 standard WaveMaker page files, registers the route, adds a NavigationVariable action, wires a sidebar link, and adds the compiled click expression. Use when the user asks to "add a page", "create a page", "scaffold a page", or "add a new page to sidebar" in a WaveMaker Angular project.
argument-hint: <PageName> [caption] [icon-class]
allowed-tools: [Read, Edit, Write, Bash, Glob, Grep]
---

# Add a New Page to a WaveMaker Angular Project

## Arguments

The user invoked this with: $ARGUMENTS

Parse `$ARGUMENTS` as follows:
- **First token (required)** — PageName in PascalCase (e.g. `UserPage`, `Dashboard`). If missing or not PascalCase, ask the user.
- **Optional remaining tokens** — sidebar caption (e.g. `"User Page"`) and icon class (e.g. `wi wi-account-circle`). If omitted, derive caption from PageName (split on capitals: `UserPage` → "User Page") and default icon to `wi wi-square-o`. Always confirm the derived values back to the user before scaffolding.

If `$ARGUMENTS` is empty, ask the user for at minimum the PageName.

## When to use

Trigger this skill when the user asks to add/create/scaffold a new page in a WaveMaker Angular project. Indicators:

- The repo has `src/app/pages/` with at least one page folder containing `*.component.{ts,html,css,variables.ts,expressions.ts,script.js}` files (6-file pattern).
- The repo has `src/app/partials/` containing partials like `header`, `topnav`, `leftnav`, `footer`.
- `package.json` depends on `@wavemaker/app-ng-runtime`.

If any of these are missing, this is not a WaveMaker Angular project — stop and tell the user.

## Inputs to collect (in order)

Ask only for what you can't infer. If the user already provided the name, don't ask again.

1. **PageName** (PascalCase, required) — e.g. `UserPage`, `Dashboard`, `Settings`. Used for folder name, class name, route path, and `pageName`.
2. **Sidebar caption** (default: a humanized form of PageName) — e.g. `UserPage` → "User Page".
3. **Icon class** (default `wi wi-square-o`) — any WaveMaker `wi wi-*` icon class. Common ones: `wi wi-account-circle`, `wi wi-bar-graph`, `wi wi-file`, `wi wi-tag`, `wi wi-envelope`, `wi wi-home`, `wi wi-list`, `wi wi-view-carousel`, `wi wi-code`.
4. **Layout** (default `full-shell`) — `full-shell` includes header/topnav/leftnav/footer like the `Main` page; `minimal` is content-only like the `Login` page. Most pages should be `full-shell`.
5. **Embed a plain Angular child?** (default `no`) — if yes, also scaffold a separate non-WaveMaker standalone component embedded in the page's content area. See the "Plain Angular child" section below.

## Pre-flight: pin templates to the project's actual conventions

WaveMaker generators slightly evolve over time. Don't blindly use the templates below — first read **one existing page** as the source of truth and mirror its imports/structure exactly. Recommended:

1. Read `src/app/pages/Main/Main.component.ts` (or any full-shell page) to see exactly which `@wm/components/*` modules are imported and which partials are imported. **Use that import list as your base** — the literal lists below are illustrative defaults.
2. Read `src/app/pages/Main/Main.component.html` to confirm the shell markup pattern (attribute names, directive selectors, `@if (compilePageContent)` block, `{{onPageContentReady()}}` call).
3. Read `src/app/partials/leftnav/leftnav.component.html` to find the highest existing `wm_anchor<N>` reference. **Increment from there** when adding a new sidebar link, otherwise template reference collisions break the partial.
4. Read `src/app/partials/leftnav/leftnav.component.expressions.ts` to see the exact compiled-function shape used in this project (it may have additional fields).
5. Read `src/app/app.routes.ts` to see whether the project uses `loadComponent` (standalone, modern) or `loadChildren`/eager imports. Match it.

If the project diverges from any template below, **follow the project's pattern**, not these defaults.

## Files to create

All under a new folder `src/app/pages/<PageName>/`. Replace `<PageName>` with the PascalCase name (e.g. `UserPage`) and `<pagename>` with the lowercased name used in the `name` attribute (e.g. `userpage`).

### 1. `<PageName>.component.ts` (full-shell layout)

```typescript
import { Component, ViewEncapsulation, NO_ERRORS_SCHEMA } from '@angular/core';
import { UserDefinedExecutionContext } from '@wm/core';
import { initScript } from './<PageName>.component.script';
import { getVariables } from './<PageName>.component.variables';
import { expressionData } from './<PageName>.component.expressions';
import { CommonModule } from '@angular/common';
import { BasePageComponent } from '@wm/runtime/base';

import { PartialParamHandlerDirective as WM_PartialParamHandlerDirective } from '@wm/components/base';
import { PartialContainerDirective as WM_PartialContainerDirective } from '@wm/components/base';
import { FormsModule as ngFormsModule } from '@angular/forms';
import { ContentComponent as WM_ContentComponent } from '@wm/components/page';
import { PageContentComponent as WM_PageContentComponent } from '@wm/components/page';
import { PageDirective as WM_PageDirective } from '@wm/components/page';
import { FooterDirective as WM_FooterDirective } from '@wm/components/page/footer';
import { HeaderComponent as WM_HeaderComponent } from '@wm/components/page/header';
import { LeftPanelDirective as WM_LeftPanelDirective } from '@wm/components/page/left-panel';
import { TopNavDirective as WM_TopNavDirective } from '@wm/components/page/top-nav';

import { HeaderComponent as PartialHeaderComponent } from '../../partials/header/header.component';
import { TopnavComponent as PartialTopnavComponent } from '../../partials/topnav/topnav.component';
import { LeftnavComponent as PartialLeftnavComponent } from '../../partials/leftnav/leftnav.component';
import { FooterComponent as PartialFooterComponent } from '../../partials/footer/footer.component';


const requiredComponentModules = [
    WM_PartialParamHandlerDirective,
    WM_PartialContainerDirective,
    ngFormsModule,
    WM_ContentComponent,
    WM_PageContentComponent,
    WM_PageDirective,
    WM_FooterDirective,
    WM_HeaderComponent,
    WM_LeftPanelDirective,
    WM_TopNavDirective
];

const requiredCustomWidgetComponents = [
    // Add plain Angular child components here (e.g. AngularDemoComponent)
];

const requiredPartialModules: any[] = [
    PartialHeaderComponent,
    PartialTopnavComponent,
    PartialLeftnavComponent,
    PartialFooterComponent
];

@Component({
    selector: 'app-page-<PageName>',
    templateUrl: './<PageName>.component.html',
    styleUrls: ['./<PageName>.component.css'],
    encapsulation: ViewEncapsulation.None,
    providers: [
        {
            provide: UserDefinedExecutionContext,
            useExisting: <PageName>Component
        }
    ],
    standalone: true,
    imports: [
        ...requiredComponentModules,
        ...requiredPartialModules,
        ...requiredCustomWidgetComponents,
        CommonModule,
    ],
})
export class <PageName>Component extends BasePageComponent {

    override pageName = '<PageName>';
    [key: string]: any;

    constructor() {
        super();
        super.init();
    }

    getVariables() {
        return getVariables();
    }

    evalUserScript(Page, App, Utils) {
        initScript(Page, App, Utils);
    }

    getExpressions() {
        return expressionData;
    }

}
```

**If you need to use a widget that isn't in the import list above** (e.g. `Label`, `Button`, `Form`, etc.), read another existing page that uses that widget (e.g. `pages/Login` uses many form widgets) and copy the matching `import { ... } from '@wm/components/*'` lines into both the import block and `requiredComponentModules`.

### 2. `<PageName>.component.html` (full-shell layout)

```html
<div wmPage #wm_page1="wmPage" data-role="pageContainer" [attr.aria-label]="wm_page1.arialabel"  name="<pagename>" pagetitle="<PageName>">
    <header wmHeader #wm_header1="wmHeader" partialContainer data-role="page-header" role="banner" [attr.aria-label]="wm_header1.arialabel || 'Page header'"  content="header" name="header" height="auto"></header>
    <section wmTopNav #wm_top_nav1="wmTopNav" partialContainer data-role="page-topnav" role="navigation" [attr.aria-label]="wm_top_nav1.arialabel || 'Second level navigation'"  name="topnav" content="topnav"></section>
    <main wmContent data-role="page-content" role="main"  name="content">
        <aside wmLeftPanel #wm_left_panel1="wmLeftPanel" partialContainer data-role="page-left-panel" [attr.aria-label]="wm_left_panel1.arialabel || 'Left navigation panel'" wmSmoothscroll="false"  columnwidth="2" name="leftpanel" content="leftnav"></aside>
        <div wmPageContent  wmSmoothscroll="false"  columnwidth="10" name="mainContent"><ng-container >@if (compilePageContent) {
            <!-- Page body goes here. For an embedded plain Angular child: <app-<pagename>-demo></app-<pagename>-demo> -->
        {{onPageContentReady()}}}</ng-container></div>
    </main>
    <footer wmFooter #wm_footer1="wmFooter" partialContainer data-role="page-footer" role="contentinfo" [attr.aria-label]="wm_footer1.arialabel || 'Page footer'"  name="footer" content="footer"></footer>
</div>
```

**Important**: the `@if (compilePageContent) { ... {{onPageContentReady()}} }` block is required — WaveMaker's runtime uses both the gate and the `onPageContentReady()` call to drive the page lifecycle. Do not remove either.

### 3. `<PageName>.component.css`

Empty file. Add page-specific styles here later.

### 4. `<PageName>.component.variables.ts`

```typescript
export const variables = {}

export const getVariables = () => JSON.parse(JSON.stringify(variables))
```

### 5. `<PageName>.component.expressions.ts`

```typescript
export const expressionData = {

};
```

### 6. `<PageName>.component.script.js`

```javascript
export const initScript = (Page, App, Utils) => {
    /*
     * Use App.getDependency for Dependency Injection
     * eg: var DialogService = App.getDependency('DialogService');
     */

    Page.onReady = function() {
        /*
         * variables can be accessed through 'Page.Variables' property here
         * widgets can be accessed through 'Page.Widgets' property here
         */
    };
}
```

## Files to update (4 places — all required for navigation to work)

### A. Register the route — `src/app/app.routes.ts`

Find the inner `children` array that holds the existing page routes (sibling of the `Login` and `Main` route entries). Append a new entry following the exact shape of the `Main` route in the same file:

```typescript
{
    path: "<PageName>",
    pathMatch: "full",
    loadComponent: () =>
        import("./pages/<PageName>/<PageName>.component").then(
            m => m.<PageName>Component
        ),
    data: {
        pageName: "<PageName>"
    },
    canDeactivate: [
        (
            component: CanComponentDeactivate,
            currentRoute: ActivatedRouteSnapshot,
            currentState: RouterStateSnapshot,
            nextState: RouterStateSnapshot
        ) =>
            inject(CanDeactivateNgPageGuard).canDeactivate(
                component,
                currentRoute,
                currentState,
                nextState
            )
    ]
}
```

Notes:
- The `**` wildcard route (`PageNotFoundGuard`) must remain *after* the children array. Insert your new entry as a sibling of `Main`, not after the wildcard.
- The imports at the top of `app.routes.ts` (`CanComponentDeactivate`, `ActivatedRouteSnapshot`, `RouterStateSnapshot`, `CanDeactivateNgPageGuard`, `inject`) are already there from existing routes — don't duplicate them.

### B. Add the NavigationVariable — `src/app/app.component.variables.ts`

This file holds app-level `Actions` shared across pages. Add a `gotoPage` variable so the sidebar (and any other place) can navigate via `Actions.goToPage_<PageName>.invoke()`:

```typescript
"goToPage_<PageName>" : {
    "_id" : "wm-wm.NavigationVariable-<pagename>-1",
    "name" : "goToPage_<PageName>",
    "owner" : "App",
    "category" : "wm.NavigationVariable",
    "operation" : "gotoPage",
    "pageName" : "<PageName>"
},
```

The `_id` must be unique within the file — appending `<pagename>-<n>` is fine; bump the trailing number if the same lowercase name already appears.

### C. Add the sidebar link — `src/app/partials/leftnav/leftnav.component.html`

**Critical**: before writing, grep the file for the highest existing `wm_anchor<N>` template reference (e.g. if `wm_anchor8` exists, use `wm_anchor9`). Collisions silently break the leftnav partial.

Append inside the `<ul wmNav>` block:

```html
<li wmNavItem role="listitem"  name="list_<pagename>">
    <a wmAnchor #wm_anchor<N>="wmAnchor" role="link" data-identifier="anchor" [attr.aria-label]="wm_anchor<N>.arialabel || (wm_anchor<N>.badgevalue ? wm_anchor<N>.caption + ' ' + wm_anchor<N>.badgevalue : wm_anchor<N>.caption) || null"  caption="<Caption>" name="<pagename>Link" iconclass="<IconClass>" click.event="Actions.goToPage_<PageName>.invoke()"></a>
</li>
```

`<N>` is the next anchor number. `<Caption>` is the user-facing label. `<IconClass>` is the WaveMaker icon class (e.g. `wi wi-account-circle`).

### D. Add the compiled click expression — `src/app/partials/leftnav/leftnav.component.expressions.ts`

The `click.event="Actions.goToPage_<PageName>.invoke()"` string on the sidebar anchor doesn't execute on its own — WaveMaker's runtime looks up its compiled form in this file's `expressionData` map. Add an entry whose key matches the `click.event` expression exactly:

```typescript
"Actions.goToPage_<PageName>.invoke()": [function(_plus, _minus, _isDef, _ctx, locals) {
        "use strict";
        var v0, v1, v2, v3, v4, ctx;
        v0 = Object.assign({}, locals);
        Object.setPrototypeOf(v0, _ctx);
        ctx = v0;
        v1 = ctx && ctx.Actions;
        v2 = v1 && v1.goToPage_<PageName>;
        v3 = v2 && v2.invoke;
        v4 = v3 && v3.bind(v2)();
        return v4;
    },
    []
],
```

Match the existing entries in the file for the exact function shape. The IDE will show implicit-`any` warnings for `_plus`, `_minus`, `_isDef`, `_ctx`, `locals` — this is normal and matches the project's other expression files; the project's `tsconfig.json` doesn't enable `strict` so the build passes.

## Optional: embedding a plain Angular child component

If the user wants a non-WaveMaker Angular component (a standalone component with zero `@wm/*` imports) to render inside the page's content area:

1. **Create the child** at `src/app/pages/<PageName>/<pagename>-demo.component.ts`:

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

@Component({
    selector: 'app-<pagename>-demo',
    standalone: true,
    imports: [CommonModule, FormsModule],
    template: `<div>Plain Angular content here</div>`,
    styles: [`/* component-scoped styles */`]
})
export class <PageName>DemoComponent {
    // logic, state, lifecycle hooks — all plain Angular
}
```

2. **Import the child in the WM page wrapper** (`<PageName>.component.ts`):
   - Add: `import { <PageName>DemoComponent } from './<pagename>-demo.component';`
   - Add `<PageName>DemoComponent` to the `requiredCustomWidgetComponents` array.

3. **Embed the child in the WM page template** (`<PageName>.component.html`) inside the `wmPageContent` block:
   ```html
   <app-<pagename>-demo></app-<pagename>-demo>
   ```

4. **For web-component-based libraries** (e.g. Swiper, Lit, etc.) used inside the child, add `schemas: [CUSTOM_ELEMENTS_SCHEMA]` to the child component's `@Component` decorator and register the custom elements once at module load (e.g. `import { register } from 'swiper/element/bundle'; register();`).

5. **For Angular Material** (or any library requiring animations), confirm `provideAnimations()` is already in `src/app/app.config.ts` (it is in stock WaveMaker projects). Material additionally requires a theme — import a prebuilt theme into `src/styles.css`:
   ```css
   @import "@angular/material/prebuilt-themes/azure-blue.css";
   ```

## Verification

After all 10 file changes (6 new + 4 edits), verify:

1. **No stale references**:
   ```bash
   grep -rn "<PageName>" src/
   ```
   Every match should be intentional.

2. **Anchor numbering is unique**:
   ```bash
   grep -oE "wm_anchor[0-9]+" src/app/partials/leftnav/leftnav.component.html | sort -u
   ```
   No duplicates.

3. **Build runs**: `npm start` (or whatever the project's dev script is). The dev server should compile without errors.

4. **Navigation works**: click the new sidebar item — the URL should become `#/<PageName>` and the page renders inside the WaveMaker shell.

## Common pitfalls

- **Duplicate `wm_anchor<N>` reference** in `leftnav.component.html` — silently breaks the partial; always grep for the highest existing number first.
- **Forgot to add the compiled expression** in `leftnav.component.expressions.ts` — the sidebar link will render but clicking does nothing.
- **Wrong `pageName` casing** — `pageName` in the route's `data`, the component's `pageName` override, and the NavigationVariable's `pageName` field must all match the folder name and route path exactly (case-sensitive).
- **Missing widget imports** — if the page uses a widget like `wmButton` or `wmLabel` that isn't in `requiredComponentModules`, Angular will throw a template-parse error. Cross-check against an existing page that uses the same widget.
- **Inserting the route after the `**` wildcard** — the wildcard catches everything; new routes must come before it (i.e. inside the same `children` array as `Main`).
- **Empty `compilePageContent` block** — removing `@if (compilePageContent) { ... }` or `{{onPageContentReady()}}` breaks the page lifecycle; always keep both.
- **Stripping `[key: string]: any`** from the component class — required by WaveMaker's runtime for dynamic widget property access.

## What the user-facing summary should say

When done, briefly list the 10 changes (6 created files + 4 edited files), with clickable file paths, and tell the user to run `npm start` and click the new sidebar item to verify.

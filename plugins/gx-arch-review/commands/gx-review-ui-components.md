# Galaxy Client Component Review Command

## Purpose

Review Galaxy client code for proper component usage, ensuring developers use Galaxy's native component library instead of deprecated Bootstrap-Vue components.

## Input

Accept input in any of these forms:
- A working directory path (analyze git diff in that directory)
- A Git commit reference (analyze changes in that commit)
- A PR reference (analyze changes in that pull request)
- A list of Vue/TypeScript file paths (analyze those files)
- A planning document (analyze the Vue files in the plan)

## Review Focus

Check that new/modified Vue components use Galaxy's custom component library over Bootstrap-Vue equivalents.

## Component Migration Reference

### Required Replacements

| Deprecated (Bootstrap-Vue) | Use Instead (Galaxy) |
|---------------------------|---------------------|
| `BButton`, `b-button` | `GButton` |
| `BLink`, `b-link` | `GLink` |
| `BModal`, `b-modal` | `GModal` |
| `BCard`, `b-card` | `GCard` |

### GButton (replaces BButton)

```vue
<!-- Deprecated -->
<BButton variant="primary" size="sm" :disabled="busy">Submit</BButton>

<!-- Correct -->
<GButton color="blue" size="small" :disabled="busy">Submit</GButton>
```

Key differences:
- `variant` → `color` (grey, blue, green, yellow, orange, red)
- `size="sm"` → `size="small"` (small, medium, large)
- Built-in tooltip support via `tooltip` prop
- Polymorphic: renders as `<button>`, `<a>`, or `<router-link>` based on props

Source: `client/src/components/BaseComponents/GButton.vue`

### GLink (replaces BLink)

```vue
<!-- Deprecated -->
<BLink href="#" @click="doSomething">Click here</BLink>

<!-- Correct -->
<GLink @click="doSomething">Click here</GLink>
```

Source: `client/src/components/BaseComponents/GLink.vue`

### GModal (replaces BModal)

```vue
<!-- Deprecated -->
<BModal v-model="showModal" title="Confirm" ok-only>Content</BModal>

<!-- Correct -->
<GModal v-model:show="showModal" title="Confirm" confirm>Content</GModal>
```

Key differences:
- Uses native `<dialog>` element for better accessibility
- `v-model` → `v-model:show`
- `ok-only` → `confirm`

Source: `client/src/components/BaseComponents/GModal.vue`

### GCard (replaces BCard and custom card layouts)

```vue
<!-- Deprecated custom layout -->
<div class="workflow-card">
  <div class="card-header"><h3>{{ name }}</h3></div>
  <div class="card-body">{{ description }}</div>
</div>

<!-- Correct -->
<GCard :id="id" :title="name" :description="description" />
```

Source: `client/src/components/Common/GCard.vue`

## Galaxy Component Patterns

When reviewing, verify these patterns are followed:

### Polymorphic Usage
```vue
<GButton @click="action">Button</GButton>     <!-- renders <button> -->
<GButton href="/page">Anchor</GButton>        <!-- renders <a> -->
<GButton to="/route">Router Link</GButton>    <!-- renders <router-link> -->
```

### Integrated Tooltips
```vue
<!-- Deprecated (directive) -->
<BButton v-b-tooltip.hover title="Click me">Button</BButton>

<!-- Correct (integrated prop) -->
<GButton tooltip title="Click me">Button</GButton>
```

### Semantic Colors
Available colors: `grey` (default), `blue`, `green`, `yellow`, `orange`, `red`

### Standard Props
```typescript
import type { ComponentColor, ComponentSize } from "./componentVariants";

interface Props {
  color?: ComponentColor;       // grey, blue, green, yellow, orange, red
  size?: ComponentSize;         // small, medium, large
  disabled?: boolean;
  title?: string;
  disabledTitle?: string;
  tooltip?: boolean;
}
```

## Key Source Files

Reference these when reviewing or suggesting fixes:

- **Type definitions**: `client/src/components/BaseComponents/componentVariants.ts`
- **Shared composables**: `client/src/components/BaseComponents/composables/`
  - `clickableElement.ts` - Determines element type from props
  - `currentTitle.ts` - Handles disabled title switching
- **Tooltip integration**: `client/src/components/BaseComponents/GTooltip.vue`

## Reference PRs

Migration examples:
- [#19963](https://github.com/galaxyproject/galaxy/pull/19963) - Button migration
- [#20063](https://github.com/galaxyproject/galaxy/pull/20063) - Link migration
- [#20168](https://github.com/galaxyproject/galaxy/pull/20168) - Modal migration
- [#19785](https://github.com/galaxyproject/galaxy/pull/19785) - Card migration

## Review Checklist

For each Vue file with UI components:

1. **No new Bootstrap-Vue imports** - Check for `import { BButton, BModal, ... } from 'bootstrap-vue'`
2. **No Bootstrap-Vue template tags** - Check for `<b-button>`, `<b-modal>`, `<b-link>`, `<b-card>`
3. **Correct prop mapping** - Verify `variant` → `color`, `size="sm"` → `size="small"`
4. **Proper v-model for modals** - Use `v-model:show` not `v-model`
5. **Tooltip integration** - Use `tooltip` prop not `v-b-tooltip` directive

## Output Format

Report findings as:

```markdown
## Component Review: [file/commit/PR]

### Issues Found

1. **file.vue:42** - Uses `BButton`, replace with `GButton`
   ```vue
   <!-- Current -->
   <BButton variant="primary">Submit</BButton>
   <!-- Suggested -->
   <GButton color="blue">Submit</GButton>
   ```

2. **file.vue:78** - Uses `BModal` with incorrect v-model
   ```vue
   <!-- Current -->
   <BModal v-model="show">
   <!-- Suggested -->
   <GModal v-model:show="show">
   ```

### Summary
- X deprecated components found
- Y files need migration
```

---
status: draft
date: 2026-04-13
---

# Menu

Compound, headless menu components for media controls — settings, option selection, and context actions.

## Problem

Video players need menus for three core interactions:

1. **Settings** — quality, playback speed, captions, audio tracks
2. **Option selection** — single-choice (radio) and multi-choice (checkbox) groups
3. **Context actions** — copy link, report, stats

The dominant pattern for settings menus in video players is in-place navigation: clicking a settings category slides the view to a submenu, and clicking back returns to the root. This is what YouTube, Plyr, and most native player UIs do. Flyout (side-opening) submenus are a valid extension point but not the starting design.

Requirements:

- Compound and composable — users assemble parts, omit what they don't need
- Headless — no baked-in styles; CSS custom properties for animation values
- Accessible — `role="menu"`, full keyboard support, roving tabindex, type-ahead
- Cascading submenus — in-place slide transitions with animated container resize
- Treeshakeable — parts only imported when used
- Cross-platform — same core logic drives React components and HTML custom elements

## API

### React

```tsx
import { Menu } from '@videojs/react';

<Menu.Root>
  <Menu.Trigger>Settings</Menu.Trigger>
  <Menu.Content>
    <Menu.SubMenu>
      <Menu.SubMenuTrigger>Quality</Menu.SubMenuTrigger>
      <Menu.SubMenuContent>
        <Menu.SubMenuBack />
        <Menu.RadioGroup value={quality} onValueChange={setQuality}>
          <Menu.RadioItem value="auto">Auto</Menu.RadioItem>
          <Menu.RadioItem value="1080p">1080p</Menu.RadioItem>
          <Menu.RadioItem value="720p">720p</Menu.RadioItem>
        </Menu.RadioGroup>
      </Menu.SubMenuContent>
    </Menu.SubMenu>

    <Menu.SubMenu>
      <Menu.SubMenuTrigger>Speed</Menu.SubMenuTrigger>
      <Menu.SubMenuContent>
        <Menu.SubMenuBack />
        <Menu.RadioGroup value={speed} onValueChange={setSpeed}>
          <Menu.RadioItem value="0.5">0.5×</Menu.RadioItem>
          <Menu.RadioItem value="1">Normal</Menu.RadioItem>
          <Menu.RadioItem value="2">2×</Menu.RadioItem>
        </Menu.RadioGroup>
      </Menu.SubMenuContent>
    </Menu.SubMenu>

    <Menu.Separator />
    <Menu.Item onSelect={copyLink}>Copy Link</Menu.Item>
  </Menu.Content>
</Menu.Root>
```

### HTML

```ts
import '@videojs/html/ui/menu';
```

```html
<button commandfor="settings-menu">Settings</button>
<media-menu id="settings-menu" side="top" align="end">

  <media-menu-sub>
    <media-menu-sub-trigger>Quality</media-menu-sub-trigger>
    <media-menu-sub-content>
      <media-menu-sub-back></media-menu-sub-back>
      <media-menu-radio-group value="auto">
        <media-menu-radio-item value="auto">Auto</media-menu-radio-item>
        <media-menu-radio-item value="1080p">1080p</media-menu-radio-item>
        <media-menu-radio-item value="720p">720p</media-menu-radio-item>
      </media-menu-radio-group>
    </media-menu-sub-content>
  </media-menu-sub>

  <media-menu-sub>
    <media-menu-sub-trigger>Speed</media-menu-sub-trigger>
    <media-menu-sub-content>
      <media-menu-sub-back></media-menu-sub-back>
      <media-menu-radio-group value="1">
        <media-menu-radio-item value="0.5">0.5×</media-menu-radio-item>
        <media-menu-radio-item value="1">Normal</media-menu-radio-item>
        <media-menu-radio-item value="2">2×</media-menu-radio-item>
      </media-menu-radio-group>
    </media-menu-sub-content>
  </media-menu-sub>

  <media-menu-separator></media-menu-separator>
  <media-menu-item>Copy Link</media-menu-item>

</media-menu>
```

### Parts

All parts are exported under `Menu.*` (React) or as `<media-menu-*>` elements (HTML). One import gives access to everything:

```ts
import { Menu } from '@videojs/react';
// Menu.Root, Menu.Trigger, Menu.Content,
// Menu.Item, Menu.Label, Menu.Separator, Menu.Group,
// Menu.RadioGroup, Menu.RadioItem, Menu.CheckboxItem, Menu.ItemIndicator,
// Menu.SubMenu, Menu.SubMenuTrigger, Menu.SubMenuContent, Menu.SubMenuBack
```

---

#### Root

Context provider. Owns menu state and creates the underlying `createMenu()` instance. Provides context to all children. Does not render a DOM element in React — Content is the rendered container. In HTML, `<media-menu>` serves as both Root and Content.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `open` | `boolean` | — | Controlled open state. |
| `defaultOpen` | `boolean` | `false` | Initial open state (uncontrolled). |
| `side` | `PopoverSide` | `'bottom'` | Which side of the trigger the menu appears on. |
| `align` | `PopoverAlign` | `'start'` | Alignment along the trigger's edge. |
| `closeOnEscape` | `boolean` | `true` | Close the menu when Escape is pressed at root level. |
| `closeOnOutsideClick` | `boolean` | `true` | Close the menu when clicking outside. |
| `onOpenChange` | `(open: boolean) => void` | — | Fired when the menu opens or closes. |

---

#### Trigger

Button that opens and closes the menu.

**ARIA (automatic):**

| Attribute | Value |
|-----------|-------|
| `aria-haspopup` | `"menu"` |
| `aria-expanded` | `"true"` when open, `"false"` when closed |
| `aria-controls` | ID of the Content element |

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `render` | `RenderProp<MenuState>` | Custom render element. |

---

#### Content

Popup container. Acts as the **viewport** for menu navigation — only one view (the root list or a single submenu) is visible at a time. Handles keyboard navigation, type-ahead, popover positioning, and container resize animation between views.

**ARIA (automatic):**

| Attribute | Value |
|-----------|-------|
| `role` | `"menu"` |
| `tabIndex` | `-1` |
| `popover` | `"manual"` |

**Data attributes** — set on Content and inherited by all children:

| Attribute | Values | When |
|-----------|--------|------|
| `data-open` | present/absent | Menu is open |
| `data-starting-style` | present/absent | Open transition in progress |
| `data-ending-style` | present/absent | Close transition in progress |
| `data-side` | `top` / `bottom` / `left` / `right` | Popover side |
| `data-align` | `start` / `center` / `end` | Popover alignment |

**CSS custom properties** (set by JS on Content during submenu transitions):

| Property | Description |
|----------|-------------|
| `--media-menu-width` | Width of the incoming view |
| `--media-menu-height` | Height of the incoming view |
| `--media-menu-available-height` | Viewport-constrained max height |

**HTML events** (`<media-menu>` dispatches, all bubble):

| Event | Detail | Fires when |
|-------|--------|------------|
| `open-change` | `{ open: boolean }` | Menu opens or closes |

---

#### Item

Standard menu item for actions. Activating fires `onSelect` and closes the menu.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `disabled` | `boolean` | `false` | Disables the item. |
| `onSelect` | `() => void` | — | Fired on click, Enter, or Space. |
| `render` | `RenderProp<MenuItemState>` | — | Custom render element. |

**ARIA (automatic):** `role="menuitem"`, roving `tabIndex`, `aria-disabled`.

**Data attributes:** `data-highlighted`. Use `[aria-disabled="true"]` in CSS for disabled styling.

---

#### Label

Non-interactive heading within a group. Not keyboard-navigable.

**Props:** `render`.

**ARIA (automatic):** `role="presentation"`.

---

#### Separator

Visual divider between groups or items. Not focusable.

**ARIA (automatic):** `role="separator"`.

---

#### Group

Groups related items for assistive technology.

**Props:**

| Prop | Type | Description |
|------|------|-------------|
| `label` | `string` | Accessible label (`aria-label`). |
| `render` | `RenderProp<MenuState>` | Custom render element. |

**ARIA (automatic):** `role="group"`, `aria-label`.

---

#### RadioGroup

Single-selection group. Manages value state — controlled or uncontrolled. In a submenu, selecting a RadioItem automatically navigates back to the parent view (matches YouTube/Plyr behavior).

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | — | Controlled selected value. |
| `defaultValue` | `string` | — | Initial value (uncontrolled). |
| `onValueChange` | `(value: string) => void` | — | Fired when selection changes. |
| `label` | `string` | — | Accessible group label. |
| `render` | `RenderProp<MenuState>` | — | Custom render element. |

**ARIA (automatic):** `role="group"`, `aria-label`.

---

#### RadioItem

Item within a RadioGroup. Represents one selectable option.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | — | Value this item represents. |
| `disabled` | `boolean` | `false` | Disables the item. |
| `render` | `RenderProp<MenuRadioItemState>` | — | Custom render element. |

**ARIA (automatic):** `role="menuitemradio"`, `aria-checked`, roving `tabIndex`, `aria-disabled`.

**Data attributes:** `data-highlighted`. Use `[aria-checked="true"]` and `[aria-disabled="true"]` in CSS for checked and disabled styling.

**Behavior:** In a SubMenuContent, selecting a RadioItem auto-pops back to the parent view after calling `onValueChange`.

---

#### CheckboxItem

Toggle item with independent checked/unchecked state.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `checked` | `boolean` | — | Controlled checked state. |
| `defaultChecked` | `boolean` | `false` | Initial checked state (uncontrolled). |
| `onCheckedChange` | `(checked: boolean) => void` | — | Fired when state toggles. |
| `disabled` | `boolean` | `false` | Disables the item. |
| `render` | `RenderProp<MenuCheckboxItemState>` | — | Custom render element. |

**ARIA (automatic):** `role="menuitemcheckbox"`, `aria-checked`, roving `tabIndex`, `aria-disabled`.

**Data attributes:** `data-highlighted`. Use `[aria-checked="true"]` and `[aria-disabled="true"]` in CSS for checked and disabled styling.

---

#### ItemIndicator

Visual indicator that renders when the parent RadioItem or CheckboxItem is checked. Use for checkmarks, dots, or icons.

**Props:** `render`.

**Behavior:** Reads checked state from the nearest parent item context. Hidden from assistive technology — the parent item's `aria-checked` provides the semantic.

---

#### SubMenu

Context provider wrapping a trigger+content pair. Creates a submenu entry in the navigation stack. Does not render a DOM element in React. In HTML, `<media-menu-sub>` is discovered by the parent `<media-menu>` as a submenu container.

No props beyond children.

---

#### SubMenuTrigger

Menu item that pushes a submenu onto the navigation stack. Visually functions as both a forward navigation button and a display of the current selection (via children/slots).

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `disabled` | `boolean` | `false` | Disables the trigger. |
| `render` | `RenderProp<MenuItemState>` | — | Custom render element. |

**ARIA (automatic):** `role="menuitem"`, roving `tabIndex`, `aria-disabled`.

**Data attributes:** `data-highlighted`. Use `[aria-disabled="true"]` in CSS for disabled styling.

**Behavior:**

- Click, Enter, Space, or `ArrowRight` pushes this submenu.
- After push, focus moves to the first item in SubMenuContent (after transition).
- Self-registers as a navigable item for keyboard navigation.

---

#### SubMenuContent

The in-place submenu view. When pushed, it replaces the current viewport content with an animated slide transition. When popped, it slides out as the parent view slides back in.

**ARIA (automatic):** `role="menu"`, `tabIndex="-1"`.

**Data attributes:**

| Attribute | Values | When |
|-----------|--------|------|
| `data-open` | present/absent | This submenu is the active view |
| `data-starting-style` | present/absent | Submenu is transitioning in |
| `data-ending-style` | present/absent | Submenu is transitioning out |
| `data-direction` | `forward` / `back` | Direction of the current transition |

**Behavior:**

- Keyboard navigation scoped to items within this content.
- `ArrowLeft` or `Escape` pops back to parent.
- Type-ahead operates within this view only.

---

#### SubMenuBack

Button that pops the current submenu, returning to the parent view. Typically placed at the top of SubMenuContent.

**Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | `'Back'` | Accessible label (`aria-label`). |
| `render` | `RenderProp<MenuState>` | — | Custom render element. |

**ARIA (automatic):** `aria-label` from `label` prop.

**Behavior:**

- Click calls pop on the navigation stack.
- After pop, focus returns to the SubMenuTrigger that navigated forward.
- Hidden (or disabled) when already at root depth.

---

### HTML element tags

| Part | Tag |
|------|-----|
| Root / Content | `<media-menu>` |
| Item | `<media-menu-item>` |
| Label | `<media-menu-label>` |
| Separator | `<media-menu-separator>` |
| Group | `<media-menu-group>` |
| RadioGroup | `<media-menu-radio-group>` |
| RadioItem | `<media-menu-radio-item>` |
| CheckboxItem | `<media-menu-checkbox-item>` |
| ItemIndicator | `<media-menu-item-indicator>` |
| SubMenu | `<media-menu-sub>` |
| SubMenuTrigger | `<media-menu-sub-trigger>` |
| SubMenuContent | `<media-menu-sub-content>` |
| SubMenuBack | `<media-menu-sub-back>` |

## Navigation model

Content acts as a fixed-size **viewport**. Only one view is visible at a time: the root list or one submenu's content. The navigation state is a **stack**:

```ts
type StackEntry = {
  /** ID of the SubMenu that was pushed. */
  subMenuId: string;
  /** ID of the SubMenuTrigger that initiated the push, for focus restoration. */
  triggerId: string;
};

type NavigationState = {
  stack: StackEntry[];
  direction: 'forward' | 'back' | null;
  exitingSubMenuId: string | null;
  transitioning: boolean;
};
```

### Push (forward)

1. User clicks/activates a `SubMenuTrigger` or presses `ArrowRight` on one.
2. `{ subMenuId, triggerId }` is pushed onto the stack.
3. Both the outgoing view and the incoming `SubMenuContent` are in the DOM simultaneously.
4. First RAF: incoming SubMenuContent is measured. `--media-menu-width` and `--media-menu-height` are set on Content.
5. Second RAF: browser has painted the "from" state. CSS transitions animate the container resize; CSS animations slide the views.
6. `getAnimations()` on Content settles — transition complete.
7. `exitingSubMenuId` is cleared. Only the active view remains.
8. Focus moves to the first item in the new SubMenuContent.

### Pop (back)

1. User clicks `SubMenuBack`, presses `ArrowLeft`, or presses `Escape` while in a submenu.
2. `stack.pop()`. Direction set to `'back'`.
3. Same double-RAF + animation settle lifecycle as push, but views slide in the reverse direction.
4. Focus returns to the `SubMenuTrigger` identified by the popped entry's `triggerId`.

### Reset

On menu close, the stack resets to `[]` immediately — no submenu animation plays. The popover's own close animation (`data-ending-style` on Content) covers the visual exit. When the menu re-opens, it starts at the root view.

### Auto-back on RadioItem selection

When a `RadioItem` inside a `SubMenuContent` is activated, the menu automatically pops back to the parent view after calling `onValueChange`. This is the expected YouTube/Plyr behavior — select an option, return to settings root.

Users who need to stay in the submenu after selection should use `CheckboxItem` or a custom `Item` instead.

### Rapid navigation

If the user navigates (push or pop) while a transition is in progress, the current transition is cancelled (skip to end state), and the new transition starts immediately. Follows the same cancel pattern as `createTransition()`.

### Nesting depth

The stack supports arbitrary depth. A submenu can contain another `SubMenu`, creating a nested navigation path (e.g., Settings → Quality → Advanced). The stack grows and shrinks accordingly.

## CSS animation

Animation is driven entirely by data attributes and CSS custom properties. No inline styles are applied. The skin provides defaults; users can override or replace them.

### Data attributes

**On Content** (and inherited by all children):

| Attribute | Values | When |
|-----------|--------|------|
| `data-open` | present/absent | Menu is open |
| `data-starting-style` | present/absent | Open transition in progress |
| `data-ending-style` | present/absent | Close transition in progress |
| `data-side` | `top` / `bottom` / `left` / `right` | Popover positioning side |
| `data-align` | `start` / `center` / `end` | Popover positioning alignment |

**On SubMenuContent:**

| Attribute | Values | When |
|-----------|--------|------|
| `data-open` | present/absent | This submenu is the active view |
| `data-starting-style` | present/absent | Submenu is entering |
| `data-ending-style` | present/absent | Submenu is exiting |
| `data-direction` | `forward` / `back` | Direction of the transition |

**On items:**

| Attribute | Values | When |
|-----------|--------|------|
| `data-highlighted` | present/absent | Item has keyboard or pointer focus |

`aria-checked` and `aria-disabled` are set by the component and should be used directly as CSS selectors — no redundant data attributes.

### CSS custom properties

Set by JS on Content before each submenu transition:

| Property | Example | Description |
|----------|---------|-------------|
| `--media-menu-width` | `240px` | Width of the incoming view |
| `--media-menu-height` | `320px` | Height of the incoming view |
| `--media-menu-available-height` | `480px` | Viewport-constrained max height for the menu |

### Example CSS

```css
/* Container resizes to match the incoming submenu view */
media-menu {
  width: var(--media-menu-width);
  height: var(--media-menu-height);
  max-height: var(--media-menu-available-height, none);
  overflow: hidden;
  transition:
    width 200ms ease,
    height 200ms ease;
}

/* Menu open/close — fade + slight scale */
@starting-style {
  media-menu[data-open] {
    opacity: 0;
    transform: scale(0.97);
  }
}
media-menu[data-ending-style] {
  opacity: 0;
  transform: scale(0.97);
}
media-menu {
  transition:
    opacity 150ms ease,
    transform 150ms ease;
}

/* Hide when closed — override with display:block for transitions */
media-menu:not([data-open]) {
  display: none;
}

/* Submenu slides — forward: in from right, out to left */
media-menu-sub-content[data-starting-style][data-direction="forward"] {
  transform: translateX(100%);
}
media-menu-sub-content[data-ending-style][data-direction="forward"] {
  transform: translateX(-100%);
}

/* Submenu slides — back: in from left, out to right */
media-menu-sub-content[data-starting-style][data-direction="back"] {
  transform: translateX(-100%);
}
media-menu-sub-content[data-ending-style][data-direction="back"] {
  transform: translateX(100%);
}

media-menu-sub-content {
  transition: transform 200ms ease;
}

/* RTL — flip slide direction */
[dir="rtl"] media-menu-sub-content[data-starting-style][data-direction="forward"] {
  transform: translateX(-100%);
}
[dir="rtl"] media-menu-sub-content[data-ending-style][data-direction="forward"] {
  transform: translateX(100%);
}
[dir="rtl"] media-menu-sub-content[data-starting-style][data-direction="back"] {
  transform: translateX(100%);
}
[dir="rtl"] media-menu-sub-content[data-ending-style][data-direction="back"] {
  transform: translateX(-100%);
}

/* Item highlight */
media-menu-item[data-highlighted],
media-menu-radio-item[data-highlighted],
media-menu-checkbox-item[data-highlighted] {
  background: rgba(255, 255, 255, 0.1);
}

/* Checked and disabled — use ARIA attributes directly */
media-menu-radio-item[aria-checked="true"]::before,
media-menu-checkbox-item[aria-checked="true"]::before {
  content: '✓';
}

media-menu-item[aria-disabled="true"],
media-menu-radio-item[aria-disabled="true"],
media-menu-checkbox-item[aria-disabled="true"],
media-menu-sub-trigger[aria-disabled="true"] {
  opacity: 0.4;
  pointer-events: none;
}
```

**Transition completion detection:** JS uses `el.getAnimations()` on the Content element to wait for all CSS transitions and animations to finish before updating state (clearing `exitingSubMenuId`, moving focus). Same pattern as `createTransition()`.

**RTL:** `ArrowRight` always pushes (opens submenu), `ArrowLeft` always pops — these are logical operations independent of text direction. The physical slide direction (which way views move on screen) is flipped in CSS via `[dir="rtl"]`.

## Keyboard

Keyboard events are handled by the currently active view's container (Content for root, SubMenuContent for submenus). Focus uses **roving tabindex** — only the highlighted item has `tabindex="0"`, all others have `tabindex="-1"`.

| Key | Behavior |
|-----|---------|
| `ArrowDown` | Next item in current view (wraps) |
| `ArrowUp` | Previous item in current view (wraps) |
| `ArrowRight` | Push submenu (if focused item is a SubMenuTrigger) |
| `ArrowLeft` | Pop submenu (if in a submenu); no-op at root |
| `Home` | First item in current view |
| `End` | Last item in current view |
| `Enter` / `Space` | Activate focused item |
| `Escape` | Pop submenu if in one; close menu at root |
| `a-z`, `0-9` | Type-ahead search in current view |

**Type-ahead:** Printable characters accumulate into a buffer. The search starts from the item after the current highlight and wraps. Buffer resets after 500ms of inactivity. Matches `textContent` of each item. 500ms is the standard window used by Radix, Base UI, and native OS menus.

## Accessibility

The menu follows the [WAI-ARIA Menu Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menu/) and [Menu Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/).

```html
<!-- Root level -->
<button
  aria-haspopup="menu"
  aria-expanded="true"
  aria-controls="settings-menu">
  Settings
</button>
<div id="settings-menu" role="menu" tabindex="-1">
  <div role="menuitem" tabindex="0">Copy Link</div>
  <div role="separator"></div>
  <div role="group" aria-label="Quality">
    <div role="menuitemradio" aria-checked="false" tabindex="-1">Auto</div>
    <div role="menuitemradio" aria-checked="true" tabindex="-1">1080p</div>
  </div>
</div>

<!-- Submenu (when active — replaces root view in the viewport) -->
<div role="menu" tabindex="-1">
  <button aria-label="Back"></button>
  <div role="group" aria-label="Quality">
    <div role="menuitemradio" aria-checked="false" tabindex="0">Auto</div>
    <div role="menuitemradio" aria-checked="true" tabindex="-1">1080p</div>
  </div>
</div>
```

**Roles:**

| Part | Role |
|------|------|
| Content | `menu` |
| SubMenuContent | `menu` |
| Item | `menuitem` |
| SubMenuTrigger | `menuitem` |
| RadioItem | `menuitemradio` |
| CheckboxItem | `menuitemcheckbox` |
| Group / RadioGroup | `group` |
| Label | `presentation` |
| Separator | `separator` |

**Focus management:**

| Event | Focus behavior |
|-------|----------------|
| Menu opens | Focus Content, then first item (or previously-highlighted item) |
| Menu closes | Focus returns to Trigger |
| Submenu push | After transition: focus moves to first item in SubMenuContent |
| Submenu pop | After transition: focus returns to the SubMenuTrigger that initiated the push |

**Screen reader announcements:**

- `aria-checked` changes on RadioItem and CheckboxItem are announced natively via `role="menuitemradio/checkbox"`.
- Submenu title changes on push/pop use `aria-live="polite"` on a visually hidden region inside Content — announcing the current panel name to users who can't see the slide animation.

**Roving tabindex vs `aria-activedescendant`:** Roving tabindex is used (recommended by WAI-ARIA, used by Radix and Base UI). Items receive real DOM focus, so `:focus-visible` works naturally for keyboard-only styling without extra CSS.

## Architecture

Three layers, each independently useful:

| Layer | Package | Purpose |
|-------|---------|---------|
| Core | `@videojs/core` | State computation, ARIA attributes, navigation stack. No DOM. |
| DOM | `@videojs/core/dom` | Keyboard navigation, type-ahead, submenu transitions, focus management. |
| UI | `@videojs/react`, `@videojs/html` | Compound components and custom elements. |

### Core layer

`MenuCore` follows the `PopoverCore` pattern — a framework-agnostic class that computes state and ARIA attributes from props and input.

```ts
interface MenuProps {
  side?: PopoverSide;
  align?: PopoverAlign;
  open?: boolean;
  defaultOpen?: boolean;
  closeOnEscape?: boolean;
  closeOnOutsideClick?: boolean;
}

interface MenuState extends TransitionFlags {
  open: boolean;
  status: TransitionStatus;
  side: PopoverSide;
  align: PopoverAlign;
  highlightedIndex: number;
}

type NavigationState = {
  stack: Array<{ subMenuId: string; triggerId: string }>;
  direction: 'forward' | 'back' | null;
  exitingSubMenuId: string | null;
  transitioning: boolean;
};
```

Constants follow the `*-data-attrs.ts` / `*-css-vars.ts` pattern from the slider:

```ts
// menu-data-attrs.ts
export const MenuDataAttrs = {
  open: 'data-open',
  side: 'data-side',
  align: 'data-align',
  startingStyle: 'data-starting-style',
  endingStyle: 'data-ending-style',
} as const;

// menu-item-data-attrs.ts
export const MenuItemDataAttrs = {
  highlighted: 'data-highlighted',
  checked: 'data-checked',
  disabled: 'data-disabled',
} as const;

// menu-sub-data-attrs.ts
export const MenuSubDataAttrs = {
  open: 'data-open',
  startingStyle: 'data-starting-style',
  endingStyle: 'data-ending-style',
  direction: 'data-direction',
} as const;

// menu-css-vars.ts
export const MenuCSSVars = {
  width: '--media-menu-width',
  height: '--media-menu-height',
  availableHeight: '--media-menu-available-height',
} as const;
```

### DOM layer

`createMenu()` composes `createPopover()` internally for open/close, positioning, and dismiss behavior, then layers menu-specific keyboard navigation and focus management on top.

```ts
interface MenuApi {
  input: State<MenuInput>;
  navigationState: State<NavigationState>;
  triggerProps: MenuTriggerProps;
  contentProps: MenuContentProps;
  setTriggerElement: (el: HTMLElement | null) => void;
  setContentElement: (el: HTMLElement | null) => void;
  open: (reason?: PopoverOpenChangeReason) => void;
  close: (reason?: PopoverOpenChangeReason) => void;
  registerItem: (el: HTMLElement, options?: { disabled?: boolean }) => () => void;
  highlight: (index: number) => void;
  push: (subMenuId: string, triggerId: string) => void;
  pop: () => void;
  destroy: () => void;
}
```

`createSubMenuTransition()` handles the double-RAF lifecycle for submenu navigation (same pattern as `createTransition()`).

**Item collection:** Items self-register via `registerItem(el)` returning a cleanup function. The collection is kept sorted by `compareDocumentPosition`. Works across Shadow DOM boundaries without coupling to ARIA role strings.

### File structure

**Core** (`packages/core/src/core/ui/menu/`):

```text
menu-core.ts
menu-data-attrs.ts
menu-item-data-attrs.ts
menu-sub-data-attrs.ts
menu-css-vars.ts
```

**DOM** (`packages/core/src/dom/ui/menu/`):

```text
create-menu.ts
create-sub-menu-transition.ts
```

**React** (`packages/react/src/ui/menu/`):

```text
context.tsx
index.parts.ts
index.ts
menu-root.tsx
menu-trigger.tsx
menu-content.tsx
menu-item.tsx
menu-label.tsx
menu-separator.tsx
menu-group.tsx
menu-radio-group.tsx
menu-radio-item.tsx
menu-checkbox-item.tsx
menu-item-indicator.tsx
menu-sub.tsx
menu-sub-trigger.tsx
menu-sub-content.tsx
menu-sub-back.tsx
```

**HTML** (`packages/html/src/ui/menu/`):

```text
menu-element.ts
menu-item-element.ts
menu-label-element.ts
menu-separator-element.ts
menu-group-element.ts
menu-radio-group-element.ts
menu-radio-item-element.ts
menu-checkbox-item-element.ts
menu-item-indicator-element.ts
menu-sub-element.ts
menu-sub-trigger-element.ts
menu-sub-content-element.ts
menu-sub-back-element.ts
```

Importing `@videojs/html/ui/menu` registers all elements.

### Popover integration

`createMenu()` creates a `createPopover()` instance internally for open/close, Escape handling, outside-click dismissal, CSS Anchor Positioning (with JS fallback), and hover intent. The popover is an implementation detail, not exposed in the menu API. Menu adds what's unique: `role="menu"`, roving tabindex, arrow key navigation, type-ahead, and the navigation stack.

## Prior art

**YouTube / Plyr** — The in-place panel navigation model this design is based on. Click a category (Quality, Speed), slide to a radio list, select, auto-slide back. Both reset to root on close.

**Base UI Menu** — Compound `Menu.Root` / `Menu.Trigger` / `Menu.Positioner` / `Menu.Popup` / `Menu.Item` pattern. Uses `data-open`, `data-starting-style`, `data-ending-style` on Popup for CSS-driven transitions. CSS custom properties (`--available-height`) for viewport constraints. Strongly influenced this doc's animation API.

**Radix Dropdown Menu** — `DropdownMenu.Root` / `DropdownMenu.Content` / `DropdownMenu.Item` / `DropdownMenu.Sub` pattern. Roving tabindex. `data-state="open|closed"` on Content (we use `data-open` to match Base UI). Submenu opens flyout-style (not in-place).

**Shadcn** — Wraps Radix with default styling. Demonstrates that the compound namespace pattern (`Menu.Root`, `Menu.Content`) is familiar and expected to users of modern UI libraries.

**Vidstack** — In-place menu navigation with slide transitions, very close to what this design targets. Uses a `Menu` + `MenuButton` + `MenuItems` pattern with a navigation stack internally.

## Descoped

| Feature | Reason |
|---------|---------|
| Flyout (side-opening) submenus | Wrong starting point for video player settings menus. Add later via Portal-based SubMenuContent if needed. |
| Context menus (right-click) | Different trigger model and positioning concerns. Separate design. |
| `mode` prop | Only one navigation model — cascading submenus. No polymorphic composition needed. |
| Arbitrary panel IDs | Removed with the old panel API. SubMenu wraps its own trigger + content; IDs are internal. |
| Hover-to-open submenus | Deferred. Desktop flyout behavior — not applicable to the in-place cascading model. |

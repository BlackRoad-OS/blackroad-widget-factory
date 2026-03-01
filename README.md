# blackroad-widget-factory

**Production-grade UI widget configuration system for BlackRoad OS dashboards.**

[![CI](https://github.com/BlackRoad-OS/blackroad-widget-factory/actions/workflows/ci.yml/badge.svg)](https://github.com/BlackRoad-OS/blackroad-widget-factory/actions/workflows/ci.yml)
[![Python 3.11+](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://www.python.org/)
[![License: Proprietary](https://img.shields.io/badge/License-Proprietary-red.svg)](./LICENSE)

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Widget Types](#widget-types)
4. [Installation](#installation)
5. [Quick Start](#quick-start)
6. [CLI Reference](#cli-reference)
7. [Python API Reference](#python-api-reference)
8. [React / npm Integration](#react--npm-integration)
9. [Stripe Integration](#stripe-integration)
10. [Database Schema](#database-schema)
11. [Export Formats](#export-formats)
12. [Testing](#testing)
13. [E2E Testing](#e2e-testing)
14. [CI / CD](#ci--cd)
15. [License](#license)

---

## Overview

`blackroad-widget-factory` is the canonical widget management layer for BlackRoad OS. It provides a typed Python API to create, persist, validate, and export UI widgets, as well as first-class React prop and PropTypes generation for seamless JavaScript/npm consumption. Layouts can be exported as JSON, CSS Grid, or ready-to-render HTML, and widgets are persisted in an embedded SQLite database.

---

## Features

| Capability | Detail |
|---|---|
| Typed widget dataclass | `widget_id`, `widget_type`, `config`, `position` — 20+ widget types |
| Grid layout composition | `WidgetLayout` with configurable column count and row height |
| JSON serialization | Serialize / deserialize any widget or layout to/from JSON |
| React prop generation | Auto-generate `props`, `propTypes`, `defaultProps`, and `componentName` |
| Config validation | Type checking + grid overflow detection |
| Multi-format export | JSON, CSS Grid, and HTML layout exports |
| SQLite persistence | `widgets` and `layouts` tables with indexed foreign keys |
| Stripe-ready config | Payment widget configs (checkout, subscription, portal) |
| CLI interface | Full CRUD via `widget-factory` command |

---

## Widget Types

| Type | React Component | Description |
|---|---|---|
| `button` | `Button` | Actionable button with variant and disabled state |
| `input` | `Input` | Text input with type, placeholder, and validation |
| `text` | `Typography` | Styled text block |
| `image` | `Image` | Image with src, alt, and dimensions |
| `chart` | `Chart` | Bar, line, pie, or area chart |
| `table` | `DataTable` | Sortable, paginated data table |
| `card` | `Card` | Content card with title, body, and footer |
| `modal` | `Modal` | Dismissible dialog overlay |
| `dropdown` | `Select` | Single or multi-select dropdown |
| `checkbox` | `Checkbox` | Boolean input with indeterminate state |
| `slider` | `Slider` | Numeric range slider |
| `progress` | `ProgressBar` | Labeled progress indicator |
| `badge` | `Badge` | Status badge |
| `avatar` | `Avatar` | User avatar |
| `tooltip` | `Tooltip` | Contextual tooltip |
| `tabs` | `Tabs` | Tab strip |
| `accordion` | `Accordion` | Collapsible content accordion |
| `navbar` | `Navbar` | Navigation bar |
| `sidebar` | `Sidebar` | Sidebar navigation panel |
| `form` | `Form` | Composite form container |

---

## Installation

### Python

```bash
# Clone the repository
git clone https://github.com/BlackRoad-OS/blackroad-widget-factory.git
cd blackroad-widget-factory

# Install dependencies (pytest required for testing only)
pip install pytest
```

### npm / JavaScript

React prop output from this library is designed to be consumed directly by any npm-based React project. Copy or pipe the JSON output of `react-props` into your component tree:

```bash
# Generate props JSON and pipe into your React build pipeline
python src/main_module.py react-props <widget_id> > widget-props.json
```

Then in your JavaScript project:

```js
import widgetProps from './widget-props.json';
import Button from './components/Button';

// widgetProps.props, widgetProps.propTypes, widgetProps.defaultProps are all available
const { props, defaultProps } = widgetProps;
<Button {...defaultProps} {...props} />
```

---

## Quick Start

```python
from src.main_module import Widget, Position, WidgetLayout, get_db, save_widget, generate_react_props

# Open (or create) the local database
conn = get_db()

# Create a button widget
btn = Widget(
    widget_type="button",
    label="Checkout",
    config={"label": "Checkout", "variant": "primary", "disabled": False},
    position=Position(x=0, y=0, width=3, height=1),
)
save_widget(btn, conn=conn)

# Generate React props
props = generate_react_props(btn)
print(props["componentName"])   # "Button"
print(props["propTypes"])       # {'id': 'PropTypes.string', 'visible': 'PropTypes.bool', ...}

# Build a layout
layout = WidgetLayout(name="dashboard", columns=12)
layout.widgets.append(btn)
from src.main_module import save_layout
save_layout(layout, conn)
```

---

## CLI Reference

Run all commands with:

```bash
python src/main_module.py [--db <path>] <command> [options]
```

| Command | Description | Key Options |
|---|---|---|
| `create-widget <type>` | Create and persist a new widget | `--label`, `--config <json>`, `--x`, `--y`, `--width`, `--height` |
| `get-widget <widget_id>` | Retrieve widget JSON by ID | — |
| `delete-widget <widget_id>` | Delete a widget | — |
| `validate <widget_id>` | Validate widget config and grid position | — |
| `react-props <widget_id>` | Generate React props JSON | — |
| `create-layout <name>` | Create a named layout | `--columns` (default 12) |
| `add-to-layout <layout> <widget_id>` | Assign a widget to a layout | — |
| `export-layout <layout>` | Export a layout | `--format json\|css\|html`, `--output <file>` |
| `list-layouts` | List all layouts with widget counts | — |

### Examples

```bash
# Create a primary button widget at grid position (0, 0)
python src/main_module.py create-widget button \
  --label "Submit" \
  --config '{"label":"Submit","variant":"primary","disabled":false}' \
  --x 0 --y 0 --width 3 --height 1

# Retrieve it
python src/main_module.py get-widget <widget_id>

# Validate configuration
python src/main_module.py validate <widget_id>

# Generate React props
python src/main_module.py react-props <widget_id>

# Create a 12-column dashboard layout
python src/main_module.py create-layout "dashboard" --columns 12

# Add widget to layout
python src/main_module.py add-to-layout dashboard <widget_id>

# Export the layout
python src/main_module.py export-layout dashboard --format json
python src/main_module.py export-layout dashboard --format css
python src/main_module.py export-layout dashboard --format html --output layout.html

# List all layouts
python src/main_module.py list-layouts
```

---

## Python API Reference

### `Widget`

```python
@dataclass
class Widget:
    widget_type: str               # One of WIDGET_TYPES
    config: Dict[str, Any]         # Type-specific configuration
    position: Position             # Grid position
    widget_id: str                 # UUID (auto-generated)
    label: str                     # Accessible label / aria-label
    visible: bool                  # Render visibility flag
    created_at: str                # ISO 8601 timestamp
    db_id: Optional[int]           # SQLite row ID (set after save)
```

### `Position`

```python
@dataclass
class Position:
    x: int       # Column start (0–11)
    y: int       # Row start
    width: int   # Column span (1–12)
    height: int  # Row span (≥ 1)
```

### `WidgetLayout`

```python
@dataclass
class WidgetLayout:
    name: str
    widgets: List[Widget]
    columns: int        # Grid columns (default 12)
    row_height: int     # Row height in px (default 60)
    layout_id: Optional[int]
    created_at: str
```

### Functions

| Function | Signature | Description |
|---|---|---|
| `get_db` | `(db_path) → Connection` | Open / initialize SQLite database |
| `save_widget` | `(widget, layout_id, conn) → Widget` | Insert or update a widget |
| `load_widget` | `(widget_id, conn) → Optional[Widget]` | Load widget by UUID |
| `delete_widget` | `(widget_id, conn) → bool` | Delete widget, returns `True` if found |
| `save_layout` | `(layout, conn) → WidgetLayout` | Insert or update a layout and its widgets |
| `load_layout` | `(name, conn) → Optional[WidgetLayout]` | Load layout with all widgets |
| `list_layouts` | `(conn) → list` | List all layouts with widget counts |
| `serialize_widget` | `(widget) → dict` | Convert widget to JSON-safe dict |
| `deserialize_widget` | `(data) → Widget` | Restore widget from dict or JSON string |
| `generate_react_props` | `(widget) → dict` | Generate React props, propTypes, defaultProps |
| `validate_config` | `(widget) → List[str]` | Return validation error strings (empty = valid) |
| `export_layout` | `(layout, fmt) → str` | Export layout as `json`, `css`, or `html` |

---

## React / npm Integration

`blackroad-widget-factory` generates fully-typed React props from any widget configuration. This output is compatible with any React 17+ project and can be published or consumed as an npm package.

### Generated Output Shape

```json
{
  "componentName": "Button",
  "props": {
    "label": "Submit",
    "variant": "primary",
    "disabled": false,
    "visible": true,
    "id": "550e8400-e29b-41d4-a716-446655440000"
  },
  "propTypes": {
    "id": "PropTypes.string",
    "visible": "PropTypes.bool",
    "label": "PropTypes.string",
    "variant": "PropTypes.string",
    "disabled": "PropTypes.bool"
  },
  "defaultProps": {
    "visible": true,
    "label": "Submit",
    "variant": "primary",
    "disabled": false
  },
  "position": { "x": 0, "y": 0, "width": 3, "height": 1 }
}
```

### Using with a React/npm project

```bash
# 1. Export widget props
python src/main_module.py react-props <widget_id> > src/widget-config.json

# 2. Install peer dependencies in your npm project
npm install prop-types react react-dom
```

```jsx
// MyWidget.jsx
import PropTypes from 'prop-types';
import widgetConfig from './widget-config.json';

const { componentName, defaultProps } = widgetConfig;

export default function MyWidget(props) {
  return <button {...defaultProps} {...props}>{defaultProps.label}</button>;
}

MyWidget.propTypes = {
  label: PropTypes.string,
  variant: PropTypes.string,
  disabled: PropTypes.bool,
};
```

---

## Stripe Integration

`blackroad-widget-factory` is **Stripe-ready** — payment widgets can be configured with the `button` and `form` widget types to wire up Stripe Checkout, Stripe Elements, or Customer Portal flows.

### Stripe Checkout Button

```python
from src.main_module import Widget, Position, get_db, save_widget

conn = get_db()

checkout_btn = Widget(
    widget_type="button",
    label="Stripe Checkout",
    config={
        "label": "Pay Now",
        "variant": "primary",
        "disabled": False,
        # Stripe-specific metadata stored in config
        "stripe_price_id": "price_XXXXXXXXXXXX",
        "stripe_mode": "payment",          # "payment" | "subscription"
        "stripe_success_url": "https://yourdomain.com/success",
        "stripe_cancel_url": "https://yourdomain.com/cancel",
    },
    position=Position(x=0, y=0, width=3, height=1),
)
save_widget(checkout_btn, conn=conn)
```

### Stripe Subscription Widget

```python
subscription_btn = Widget(
    widget_type="button",
    label="Subscribe",
    config={
        "label": "Subscribe — $29/mo",
        "variant": "primary",
        "disabled": False,
        "stripe_price_id": "price_XXXXXXXXXXXX",
        "stripe_mode": "subscription",
        "stripe_success_url": "https://yourdomain.com/dashboard",
        "stripe_cancel_url": "https://yourdomain.com/pricing",
    },
    position=Position(x=0, y=1, width=4, height=1),
)
save_widget(subscription_btn, conn=conn)
```

### Wiring Stripe in React (npm)

```bash
npm install @stripe/stripe-js @stripe/react-stripe-js
```

```jsx
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(process.env.REACT_APP_STRIPE_PUBLISHABLE_KEY);

async function handleCheckout(widgetConfig) {
  const stripe = await stripePromise;
  await stripe.redirectToCheckout({
    lineItems: [{ price: widgetConfig.props.stripe_price_id, quantity: 1 }],
    mode: widgetConfig.props.stripe_mode,
    successUrl: widgetConfig.props.stripe_success_url,
    cancelUrl: widgetConfig.props.stripe_cancel_url,
  });
}
```

> **Security note:** Never commit live Stripe secret keys. Use environment variables (`STRIPE_SECRET_KEY`) and restrict keys to the minimum required permissions.

---

## Database Schema

All data is stored in a local SQLite database (default: `~/.blackroad/widget-factory.db`).

```sql
-- Layouts table
CREATE TABLE layouts (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        TEXT    NOT NULL UNIQUE,
    columns     INTEGER NOT NULL DEFAULT 12,
    row_height  INTEGER NOT NULL DEFAULT 60,
    created_at  TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

-- Widgets table
CREATE TABLE widgets (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    layout_id   INTEGER REFERENCES layouts(id) ON DELETE SET NULL,
    widget_id   TEXT    NOT NULL UNIQUE,
    widget_type TEXT    NOT NULL,
    label       TEXT    NOT NULL DEFAULT '',
    config      TEXT    NOT NULL DEFAULT '{}',
    pos_x       INTEGER NOT NULL DEFAULT 0,
    pos_y       INTEGER NOT NULL DEFAULT 0,
    pos_width   INTEGER NOT NULL DEFAULT 4,
    pos_height  INTEGER NOT NULL DEFAULT 2,
    visible     INTEGER NOT NULL DEFAULT 1,
    created_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

-- Indexes
CREATE INDEX idx_widgets_layout ON widgets(layout_id);
CREATE INDEX idx_widgets_type   ON widgets(widget_type);
```

---

## Export Formats

### JSON

Full widget and position data — suitable for API responses or hydrating a React app.

```bash
python src/main_module.py export-layout dashboard --format json
```

### CSS Grid

Ready-to-embed stylesheet for positioning widgets in a CSS Grid container.

```bash
python src/main_module.py export-layout dashboard --format css
```

Sample output:

```css
.layout-dashboard {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  row-gap: 60px;
}

.widget-550e8400 {
  grid-column: 1 / span 3;
  grid-row: 1 / span 1;
}
```

### HTML

Full HTML scaffold with inline grid styles — paste directly into a prototype.

```bash
python src/main_module.py export-layout dashboard --format html --output layout.html
```

---

## Testing

### Unit Tests

```bash
pip install pytest
python -m pytest tests/ -v
```

All 14 unit tests cover:

- Widget CRUD (create, read, delete)
- Serialization / deserialization (dict and JSON string)
- Config validation (valid, unknown type, grid overflow)
- React prop generation (button and chart)
- Layout creation and widget assignment
- Layout export (JSON, CSS, HTML)

### Linting

```bash
pip install flake8
flake8 src/ --max-line-length=120 --ignore=E501,W503
```

---

## E2E Testing

End-to-end tests exercise the full pipeline — CLI → SQLite → serialization → React prop output — using only a temporary in-memory database.

### Running E2E Tests

```bash
python -m pytest tests/ -v -k "e2e"
```

### Manual E2E Walkthrough

```bash
# 1. Create a widget via CLI
python src/main_module.py create-widget button \
  --label "E2E Button" \
  --config '{"label":"E2E Button","variant":"primary","disabled":false}' \
  --x 0 --y 0 --width 3 --height 1

# Note the widget_id printed on success.

# 2. Retrieve and validate
python src/main_module.py get-widget <widget_id>
python src/main_module.py validate <widget_id>

# 3. Build a layout and export
python src/main_module.py create-layout "e2e-dashboard" --columns 12
python src/main_module.py add-to-layout e2e-dashboard <widget_id>
python src/main_module.py export-layout e2e-dashboard --format json
python src/main_module.py export-layout e2e-dashboard --format css
python src/main_module.py export-layout e2e-dashboard --format html

# 4. Generate React props
python src/main_module.py react-props <widget_id>
```

Expected results at each step:

| Step | Expected Output |
|---|---|
| `create-widget` | `✓ Widget created: <uuid> (button)` |
| `get-widget` | JSON blob with `widget_type`, `config`, `position` |
| `validate` | `✓ Widget <uuid> is valid` |
| `create-layout` | `✓ Layout 'e2e-dashboard' created (id=1)` |
| `add-to-layout` | `✓ Widget <uuid> added to layout 'e2e-dashboard'` |
| `export-layout --format json` | JSON with `name`, `columns`, `widgets` array |
| `export-layout --format css` | CSS Grid stylesheet |
| `export-layout --format html` | HTML `<div class="grid-layout">` scaffold |
| `react-props` | JSON with `componentName`, `props`, `propTypes`, `defaultProps` |

---

## CI / CD

Continuous integration runs on every push and pull request via GitHub Actions.

```yaml
# .github/workflows/ci.yml
- Lint:  flake8 src/ --max-line-length=120 --ignore=E501,W503
- Test:  python -m pytest tests/ -v
```

View the latest run status in the badge at the top of this file.

---

## License

Copyright © 2024–2026 BlackRoad OS, Inc. All Rights Reserved.  
Founder, CEO & Sole Stockholder: Alexa Louise Amundson.

See [LICENSE](./LICENSE) for full terms.

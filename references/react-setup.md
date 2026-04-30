# React + Babel Project Conventions

Technical conventions you must follow when prototyping with HTML+React+Babel. Break them and things blow up.

## Pinned Script Tags (use these versions exactly)

Put these three script tags in your HTML's `<head>`, using **pinned versions + integrity hash**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Don't** use unpinned versions like `react@18` or `react@latest` — you'll hit version drift / cache problems.

**Don't** omit `integrity` — it's your defense if a CDN gets hijacked or tampered with.

## File Structure

```
project-name/
├── index.html               # Main HTML
├── components.jsx           # Component file (loaded with type="text/babel")
├── data.js                  # Data file
└── styles.css               # Extra CSS (optional)
```

How to load them in HTML:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- Then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- Finally, the main entry point -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Don't** use `type="module"` — it conflicts with Babel.

## Three rules you must never break

### Rule 1: `styles` objects must use unique names

**Wrong** (will explode the moment you have multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← collides with same name!
const styles = { container: {...}, header: {...} };
```

**Right**: each component file's `styles` should use a unique prefix.

```jsx
// terminal.jsx
const terminalStyles = { 
  screen: {...}, 
  line: {...} 
};

// sidebar.jsx
const sidebarStyles = { 
  container: {...}, 
  item: {...} 
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

This rule is **non-negotiable**. Every time you write `const styles = {...}`, you must replace it with a specific name, otherwise loading multiple components will throw errors all over the stack.

### Rule 2: Scope is not shared — you must export manually

**Key insight**: each `<script type="text/babel">` is compiled independently by Babel, and **scope is not shared between them**. The `Terminal` component you defined in `components.jsx` is **undefined by default** in `pages.jsx`.

**Solution**: at the end of every component file, export the components/utilities you want to share onto `window`:

```jsx
// End of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // List everything you want to use elsewhere
});
```

Then `pages.jsx` can use `<Terminal />` directly, because JSX will look it up at `window.Terminal`.

### Rule 3: Don't use `scrollIntoView`

`scrollIntoView` pushes the entire HTML container upward and breaks the web harness layout. **Never use it**.

Alternatives:
```js
// Scroll to a specific position inside a container
container.scrollTop = targetElement.offsetTop;

// Or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling the Claude API (inside HTML)

Some native design-agent environments (such as Claude.ai Artifacts) ship with a config-free `window.claude.complete`, but most agent environments (Claude Code / Codex / Cursor / Trae / etc.) do **not** have it locally.

If your HTML prototype needs to call an LLM for the demo (e.g. building a chat interface), you have two options:

### Option A: Don't actually call — use a mock

Recommended for demo scenarios. Write a fake helper that returns a preset response:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // simulate latency
    return "This is a mock response. Replace with the real API for production.";
  }
};
```

### Option B: Really call the Anthropic API

You'll need an API key, and the user must paste their own key into the HTML for it to run. **Never hard-code the key in the HTML**.

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: calling the Anthropic API directly from a browser hits CORS issues. If the user's preview environment doesn't support a CORS bypass, this route is closed. In that case, use Option A's mock, or tell the user they need a proxy backend.

### Option C: use the agent-side LLM capability to generate mock data

If the demo is purely local, you can temporarily call the current agent session's LLM capability (or a multi-model skill the user has installed) to generate mock response data, and then hard-code it into the HTML. That way, the HTML at runtime depends on no API at all.

## Typical HTML Starter Template

Copy this template as the skeleton for a React prototype:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body { 
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- Main entry point -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ You defined `const styles` in one file and another file overwrote it. Rename each one to something specific.

**`Terminal is not defined`**
→ Cross-file references don't share scope. Add `Object.assign(window, {Terminal})` at the end of the file that defines `Terminal`.

**Whole page is blank, no console errors**
→ Most likely a JSX syntax error that Babel didn't surface in the console. Temporarily swap `babel.min.js` for the non-minified `babel.js` build — error messages are clearer.

**ReactDOM.createRoot is not a function**
→ Wrong version. Confirm you're on react-dom@18.3.1 (not 17 or anything else).

**`Objects are not valid as a React child`**
→ You rendered an object instead of JSX/string. Usually `{someObj}` should have been `{someObj.name}`.

## How to split files for larger projects

**A single file >1000 lines** is hard to maintain. How to split it up:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # Base elements: Button, Card, Badge...
│   ├── components.jsx      # Business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # Home page
│   │   ├── detail.jsx      # Detail page
│   │   └── settings.jsx    # Settings page
│   ├── router.jsx          # Simple router (React state switching)
│   └── app.jsx             # Entry component
└── data.js                 # mock data
```

Load them in order in HTML:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

**At the end of every file**, use `Object.assign(window, {...})` to export anything that needs to be shared.

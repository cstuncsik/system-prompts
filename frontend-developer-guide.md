# Senior Frontend Developer Guide

You are working with a senior frontend developer with 20+ years of experience who has deep expertise in JavaScript/TypeScript, CSS/SCSS, and HTML. They have witnessed the evolution from IE6 through the browser wars to modern browsers and understand web development fundamentals at a deep level.

## Developer Profile

- **Experience Level:** Senior (20+ years)
- **Core Technologies:** JavaScript/TypeScript, CSS/SCSS, HTML
- **Expertise Areas:** Language internals, runtime behavior, performance optimization, semantic markup
- **Browser Knowledge:** Cross-browser experience, modern web standards, no legacy support needed
- **Rendering Engine Understanding:** Deep knowledge of browser rendering pipeline and performant CSS animations

## Development Philosophy

### Progressive Enhancement Approach
- **HTML First:** Leverage semantic HTML to its full extent before considering other solutions
- **CSS Second:** When HTML capabilities are exhausted, use CSS to achieve the desired functionality and presentation
- **JavaScript Last:** Turn to JavaScript only when HTML and CSS cannot provide the solution
- **Layer Responsibility:** Each layer should handle what it does best - structure (HTML), presentation (CSS), behavior (JavaScript)

### Implementation Strategy
- Start every solution by exploring HTML semantic elements and native form controls
- Use CSS for layout, styling, simple interactions, and state changes before JavaScript
- Implement JavaScript functionality as progressive enhancement, not core dependency
- Ensure solutions degrade gracefully when JavaScript is disabled or fails to load
- Prioritize native browser capabilities over custom implementations

### Problem-Solving Methodology
- **Question the Requirement:** Always ask "what are we actually trying to achieve?" before jumping to solutions
- **Start with Constraints:** Consider semantic meaning, accessibility, and performance implications first
- **Explore Native Capabilities:** Research what the web platform already provides before building custom solutions
- **Validate Assumptions:** Test whether the "obvious" JavaScript solution is actually necessary
- **Consider Edge Cases:** Think about how the solution behaves with disabled JavaScript, slow connections, or assistive technologies

### Decision-Making Framework
- **Evaluate Trade-offs:** Weigh complexity vs. functionality vs. performance for each layer
- **Future-Proof Thinking:** Prefer solutions that align with web standards evolution
- **Maintenance Cost:** Consider who will maintain this code and how complex it is to debug
- **User Impact:** Prioritize solutions that work for the broadest range of users and devices
- **Team Knowledge:** Balance ideal solutions with team capabilities and knowledge

### Benefits of This Approach
- **Performance:** Native HTML/CSS solutions are faster than JavaScript equivalents
- **Accessibility:** Semantic HTML provides built-in accessibility features
- **Maintainability:** Less JavaScript means fewer potential points of failure
- **Resilience:** Solutions work across a broader range of devices and connection conditions
- **Standards Compliance:** Aligns with web platform design principles
- **Debugging Simplicity:** Issues are easier to isolate when layers have clear responsibilities
- **Cognitive Load:** Easier to reason about solutions when each technology handles its strengths

## Platform and API Mastery

- Fluent in native Web APIs: DOM, Fetch, IntersectionObserver, ResizeObserver, etc.
- Uses `addEventListener` with `passive`, `capture` where appropriate
- Skilled with low-level APIs: `requestAnimationFrame`, `canvas`, `Web Workers`, `BroadcastChannel`
- Deep understanding of native form elements, validation, and input quirks

## Browser and Device Awareness

- Expert in browser quirks across desktop and mobile
- Handles viewport units (`svh`, `lvh`), virtual keyboards, touch/pointer events
- Designs with `prefers-reduced-motion`, `prefers-color-scheme`, and accessibility settings in mind
- Understands lifecycle of single-page apps, bfcache behavior, unload pitfalls

## Security and Privacy

- Applies frontend security principles: XSS prevention, CSP, CORS, token handling
- Avoids fingerprinting and respects privacy-first practices

## Fonts and Internationalization

- Manages web font loading: `font-display`, fallbacks, ligatures, hyphenation
- Supports RTL/LTR layouts and handles Unicode edge cases cleanly

## DevTools and Diagnostics

- Proficient with browser DevTools for performance, paint/layout debugging, accessibility
- Builds minimal, isolated test cases for layout and rendering issues
- Prefers minimal reproducible examples when diagnosing browser issues
- Values root-cause analysis over surface fixes

## Standards and Tooling Philosophy

- Follows W3C and WHATWG specifications; uses MDN and official specs as reference
- Aware of TC39 proposals and tracks JavaScript evolution beyond Stage 2
- Prefers minimal, fast tooling (e.g., esbuild, Vite)
- Avoids bloated toolchains and unnecessary abstractions

## Collaboration and Review Preferences

- Appreciates precise PRs with focused changes and clear commit intent
- Reviews for architectural decisions, not just syntax or formatting
- Expects peers to understand rendering tradeoffs and DOM impacts
- Pragmatic about tech debt — flags it when meaningful, not theoretical

## Coding Style Preferences

### JavaScript/TypeScript

- Use modern ES6+ features freely (auto-polyfilling handles compatibility)
- Prefer functional programming patterns over OOP
- Avoid ES6 classes; use functions, objects, and prototypal inheritance
- Use arrow functions over function declarations
- Define functions as `const` variables, treating them as first-class values
- Naming: camelCase, short but meaningful, avoid verbose Java-style naming
- Use separate functions for event handlers, avoid inline handlers
- Prefer `Map` and `Set` over `Object`/`Array` when applicable, but consider performance
- Prefer array methods (`map`, `filter`, `reduce`) over loops, but weigh performance
- Use point-free style (e.g., `arr.filter(Boolean)`)
- Be cautious of multi-parameter function traps (e.g., `numbers.map(parseInt)`)
- Wrap async arrow functions in `try/catch`
- Use `return await` for async results to preserve stack trace
- Use Observables when more appropriate than Promises (streams, cancellation)
- Organize code in modules
- Comfortable with pre-class JS patterns
- **CSS-first mindset:** Exhaust CSS options before JavaScript

### Import Organization

- **Core packages first:** Framework core libraries (Vue, React) and Node.js built-ins (fs, path, etc.)
- **Node modules second:** Third-party packages from node_modules (pinia, lodash, axios, etc.)
- **Project files last:** Start with furthest (@/ alias paths), then relative imports (./), then same directory
- **Type imports:** Use `import type` for type-only imports, group with their respective categories
- **ESLint import/order compliance:** Follow standard import ordering rules for consistency and tooling compatibility

```typescript
// Example import order:
import { ref, computed } from 'vue';          // 1. Core frameworks
import { defineStore } from 'pinia';          // 2. Node modules
import { CONSTANTS } from '@/constants';      // 3. @ alias paths
import { helper } from '../utils/helper';     // 4. Relative paths
import type { IUser } from './types';        // 5. Same directory
```

### TypeScript Specific

- Prefer `type` over `interface`
- Use `satisfies` for type checks that preserve literals
- Avoid `as` assertions; prefer inference and validation
- Use template literal types over `string`
- Use built-in utility types (`Pick`, `Omit`, `Partial`) instead of custom
- Use type guard functions for runtime type checks
- Use `never` type to check exhaustiveness
- Use TypeScript where it adds real code quality

## CSS/SCSS

- Always exhaust CSS before reaching for JS
- Leverage modern CSS features
- Prefer native CSS variables
- Use SCSS for mixins, logic, structure when needed
- Inspired by ITCSS (inverted triangle CSS methodology)
- Aware of utility-first frameworks like Tailwind
- Auto-prefixing is assumed
- Use `:before`/`:after` instead of extra markup
- Prefer selectors like `:first-child`, `:empty` over JS manipulation
- Mobile-first, progressive enhancement
- Prefer `transition` for simple animations, `animation` for complex
- Use shorthand CSS properties when possible
- Prefer Flexbox; use Grid only if Flexbox isn't sufficient

## HTML

- Use minimal, semantic HTML
- Prefer semantic tags over `div`/`span`
- Avoid unnecessary elements — use pseudo-elements
- Accessibility is a priority: semantic structure, ARIA, keyboard nav, screen readers
- Optimize loading and performance
- Manages keyboard navigation, focus control, and live regions for dynamic UIs

## Communication Style

- Direct and technical — skip basics, use precise terminology
- Focus on implementations, not long-winded explanations
- Favor modern best practices, not legacy
- Prioritize clean, performant, maintainable code

## Legacy Support Philosophy

- Does not support legacy browsers (IE11 or similar) unless contractually required
- Assumes modern baseline (ES6+, flexbox, CSS variables, fetch)

## Code Expectations

- Deep understanding of closures, prototypes, event loop, etc.
- No need to explain CSS fundamentals
- Familiar with the browser rendering pipeline and paint/composite triggers
- Always consider CSS first for UI/UX
- Accessibility should be baked into all solutions
- Provide rationale only when necessary
- Use semantic, minimal HTML
- Favor CSS pseudo-elements/selectors over JS DOM manipulation
- Animation performance awareness: GPU acceleration, transform/opacity, render blocking

**Remember**: This developer values efficiency, modern approaches, and clean code. Respect their expertise while delivering practical, well-crafted solutions.

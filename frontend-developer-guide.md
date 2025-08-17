# Senior Frontend Developer Guide

You are working with a senior frontend developer with 20+ years of experience who has deep expertise in JavaScript/TypeScript, CSS/SCSS, and HTML. They have witnessed the evolution from IE6 through the browser wars to modern browsers and understand web development fundamentals at a deep level.

This developer strongly favors functional programming patterns over classical object-oriented programming, emphasizing pure functions, immutability, and composition over classes and inheritance.

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
- **Avoid "this" context unless absolutely necessary** - prefer pure functions with explicit parameters
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

#### JavaScript preferences

**Function Declarations - Preferred Patterns:**

```javascript
// AVOID: Function declarations with potential "this" confusion
function processData() {
  return this.data.filter(item => item.active); // "this" dependency
}

function UserService() {
  this.users = [];
  this.getActive = function() {
    return this.users.filter(user => user.active); // "this" context
  };
}

// PREFERRED: Const functions with explicit parameters
const processData = (data) => data.filter(item => item.active);

const createUserService = (initialUsers = []) => {
  let users = [...initialUsers];

  const getActive = () => users.filter(user => user.active);
  const addUser = (user) => { users = [...users, user]; };
  const updateUser = (id, updates) => {
    users = users.map(user => user.id === id ? { ...user, ...updates } : user);
  };

  return { getActive, addUser, updateUser };
};
```

**Loop Alternatives - Array Method Composition:**

```javascript
// AVOID: Traditional loops
const processUsers = (users) => {
  const results = [];
  for (let i = 0; i < users.length; i++) {
    if (users[i].active && users[i].age >= 18) {
      results.push({
        ...users[i],
        displayName: `${users[i].firstName} ${users[i].lastName}`,
        canVote: true
      });
    }
  }
  return results.sort((a, b) => a.lastName.localeCompare(b.lastName));
};

// PREFERRED: Array method composition
const processUsers = (users) =>
  users
    .filter(user => user.active && user.age >= 18)
    .map(user => ({
      ...user,
      displayName: `${user.firstName} ${user.lastName}`,
      canVote: true
    }))
    .sort((a, b) => a.lastName.localeCompare(b.lastName));

// PREFERRED: Function composition with reusable utilities
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

const isEligibleUser = (user) => user.active && user.age >= 18;
const addDisplayName = (user) => ({
  ...user,
  displayName: `${user.firstName} ${user.lastName}`
});
const addVotingStatus = (user) => ({ ...user, canVote: true });
const sortByLastName = (users) => users.sort((a, b) => a.lastName.localeCompare(b.lastName));

const processUsers = pipe(
  users => users.filter(isEligibleUser),
  users => users.map(addDisplayName),
  users => users.map(addVotingStatus),
  sortByLastName
);
```

**Event Handling - Pure Function Patterns:**

```javascript
// AVOID: "this" context and inline handlers
class ComponentOld {
  constructor() {
    this.count = 0;
    this.element = document.querySelector('.button');
    this.element.onclick = () => this.handleClick(); // "this" dependency
  }

  handleClick() {
    this.count++; // "this" context
    this.render();
  }
}

// PREFERRED: Pure functions with explicit state
const createCounter = (initialCount = 0) => {
  let count = initialCount;

  const increment = () => count++;
  const getCount = () => count;
  const reset = () => { count = initialCount; };

  return { increment, getCount, reset };
};

const handleButtonClick = (counter, renderFn) => () => {
  counter.increment();
  renderFn(counter.getCount());
};

// Usage: Explicit dependencies, no "this"
const counter = createCounter(0);
const button = document.querySelector('.button');
const render = (count) => { /* render logic */ };

button.addEventListener('click', handleButtonClick(counter, render));
```

**Data Transformation - Functional Pipelines:**

```javascript
// AVOID: Imperative mutations and loops
const processApiResponse = (response) => {
  const data = response.data;
  for (let i = 0; i < data.length; i++) {
    data[i].processed = true;
    data[i].timestamp = Date.now();
    if (data[i].status === 'pending') {
      data[i].status = 'active';
    }
  }
  return data.filter(item => item.active);
};

// PREFERRED: Functional transformation pipeline
const markAsProcessed = (item) => ({ ...item, processed: true, timestamp: Date.now() });
const activatePending = (item) => ({
  ...item,
  status: item.status === 'pending' ? 'active' : item.status
});
const isActive = (item) => item.active;

const processApiResponse = (response) =>
  response.data
    .map(markAsProcessed)
    .map(activatePending)
    .filter(isActive);

// ADVANCED: Generic transformation utilities
const transform = (transformations) => (data) =>
  transformations.reduce((result, fn) => result.map(fn), data);

const filter = (predicate) => (data) => data.filter(predicate);

const processApiResponse = pipe(
  response => response.data,
  transform([markAsProcessed, activatePending]),
  filter(isActive)
);
```

**Async Patterns - Pure Functions:**

```javascript
// AVOID: "this" context in async operations
class ApiServiceOld {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }

  async fetchUser(id) {
    const response = await fetch(`${this.baseUrl}/users/${id}`); // "this"
    return this.processResponse(response); // "this"
  }

  processResponse(response) {
    return response.json(); // method depends on "this" context
  }
}

// PREFERRED: Pure functions with explicit parameters
const createApiService = (baseUrl) => {
  const buildUrl = (endpoint) => `${baseUrl}${endpoint}`;

  const processResponse = async (response) => {
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  };

  const fetchUser = async (id) => {
    const response = await fetch(buildUrl(`/users/${id}`));
    return processResponse(response);
  };

  const fetchUsers = async (filters = {}) => {
    const queryString = new URLSearchParams(filters).toString();
    const response = await fetch(buildUrl(`/users?${queryString}`));
    return processResponse(response);
  };

  return { fetchUser, fetchUsers };
};

// Usage: No "this", explicit dependencies
const api = createApiService('https://api.example.com');
const user = await api.fetchUser('123');
```

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

#### Advanced String Type Patterns

**Prefer strict, composable string types over broad `string` type:**

```typescript
// Template literal types with generics for API endpoints
type ApiEndpoint<Version extends string, Resource extends string> =
  `/api/${Version}/${Resource}`;

type UserEndpoint = ApiEndpoint<'v1', 'users'>; // "/api/v1/users"

// BEM CSS class naming system
type BemClass<Block extends string, Element extends string, Modifier extends string = never> =
  [Modifier] extends [never]
    ? `${Block}__${Element}`
    : `${Block}__${Element}--${Modifier}`;

// Event naming conventions
type AppEvent<Module extends string, Action extends string> = `${Module}:${Action}`;
type UserEvent = AppEvent<'auth', 'login'>; // "auth:login"

// Template literal types for domain concepts
type ThemeToken = `${string}-${number}`; // "primary-500", "gray-100"
type EmailAddress = `${string}@${string}.${string}`;
type UserId = `user_${string}`;

// Environment variable patterns with constraints
const getFeatureFlag = (flag: `FEATURE_${Uppercase<string>}`) =>
  process.env[flag] === 'true';

// CSS custom property typing
type CssCustomProp = `--${Lowercase<string>}`;
const cssVar = (name: CssCustomProp) => `var(${name})`;

// Route parameter extraction
type RouteParams<T extends string> = T extends `${string}:${infer Param}/${infer Rest}`
  ? { [K in Param]: string } & RouteParams<Rest>
  : T extends `${string}:${infer Param}`
  ? { [K in Param]: string }
  : {};

type UserRouteParams = RouteParams<'/users/:id/posts/:postId'>;
// { id: string; postId: string }
```

**Benefits of this approach:**
- **Type Safety:** Catch invalid string patterns at compile time
- **Developer Experience:** Autocomplete and IntelliSense for string patterns
- **Refactoring Safety:** Changes to string patterns are tracked by TypeScript
- **Documentation:** Types serve as documentation for expected string formats
- **Composition:** Build complex types from simple, reusable primitives

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

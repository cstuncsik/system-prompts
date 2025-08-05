# Vue 3 Testing Developer Guide

**This extends the [Senior Frontend Developer Guide](frontend-developer-guide.md), [Vue 3 Developer Guide](vue3-developer-guide.md), and [Testing Developer Guide](testing-developer-guide.md). Import all four guides together.**

You are working with a Vue 3 testing expert who combines comprehensive testing strategies with Vue-specific patterns and tools. This developer has deep experience testing complex Vue applications with sophisticated component architectures and state management patterns.

## Vue 3 Testing Expertise Profile

- **Testing Framework:** @testing-library/vue for all component testing
- **Component Strategy:** Smart/dumb component testing with strategic mocking
- **Store Testing:** @pinia/testing integration with mockedStore helper functions
- **User Interactions:** @testing-library/user-event for all user interactions (not fireEvent)
- **Query Strategy:** Behavior-focused queries using @testing-library principles

## Vue Component Testing Strategy

### Smart vs Dumb Component Testing
- **Smart Components (Containers):** Route-level components (pages/views) responsible for data loading, business logic, and event coordination
- **Dumb Components (Presentational):** Pure UI components that receive props and emit events without business logic
- **Smart Component Testing:** Mock child components with simple event emitters to test business logic and data flow
- **Dumb Component Testing:** Render with real children unless third-party UI libraries produce inaccessible markup

### Component Mocking Strategies
```typescript
// Smart component test - mock children
import UserPage from '@/views/UserPage.vue';
import { vi } from 'vitest';

// Mock child components
vi.mock('@/components/UserList.vue', () => ({
  default: {
    name: 'UserList',
    template: '<div data-testid="user-list" @click="$emit(\'userSelected\', mockUser)"></div>'
  }
}));

test('handles user selection from child component', async () => {
  const { getByTestId } = render(UserPage);
  await userEvent.click(getByTestId('user-list'));
  // Test smart component's reaction to child events
});
```

### Third-Party UI Library Handling
- **Accessible Libraries:** Render with real components when they produce semantic, accessible markup
- **Inaccessible Libraries:** Mock components that produce non-semantic markup or interfere with @testing-library queries
- **Strategic Replacement:** Replace complex third-party components with simple event emitters for behavior testing

## @testing-library/vue Integration

### Testing Philosophy and Queries
- **Behavior Over Implementation:** Test user interactions and visible behavior, not Vue internals
- **Accessibility-First Queries:** Prefer semantic queries that work with assistive technologies
- **Query Strategy:** Use `query*` functions for testing element absence, `get*` for elements that should exist
- **User-Centric Testing:** Focus on what users see and interact with rather than component implementation details

### User Interaction Testing
- **@testing-library/user-event:** Use user-event for ALL user interactions (clicks, typing, keyboard events)
- **Realistic User Simulation:** user-event provides more realistic user interaction simulation than fireEvent
- **Async User Actions:** All user-event actions are async and should be awaited

### Query and Interaction Patterns
```typescript
import userEvent from '@testing-library/user-event';

// Preferred: Semantic queries
getByRole('button', { name: 'Save User' });
getByLabelText('Email Address');
getByText('Welcome, John Doe');

// User interactions with user-event (NOT fireEvent)
test('handles user interactions', async () => {
  const { getByRole, getByLabelText } = render(Component);

  // Click interactions
  await userEvent.click(getByRole('button', { name: 'Submit' }));

  // Typing interactions
  await userEvent.type(getByLabelText('Email'), 'user@example.com');

  // Clear input
  await userEvent.clear(getByLabelText('Email'));

  // Keyboard interactions
  await userEvent.keyboard('{Enter}');
});

// For non-existence testing
expect(queryByText('Error message')).not.toBeInTheDocument();

// Avoid testing Vue internals
// DON'T: expect(wrapper.vm.internalProperty).toBe(true);
// DO: expect(getByText('Success message')).toBeInTheDocument();
```

## Custom Render Functions and Setup

### Global Plugin Configuration
- **Custom Render Function:** Create render helper that sets up global plugins (Pinia, i18n, Vue Router)
- **Consistent Setup:** Ensure all tests have access to global application context
- **Plugin Mocking:** Mock global plugins appropriately for isolated testing

```typescript
// Custom render function: test-utils/render.ts
import { render as testingLibraryRender } from '@testing-library/vue';
import { createPinia } from 'pinia';
import { createI18n } from 'vue-i18n';
import { createRouter, createMemoryHistory } from 'vue-router';

export const render = (component, options = {}) => {
  const pinia = createPinia();
  const i18n = createI18n({
    locale: 'en',
    messages: { en: {} }
  });
  const router = createRouter({
    history: createMemoryHistory(),
    routes: [{ path: '/', component: { template: '<div></div>' } }]
  });

  return testingLibraryRender(component, {
    global: {
      plugins: [pinia, i18n, router],
      ...options.global
    },
    ...options
  });
};
```

## Pinia Store Testing

### Store Testing Strategy
- **Isolated Store Testing:** Test stores independently using Pinia testing utilities
- **@pinia/testing Integration:** Use official Pinia testing helpers for store mocking in components
- **Helper Functions:** Create reusable helper functions following Pinia documentation recommendations

### Store Mocking in Components with mockedStore Helper
```typescript
import { vi } from 'vitest';
import { createTestingPinia } from '@pinia/testing';
import { mockedStore, type MockedStore } from '@/__tests__/utils';
import { useUsersStore } from '@/stores/users.store';
import { useUIStore } from '@/stores/ui.store';

// mockedStore helper function (from test utils)
/**
 * TypeScript helper for mocking Pinia store actions return value
 * @see https://pinia.vuejs.org/cookbook/testing.html#Mocking-the-returned-value-of-an-action
 */
export const mockedStore = <TStoreDef extends () => unknown>(
  useStore: TStoreDef,
): TStoreDef extends StoreDefinition<infer Id, infer State, infer Getters, infer Actions>
  ? Store<Id, State, Record<string, never>, {
      [K in keyof Actions]: Actions[K] extends (...args: infer Args) => infer ReturnT
        ? Mock<(...args: Args) => ReturnT>
        : Actions[K];
    }> & {
      [K in keyof Getters]: Getters[K] extends ComputedRef<infer T> ? T : never;
    }
  : ReturnType<TStoreDef> => {
  return useStore() as any;
};

// Component test with mockedStore pattern
let usersStore: MockedStore<typeof useUsersStore>;
let uiStore: MockedStore<typeof useUIStore>;

beforeEach(() => {
  renderComponent = createComponentRenderer(Component, {
    pinia: createTestingPinia(),
  });

  usersStore = mockedStore(useUsersStore);
  uiStore = mockedStore(useUIStore);

  // Setup store state and mocked methods
  usersStore.currentUser = mockUser;
  usersStore.updateUser = vi.fn().mockResolvedValue(undefined);
});

test('component handles store actions', async () => {
  const { getByRole } = renderComponent();

  await userEvent.click(getByRole('button', { name: 'Update User' }));

  expect(usersStore.updateUser).toHaveBeenCalledWith(mockUser);
});
```

### Isolated Store Testing
```typescript
// Store unit test
import { setActivePinia, createPinia } from 'pinia';
import { useUserStore } from '@/stores/user.store';

beforeEach(() => {
  setActivePinia(createPinia());
});

test('store handles user login', async () => {
  const store = useUserStore();
  vi.mocked(userApi.login).mockResolvedValue({ user: mockUser });

  await store.login(credentials);

  expect(store.currentUser).toEqual(mockUser);
  expect(store.isAuthenticated).toBe(true);
});
```

## Composables Testing

### Composable Testing Strategy
- **Complexity-Based Decision:** Test composables separately when they contain significant logic
- **UI-Heavy Composables:** Test through component integration when primarily UI-focused
- **Isolation Testing:** Use Vue's testing utilities for composables with complex reactive logic

### Composable Testing Patterns
```typescript
import { nextTick } from 'vue';
import { useUserValidation } from '@/composables/useUserValidation';

test('user validation composable', async () => {
  const { isValid, errors, validateUser } = useUserValidation();

  validateUser({ email: 'invalid-email' });
  await nextTick();

  expect(isValid.value).toBe(false);
  expect(errors.value.email).toBe('Invalid email format');
});
```

## Internationalization Testing

### i18n Testing Strategies
- **Translation Key Strategy:** Mock translation functions to return translation keys for reliable testing
- **English Default Strategy:** Load English translations by default and query by translated text
- **Project-Specific Approach:** Adapt strategy based on project setup and requirements

### Translation Mocking Patterns
```typescript
// Translation key mocking approach
const mockT = vi.fn((key) => key);
vi.mock('vue-i18n', () => ({
  useI18n: () => ({ t: mockT })
}));

test('displays correct translation key', () => {
  const { getByText } = render(Component);
  expect(getByText('user.welcome.message')).toBeInTheDocument();
});

// English default approach
test('displays welcome message', () => {
  const { getByText } = render(Component);
  expect(getByText('Welcome to the application')).toBeInTheDocument();
});
```

## Router and Navigation Testing

### Router Mocking Strategy
- **Consistent Mocking:** Mock Vue Router in tests to prevent errors and warnings
- **Navigation Testing:** Test navigation behavior through route changes and programmatic navigation
- **Route-Dependent Components:** Test components that depend on route parameters or query strings

### Router Testing Patterns
```typescript
import { createRouter, createMemoryHistory } from 'vue-router';

const router = createRouter({
  history: createMemoryHistory(),
  routes: [
    { path: '/users/:id', component: UserDetail },
    { path: '/users', component: UserList }
  ]
});

test('navigates to user detail', async () => {
  await router.push('/users/123');
  const { getByText } = render(UserDetail, {
    global: { plugins: [router] }
  });

  expect(getByText('User Details: 123')).toBeInTheDocument();
});
```

## Test Data and Mock Factories

### Vue-Specific Test Data
- **Component Props Factories:** Create factories for complex component prop objects
- **Store State Factories:** Build realistic store state for testing
- **Event Payload Factories:** Generate consistent event payloads for component communication

```typescript
// Vue component test factories
export const createUserProps = (overrides = {}) => ({
  user: createUser(),
  isEditing: false,
  canDelete: true,
  ...overrides
});

export const createStoreState = (overrides = {}) => ({
  users: [createUser(), createUser()],
  loading: false,
  error: null,
  ...overrides
});

export const createComponentEvent = (type, payload = {}) => ({
  type,
  payload: { timestamp: Date.now(), ...payload }
});
```

## Advanced Vue Testing Patterns

### Slot and Scoped Slot Testing
- **Slot Content Testing:** Verify slot content renders correctly with provided data
- **Scoped Slot Testing:** Test scoped slots receive correct data and render appropriately
- **Dynamic Slot Testing:** Test conditional slot rendering based on component state

```typescript
test('renders scoped slot with user data', () => {
  const { getByText } = render(UserList, {
    slots: {
      user: '<template #user="{ user }"><span>{{ user.name }}</span></template>'
    },
    props: { users: [createUser({ name: 'John Doe' })] }
  });

  expect(getByText('John Doe')).toBeInTheDocument();
});
```

### Props and Events Testing
- **Props Validation:** Test component behavior with various prop combinations
- **Event Emission:** Verify components emit expected events with correct payloads
- **v-model Testing:** Test two-way binding behavior with defineModel or manual prop/emit patterns

```typescript
test('emits user-selected event with correct payload', async () => {
  const onUserSelected = vi.fn();
  const { getByRole } = render(UserCard, {
    props: {
      user: mockUser,
      onUserSelected
    }
  });

  await userEvent.click(getByRole('button', { name: 'Select User' }));

  expect(onUserSelected).toHaveBeenCalledWith(mockUser);
});
```

## Vue Testing Utilities and Helpers

### Custom Testing Utilities
- **Component Wrapper Helpers:** Create utilities for common component setup and interaction patterns
- **Async Testing Helpers:** Utilities for handling Vue's reactivity and async updates
- **Mock Setup Helpers:** Reusable functions for setting up common mocks (stores, router, i18n)

### Testing Environment Setup
- **Global Test Configuration:** Set up Vitest with appropriate Vue testing environment
- **Custom Matchers:** Create Vue-specific testing matchers for common assertions
- **Test Isolation:** Ensure proper cleanup between tests to prevent state leakage

## Communication Style

- **Vue-Specific Solutions:** Provide testing approaches that leverage Vue's testing ecosystem and best practices
- **Component Architecture Awareness:** Understand smart/dumb component patterns and test appropriately
- **Performance Considerations:** Balance comprehensive testing with fast test execution
- **Real-World Pragmatism:** Focus on testing patterns that work in complex, production Vue applications
- **Accessibility Integration:** Emphasize testing practices that verify accessibility and semantic correctness

**Remember**: This developer expects Vue 3 testing solutions that integrate seamlessly with modern Vue development patterns while maintaining the comprehensive coverage and performance standards established in the base testing philosophy.

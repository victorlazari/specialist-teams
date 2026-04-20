# 29 - Vitest Unit Testing Advanced

## Introduction

In a modern full-stack environment leveraging Next.js 16 with React 19, Tailwind CSS v4, shadcn/ui, Prisma ORM, Valkey cache, RabbitMQ queues, and tested with Vitest 4.1.3, mastering advanced unit testing techniques is critical for building resilient, maintainable, and high-quality applications. This comprehensive guide delves deeply into advanced Vitest unit testing practices, focusing on sophisticated mocking strategies for Prisma ORM, Valkey cache, and RabbitMQ queues, snapshot testing, code coverage optimization, testing custom React hooks, and verifying complex state management.

By the end of this documentation, you will have a robust understanding of how to architect your unit tests to maximize coverage and reliability while seamlessly integrating with your tech stack and design patterns.

---

## Table of Contents

| Section                                | Description                                                                        |
|--------------------------------------|------------------------------------------------------------------------------------|
| Advanced Mocking Strategies           | Handling Prisma, Valkey (ioredis), and RabbitMQ mocks with Vitest                  |
| Snapshot Testing in Vitest            | Techniques for snapshot testing React components, hooks, and serialized data      |
| Code Coverage Strategies              | Best practices to design tests that maximize meaningful code coverage              |
| Testing Custom React Hooks            | Approaches to test hooks including `useIdleLogout`, `use-mobile`                   |
| Testing Complex State Management      | Verifying state machines, context providers, and compound components              |
| Appendix: Utilities and Sample Code   | Code snippets and helper utilities for mocking, snapshotting, and coverage setup  |

---

## Advanced Mocking Strategies

Mocking in unit tests is essential to isolate the unit of work and avoid dependencies on external systems such as databases, caches, or messaging queues. In your stack, Prisma ORM, Valkey cache (Redis fork with ioredis), and RabbitMQ queues are critical external dependencies that require sophisticated mocking to enable deterministic and fast unit tests.

### Mocking Prisma ORM

Prisma 7.6.0 is a powerful ORM for PostgreSQL 16, enforcing strict type safety via generated client APIs. However, direct database calls during unit testing introduce latency and potential flakiness.

An effective mocking approach is to abstract Prisma client calls behind repository adapters following the Hexagonal Architecture pattern. This allows you to mock the adapter methods rather than Prisma directly.

For example, consider this Prisma repository interface:

```ts
// src/adapters/prismaUserRepository.ts
import { PrismaClient, User } from '@prisma/client';

export interface IUserRepository {
  findUserById(id: string): Promise<User | null>;
  createUser(data: Partial<User>): Promise<User>;
  // more methods...
}

export class PrismaUserRepository implements IUserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findUserById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async createUser(data: Partial<User>): Promise<User> {
    return this.prisma.user.create({ data });
  }
}
```

In unit tests, you can mock the repository interface rather than the Prisma client internals:

```ts
import { describe, it, expect, vi } from 'vitest';
import { IUserRepository } from '@/adapters/prismaUserRepository';
import { UserService } from '@/services/userService';

describe('UserService', () => {
  it('should return user by id', async () => {
    const mockUserRepo: IUserRepository = {
      findUserById: vi.fn().mockResolvedValue({ id: '123', email: 'test@example.com' }),
      createUser: vi.fn(),
    };

    const userService = new UserService(mockUserRepo);
    const user = await userService.getUserById('123');

    expect(user).toEqual({ id: '123', email: 'test@example.com' });
    expect(mockUserRepo.findUserById).toHaveBeenCalledOnce();
  });
});
```

This mocking technique avoids directly stubbing Prisma client methods, which can be complex due to Prisma’s internal caching and query building. Additionally, it aligns with the Ports & Adapters pattern by mocking ports (repositories) rather than adapters (ORM).

In rare cases where you want to mock Prisma client methods directly, Vitest's module mocking with `vi.mock()` can be used carefully:

```ts
vi.mock('@prisma/client', () => {
  const mPrisma = {
    user: {
      findUnique: vi.fn().mockResolvedValue({ id: '123', email: 'test@example.com' }),
      create: vi.fn(),
    },
  };
  return { PrismaClient: vi.fn(() => mPrisma) };
});
```

However, this is less recommended as it tightly couples tests to Prisma’s API surface and may break on Prisma upgrades.

### Mocking Valkey Cache (ioredis)

Valkey 8, a Redis fork, is accessed via `ioredis` in your application. Since Redis is an external cache store, mocking it during unit tests is essential for speed and isolation.

The strategy involves mocking the ioredis client instance methods such as `get`, `set`, `del`, and `expire`. Vitest’s mocking capabilities allow you to replace these with in-memory implementations or spies.

Here is a minimal example mocking an ioredis client:

```ts
import Redis from 'ioredis';
import { vi } from 'vitest';

vi.mock('ioredis', () => {
  return {
    default: vi.fn().mockImplementation(() => ({
      get: vi.fn((key) => Promise.resolve(mockCache[key] || null)),
      set: vi.fn((key, value) => {
        mockCache[key] = value;
        return Promise.resolve('OK');
      }),
      del: vi.fn((key) => {
        delete mockCache[key];
        return Promise.resolve(1);
      }),
      expire: vi.fn(() => Promise.resolve(1)),
      // implement other Redis commands if needed
    })),
  };
});

const mockCache: Record<string, string> = {};

// Example test using mocked Redis
describe('CacheService', () => {
  it('should set and get cache values', async () => {
    const redis = new Redis();
    await redis.set('foo', 'bar');
    const value = await redis.get('foo');
    expect(value).toBe('bar');
  });
});
```

For more sophisticated scenarios, you can implement a full in-memory Redis mock supporting TTLs and data structures if your cache usage is complex.

Alternatively, libraries such as `ioredis-mock` can be integrated with Vitest, though custom mocking provides more control in your CI/CD pipelines.

### Mocking RabbitMQ Queues

RabbitMQ 3 is used for asynchronous message processing and event-driven architecture in your system. Unit tests should not depend on a real message broker, so mocking the AMQP client is necessary.

If using the popular `amqplib` or similar client, mocking involves replacing the channel and connection methods such as `publish`, `consume`, and `ack`.

A lightweight mock could look like this:

```ts
import amqplib from 'amqplib';
import { vi } from 'vitest';

vi.mock('amqplib', () => {
  const publishMock = vi.fn();
  const consumeMock = vi.fn();
  const ackMock = vi.fn();

  const channelMock = {
    publish: publishMock,
    consume: consumeMock,
    ack: ackMock,
    assertQueue: vi.fn().mockResolvedValue({ queue: 'test-queue' }),
    // other channel methods
  };

  const connectionMock = {
    createChannel: vi.fn().mockResolvedValue(channelMock),
    close: vi.fn().mockResolvedValue(undefined),
  };

  return {
    connect: vi.fn().mockResolvedValue(connectionMock),
  };
});
```

This mock can be injected into your message queue adapter or service. You can assert that `publish` was called with expected arguments and simulate message consumption by invoking your consumer callbacks manually.

A key architectural pattern here is to abstract RabbitMQ interaction behind a facade or adapter, making the mock swap seamless and promoting testability.

---

## Snapshot Testing in Vitest

Snapshot testing is a vital technique to detect unintended UI regressions or changes in serialized data output. Vitest supports snapshot testing similar to Jest, enabling you to capture the rendered output of React components, serialized objects, or stringified results.

### Snapshot Testing React Components

Given your stack using React 19.2.4 and shadcn/ui compound components with Tailwind CSS v4 styling, snapshot testing ensures UI consistency across component updates.

Using `@testing-library/react` together with Vitest, you render the component and snapshot the container HTML:

```tsx
import { render } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import Button from '@/components/ui/Button';

describe('Button component', () => {
  it('matches the snapshot', () => {
    const { container } = render(<Button variant="primary">Click me</Button>);
    expect(container).toMatchSnapshot();
  });
});
```

Vitest stores snapshots in `__snapshots__` folders adjacent to test files. To update snapshots, run tests with `--update-snapshots`.

Because your components use Tailwind, be cautious with className variability due to dynamic styling. Consider normalizing or mocking Tailwind classes if snapshot noise arises from CSS class order or dynamic class generation.

### Snapshot Testing Custom Hooks

Testing custom React hooks like `useIdleLogout` or `use-mobile` can also benefit from snapshots, particularly when the hook returns complex objects or JSX.

Use `@testing-library/react`’s `renderHook` utility (via `@testing-library/react-hooks`) or a custom wrapper to render the hook and snapshot its output:

```ts
import { renderHook, act } from '@testing-library/react-hooks';
import { describe, it, expect } from 'vitest';
import { useIdleLogout } from '@/hooks/useIdleLogout';

describe('useIdleLogout hook', () => {
  it('returns initial state matching snapshot', () => {
    const { result } = renderHook(() => useIdleLogout({ timeoutMs: 300000 }));
    expect(result.current).toMatchSnapshot();
  });

  it('updates state on user activity', () => {
    const { result } = renderHook(() => useIdleLogout({ timeoutMs: 300000 }));
    act(() => {
      result.current.resetIdleTimer();
    });
    expect(result.current).toMatchSnapshot();
  });
});
```

This approach ensures your hook’s public API remains stable over time.

### Snapshot Testing Serialized Data

Beyond UI, snapshot testing serialized data such as API responses, Prisma query results, or cache contents is useful to guard against API contract changes or data structure drift.

For example:

```ts
describe('User API response', () => {
  it('matches the expected snapshot', () => {
    const userResponse = {
      id: 'uuid-123',
      email: 'user@example.com',
      roles: ['admin', 'user'],
      lastLogin: new Date('2024-06-10T12:00:00Z').toISOString(),
    };
    expect(userResponse).toMatchSnapshot();
  });
});
```

---

## Code Coverage Strategies

Code coverage metrics are key indicators of test completeness but can be misleading if misused. Vitest supports coverage reporting with `c8` and `istanbul` under the hood.

### Coverage Configuration

In your `vitest.config.ts`, enable and customize coverage like so:

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'c8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'src/mocks/**',
        'src/types/**',
        'src/**/*.d.ts',
        'src/**/__tests__/**',
        'src/**/index.ts',
      ],
      all: true,
      include: ['src/**/*.ts', 'src/**/*.tsx'],
      branches: 80,
      functions: 90,
      lines: 90,
      statements: 90,
    },
  },
});
```

### Targeted Coverage

Maximizing code coverage should focus on critical business logic, boundary conditions, and error paths. For example, repositories, services, and custom hooks should have near 100% coverage, while generated files and purely declarative UI components may have lower targets.

### Branch Coverage and Mutation Testing

Branch coverage ensures all conditional paths are tested. For example, in Prisma queries with optional filters, mock different scenarios to cover all branches.

Mutation testing tools (e.g., Stryker) can complement coverage by injecting faults and verifying tests catch them.

### Coverage of Async and Event-Driven Code

For RabbitMQ consumers or async cache invalidation, coverage requires triggering all async code paths. Vitest supports async test functions with `async/await`.

Ensure your tests await promises fully and use `vi.runAllTimers()` or `vi.useFakeTimers()` when testing time-dependent code (e.g., TTL in cache or retry logic in queues).

---

## Testing Custom React Hooks

Custom hooks encapsulate reusable logic and often manage state, side effects, and subscriptions. Testing them requires simulating the React lifecycle and verifying the hook’s behavior under different conditions.

### Testing `useIdleLogout` Hook

Suppose `useIdleLogout` logs out a user after inactivity. Testing involves simulating user events and timer expiration.

Example:

```tsx
import { renderHook, act } from '@testing-library/react-hooks';
import { vi } from 'vitest';
import { useIdleLogout } from '@/hooks/useIdleLogout';

describe('useIdleLogout', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should start idle timer on mount', () => {
    const { result } = renderHook(() => useIdleLogout({ timeoutMs: 1000 }));
    expect(result.current.isIdle).toBe(false);
  });

  it('should set isIdle to true after timeout', () => {
    const { result } = renderHook(() => useIdleLogout({ timeoutMs: 1000 }));
    act(() => {
      vi.advanceTimersByTime(1000);
    });
    expect(result.current.isIdle).toBe(true);
  });

  it('should reset timer on user activity', () => {
    const { result } = renderHook(() => useIdleLogout({ timeoutMs: 1000 }));
    act(() => {
      vi.advanceTimersByTime(900);
      result.current.resetIdleTimer();
      vi.advanceTimersByTime(900);
    });
    expect(result.current.isIdle).toBe(false);
  });
});
```

This test suite uses Vitest’s fake timers to fast-forward time and validate idle state transitions.

### Testing `use-mobile` Hook

A hook detecting mobile viewport might use `window.matchMedia`. Testing requires mocking the `matchMedia` API.

```ts
import { renderHook } from '@testing-library/react-hooks';

describe('use-mobile', () => {
  beforeAll(() => {
    window.matchMedia = vi.fn().mockImplementation(query => ({
      matches: query === '(max-width: 767px)',
      media: query,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    }));
  });

  it('should return true for mobile screen', () => {
    const { result } = renderHook(() => useMobile());
    expect(result.current).toBe(true); // assuming viewport mocked as mobile
  });
});
```

Mocking browser APIs is essential for reliable client component hook testing.

---

## Testing Complex State Management

Your application employs patterns like State Machine, Context/Providers, Compound Components, and Facade. These introduce complexity that requires careful testing.

### Testing State Machines

Suppose you have a state machine implemented for user onboarding:

```ts
type OnboardingState = 'start' | 'fillProfile' | 'confirmEmail' | 'completed';

interface OnboardingContext {
  emailConfirmed: boolean;
  profileCompleted: boolean;
}

function onboardingReducer(state: OnboardingState, event: string): OnboardingState {
  switch (state) {
    case 'start':
      if (event === 'START') return 'fillProfile';
      break;
    case 'fillProfile':
      if (event === 'PROFILE_FILLED') return 'confirmEmail';
      break;
    case 'confirmEmail':
      if (event === 'EMAIL_CONFIRMED') return 'completed';
      break;
  }
  return state;
}
```

Testing this reducer entails verifying all transitions:

```ts
describe('onboardingReducer', () => {
  it('transitions from start to fillProfile on START event', () => {
    expect(onboardingReducer('start', 'START')).toBe('fillProfile');
  });

  it('transitions from fillProfile to confirmEmail on PROFILE_FILLED event', () => {
    expect(onboardingReducer('fillProfile', 'PROFILE_FILLED')).toBe('confirmEmail');
  });

  it('transitions from confirmEmail to completed on EMAIL_CONFIRMED event', () => {
    expect(onboardingReducer('confirmEmail', 'EMAIL_CONFIRMED')).toBe('completed');
  });

  it('returns current state for unknown event', () => {
    expect(onboardingReducer('start', 'UNKNOWN')).toBe('start');
  });
});
```

For complex state machines, consider using libraries like `xstate` and testing with their utilities.

### Testing Context Providers

Context providers such as `ThemeProvider` or `Toaster` wrap components and provide values via React context. Test the provider by rendering a consumer component and verifying context values.

Example:

```tsx
import { render } from '@testing-library/react';
import { ThemeProvider, useTheme } from '@/contexts/ThemeContext';
import { describe, it, expect } from 'vitest';

function Consumer() {
  const { theme } = useTheme();
  return <div data-testid="theme">{theme}</div>;
}

describe('ThemeProvider', () => {
  it('provides default theme', () => {
    const { getByTestId } = render(
      <ThemeProvider>
        <Consumer />
      </ThemeProvider>
    );
    expect(getByTestId('theme').textContent).toBe('light');
  });
});
```

### Testing Compound Components

Compound components (e.g., shadcn/ui primitives) expose a flexible API via React children and context. Test these by rendering the compound with various children and asserting behavior.

```tsx
import { render, fireEvent } from '@testing-library/react';
import { Tabs, TabList, Tab, TabPanel } from '@/components/ui/Tabs';

describe('Tabs compound component', () => {
  it('changes active panel on tab click', () => {
    const { getByText, queryByText } = render(
      <Tabs defaultValue="tab1">
        <TabList>
          <Tab value="tab1">Tab 1</Tab>
          <Tab value="tab2">Tab 2</Tab>
        </TabList>
        <TabPanel value="tab1">Content 1</TabPanel>
        <TabPanel value="tab2">Content 2</TabPanel>
      </Tabs>
    );

    expect(getByText('Content 1')).toBeVisible();
    expect(queryByText('Content 2')).not.toBeVisible();

    fireEvent.click(getByText('Tab 2'));
    expect(getByText('Content 2')).toBeVisible();
    expect(queryByText('Content 1')).not.toBeVisible();
  });
});
```

Testing compound components validates that context and internal state propagate correctly.

---

## Appendix: Utilities and Sample Code

### Utility for Mocking Prisma Client

```ts
import { vi } from 'vitest';

export function createPrismaMock() {
  return {
    user: {
      findUnique: vi.fn(),
      create: vi.fn(),
      update: vi.fn(),
      delete: vi.fn(),
    },
    post: {
      findMany: vi.fn(),
      create: vi.fn(),
    },
    // Add more models as needed
  };
}
```

### Utility for Mocking Redis (Valkey)

```ts
export class InMemoryRedisMock {
  private store = new Map<string, string>();

  async get(key: string): Promise<string | null> {
    return this.store.get(key) ?? null;
  }

  async set(key: string, value: string): Promise<'OK'> {
    this.store.set(key, value);
    return 'OK';
  }

  async del(key: string): Promise<number> {
    return this.store.delete(key) ? 1 : 0;
  }

  // Add expire, ttl, etc., if needed
}
```

### Sample Vitest Config for Coverage

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'c8',
      reporter: ['text', 'html'],
      exclude: ['src/mocks/**', 'src/**/__tests__/**'],
      all: true,
      include: ['src/**/*.{ts,tsx}'],
    },
  },
});
```

---

## Conclusion

Mastering advanced Vitest unit testing requires a deep understanding of your architecture, third-party dependencies, and React component design. By adopting well-structured mocking strategies for Prisma ORM, Valkey cache, and RabbitMQ queues, you ensure your tests are fast, deterministic, and isolated. Snapshot testing enhances UI and data regression detection, while code coverage strategies focus on meaningful completeness metrics. Testing custom hooks and complex state management solidifies your confidence in dynamic app behavior.

This guide equips you with the knowledge and practical techniques to implement robust unit tests aligned with your Next.js 16, React 19, Tailwind v4, and shadcn/ui application, ensuring reliability and maintainability in your CI/CD pipelines.

---

*End of Document*
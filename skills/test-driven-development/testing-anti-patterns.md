# Testing Anti-Patterns for Laravel

**Load this reference when:** writing or changing tests, adding mocks/fakes, or tempted to add test-only methods to production code.

## Overview

Tests must verify real behavior, not mock behavior. Mocks and fakes are a means to isolate, not the thing being tested.

**Core principle:** Test what the code does, not what the mocks do.

**Following strict TDD prevents these anti-patterns.**

## The Iron Laws

```
1. NEVER test mock/fake behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## Anti-Pattern 1: Testing Mock/Fake Behavior

**The violation:**

```php
// ❌ BAD: Testing that the fake was called, not real behavior
test('sends welcome email', function () {
    Mail::fake();

    $user = User::factory()->create();

    // Only tests that Mail::fake() recorded a call
    Mail::assertSent(WelcomeEmail::class);
});
```

**Why this is wrong:**

- You're verifying the fake recorded something, not that your code works
- Missing the actual action that triggers the email
- Test passes even if email logic is broken

**Your human partner's correction:** "Are we testing the behavior of a fake?"

**The fix:**

```php
// ✅ GOOD: Test the real action that should send email
test('user registration sends welcome email', function () {
    Mail::fake();

    $action = new RegisterUserAction();
    $user = $action->execute([
        'email' => 'test@example.com',
        'password' => 'password',
    ]);

    Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email);
    });
});
```

### Gate Function

```
BEFORE asserting on any fake:
  Ask: "Am I testing real application behavior or just fake existence?"

  IF testing fake existence without triggering real code:
    STOP - Add the action that should cause the side effect

  Test real behavior that produces the side effect
```

## Anti-Pattern 2: Test-Only Methods in Production

**The violation:**

```php
// ❌ BAD: resetState() only used in tests
class PaymentGateway
{
    private array $processedPayments = [];

    public function process(Payment $payment): void
    {
        $this->processedPayments[] = $payment;
        // ... actual processing
    }

    // Looks like production API but only for tests!
    public function resetState(): void
    {
        $this->processedPayments = [];
    }
}

// In tests
afterEach(fn () => app(PaymentGateway::class)->resetState());
```

**Why this is wrong:**

- Production class polluted with test-only code
- Dangerous if accidentally called in production
- Violates single responsibility
- Confuses object lifecycle with test cleanup

**The fix:**

```php
// ✅ GOOD: Use Laravel's container to handle test isolation
// PaymentGateway has no resetState() - stateless or use DI

// Option 1: Fresh instance per test (Laravel default with RefreshDatabase)
// Container automatically provides fresh instances

// Option 2: If state must persist, use test helper
// tests/Helpers/PaymentTestHelper.php
function resetPaymentGateway(): void
{
    app()->forgetInstance(PaymentGateway::class);
}

// In tests
afterEach(fn () => resetPaymentGateway());
```

### Gate Function

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Use Laravel's container, traits, or test helpers instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## Anti-Pattern 3: Mocking Without Understanding

**The violation:**

```php
// ❌ BAD: Mock prevents behavior test depends on
test('prevents duplicate orders', function () {
    // Mock prevents inventory check that test depends on!
    $this->mock(InventoryService::class)
        ->shouldReceive('reserve')
        ->andReturn(true);

    $action = new CreateOrderAction();

    $action->execute($orderData);
    $action->execute($orderData);  // Should throw - but won't!
});
```

**Why this is wrong:**

- Mocked method had side effect test depended on (inventory reservation)
- Over-mocking to "be safe" breaks actual behavior
- Test passes for wrong reason

**The fix:**

```php
// ✅ GOOD: Mock at correct level, preserve needed behavior
test('prevents duplicate orders', function () {
    // Only mock the external API call, not the whole service
    Http::fake([
        'inventory-api.com/*' => Http::response(['reserved' => true]),
    ]);

    $action = new CreateOrderAction();

    $action->execute($orderData);  // Inventory reserved in DB

    expect(fn () => $action->execute($orderData))
        ->toThrow(DuplicateOrderException::class);
});
```

### Gate Function

```
BEFORE mocking any class or method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real implementation have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (HTTP calls, external APIs)
    OR use Laravel fakes that preserve necessary behavior
    NOT the high-level service the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## Anti-Pattern 4: Incomplete Mocks/Fakes

**The violation:**

```php
// ❌ BAD: Partial mock - only fields you think you need
Http::fake([
    'api.stripe.com/charges' => Http::response([
        'id' => 'ch_123',
        'status' => 'succeeded',
        // Missing: amount, currency, metadata that downstream code uses
    ]),
]);

// Later: breaks when code accesses $response['amount']
```

**Why this is wrong:**

- **Partial mocks hide structural assumptions** - You only mocked fields you know about
- **Downstream code may depend on fields you didn't include** - Silent failures
- **Tests pass but integration fails** - Mock incomplete, real API complete
- **False confidence** - Test proves nothing about real behavior

**The Iron Rule:** Mock the COMPLETE data structure as it exists in reality, not just fields your immediate test uses.

**The fix:**

```php
// ✅ GOOD: Mirror real API response completely
Http::fake([
    'api.stripe.com/charges' => Http::response([
        'id' => 'ch_123',
        'object' => 'charge',
        'amount' => 5000,
        'currency' => 'usd',
        'status' => 'succeeded',
        'paid' => true,
        'metadata' => ['order_id' => 'ord_456'],
        'created' => 1234567890,
        // All fields real API returns
    ]),
]);
```

### Gate Function

```
BEFORE creating fake HTTP responses or mock return values:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## Anti-Pattern 5: Over-Mocking Laravel Facades

**The violation:**

```php
// ❌ BAD: Mocking Cache when you could use array driver
test('caches expensive computation', function () {
    Cache::shouldReceive('remember')
        ->once()
        ->with('expensive-key', 3600, Mockery::type('callable'))
        ->andReturn('cached-value');

    $result = $service->getExpensiveData();

    expect($result)->toBe('cached-value');
});
```

**Why this is wrong:**

- Testing Mockery behavior, not caching behavior
- Doesn't verify the callable actually works
- Brittle - breaks if implementation changes cache key format

**The fix:**

```php
// ✅ GOOD: Use real cache with array driver (Laravel test default)
test('caches expensive computation', function () {
    // First call computes
    $result1 = $service->getExpensiveData();

    // Second call returns cached
    $result2 = $service->getExpensiveData();

    expect($result1)->toBe($result2);
    expect(Cache::has('expensive-key'))->toBeTrue();
});
```

## Anti-Pattern 6: Testing Eloquent Instead of Business Logic

**The violation:**

```php
// ❌ BAD: Testing that Eloquent works
test('user has many orders', function () {
    $user = User::factory()->has(Order::factory()->count(3))->create();

    expect($user->orders)->toHaveCount(3);
    expect($user->orders->first())->toBeInstanceOf(Order::class);
});
```

**Why this is wrong:**

- Tests Laravel's relationship system, not your code
- Eloquent relationships are already tested by Laravel
- Adds no value, creates maintenance burden

**The fix:**

```php
// ✅ GOOD: Test business logic that USES relationships
test('user total spent calculates from all orders', function () {
    $user = User::factory()
        ->has(Order::factory()->state(['total' => 5000]))
        ->has(Order::factory()->state(['total' => 3000]))
        ->create();

    expect($user->totalSpent())->toBe(8000);
});
```

## Anti-Pattern 7: Integration Tests as Afterthought

**The violation:**

```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**Why this is wrong:**

- Testing is part of implementation, not optional follow-up
- TDD would have caught this
- Can't claim complete without tests

**The fix:**

```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

## Laravel-Specific Guidance

### When to Use Laravel Fakes vs Mocks

| Scenario                           | Use                                           |
| ---------------------------------- | --------------------------------------------- |
| Mail, Notifications, Events, Queue | Laravel Fakes (`Mail::fake()`)                |
| External HTTP APIs                 | `Http::fake()`                                |
| File uploads                       | `Storage::fake()`                             |
| Time-dependent code                | `$this->travel()` or `Carbon::setTestNow()`   |
| Complex internal services          | Prefer real implementation with test database |
| Third-party SDK classes            | Mockery (last resort)                         |

### Prefer Real Implementations

```php
// ❌ BAD: Mocking your own service
$this->mock(OrderService::class)
    ->shouldReceive('create')
    ->andReturn(new Order(['id' => 1]));

// ✅ GOOD: Use real service with test database
$order = app(OrderService::class)->create($orderData);
expect($order)->toBeInstanceOf(Order::class);
expect($order->exists)->toBeTrue();
```

### Database State Over Mocks

```php
// ❌ BAD: Mock repository to return specific data
$this->mock(UserRepository::class)
    ->shouldReceive('findActive')
    ->andReturn(collect([new User(['name' => 'Test'])]));

// ✅ GOOD: Create real data, test real queries
User::factory()->count(3)->create(['active' => true]);
User::factory()->count(2)->create(['active' => false]);

$activeUsers = app(UserRepository::class)->findActive();

expect($activeUsers)->toHaveCount(3);
```

## When Mocks Become Too Complex

**Warning signs:**

- Mock setup longer than test logic
- Mocking everything to make test pass
- `shouldReceive` chains spanning multiple lines
- Test breaks when mock changes

**Your human partner's question:** "Do we need to be using a mock here?"

**Consider:** Feature tests with real database often simpler than complex mocks

```php
// ❌ Complex mock setup
$this->mock(PaymentGateway::class, function ($mock) {
    $mock->shouldReceive('createCustomer')->andReturn(['id' => 'cus_123']);
    $mock->shouldReceive('createPaymentMethod')->andReturn(['id' => 'pm_456']);
    $mock->shouldReceive('attachPaymentMethod')->andReturn(true);
    $mock->shouldReceive('charge')->andReturn(['status' => 'succeeded']);
});

// ✅ Simpler: HTTP fake at boundary
Http::fake([
    'api.stripe.com/*' => Http::sequence()
        ->push(['id' => 'cus_123'])
        ->push(['id' => 'pm_456'])
        ->push(['status' => 'succeeded']),
]);
```

## TDD Prevents These Anti-Patterns

**Why TDD helps:**

1. **Write test first** → Forces you to think about what you're actually testing
2. **Watch it fail** → Confirms test tests real behavior, not fakes
3. **Minimal implementation** → No test-only methods creep in
4. **Real dependencies** → You see what the test actually needs before faking

**If you're testing fake behavior, you violated TDD** - you added fakes without watching test fail against real code first.

## Quick Reference

| Anti-Pattern                    | Fix                                           |
| ------------------------------- | --------------------------------------------- |
| Assert on fake without action   | Test the action that causes side effect       |
| Test-only methods in production | Move to test helpers or use container         |
| Mock without understanding      | Understand dependencies first, mock minimally |
| Incomplete HTTP fakes           | Mirror real API response completely           |
| Over-mock facades               | Use array/test drivers (cache, queue, etc.)   |
| Test Eloquent relationships     | Test business logic using relationships       |
| Tests as afterthought           | TDD - tests first                             |
| Over-complex mocks              | Use feature tests with real database          |

## Red Flags

- `Mail::assertSent()` without action that sends mail
- Methods only called in test files
- Mock setup is >50% of test code
- Test fails when you remove mock
- Can't explain why mock is needed
- Mocking "just to be safe"
- Mocking your own service classes
- Testing that Eloquent works

## The Bottom Line

**Mocks and fakes are tools to isolate external dependencies, not things to test.**

If TDD reveals you're testing fake behavior, you've gone wrong.

Fix: Test real behavior or question why you're mocking at all.

### Laravel's Testing Philosophy

Laravel provides excellent testing infrastructure:

- **RefreshDatabase** gives clean state per test
- **Factories** create realistic test data quickly
- **Fakes** isolate external services (mail, queue, HTTP)
- **Feature tests** exercise real code paths

Trust this infrastructure. Mock less. Test real behavior.

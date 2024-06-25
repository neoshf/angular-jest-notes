# Angular unit testing with Jest

## Setup and Config

https://medium.com/@megha.d.parmar2018/angular-unit-testing-with-jest-2023-2676faa2e564

## Default TestBed setup

```typescript
import { MyComponent } from './my.component';

describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [ MyComponent ]
    })
    .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## No TestBed

```typescript
import { MyComponent } from './my.component';

describe('MyComponent', () => {
  let component: MyComponent;

  beforeEach(() => {
    component = new MyComponent();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## Mock function

```typescript
// example of a http call
public getCustomers(): Observable<Customer[]> {
  return this.http.get<Customer[]>(`${this.url}/customer`);
}

// mock it
const customersService = {
  getCustomers: jest.fn().mockReturnValue(of([])),
}
```

- mock.calls: Returns an array of the calls done to the mock, each call is an array of the arguments passed to the mock.
- mock.results: Returns an array of the results of the calls done to the mock, each result is an object with the value and the status of the call.
- mockImplementation: Allows you to set the implementation of the mock.
- mockImplementationOnce: Allows you to set the implementation of the mock for a specific call.
- mockReturnValue: Allows you to set the return value of the mock.
- mockResolvedValue: Allows you to set the resolved value of the mock.
- mockRejectedValue: Allows you to set the rejected value of the mock.

```typescript
const customersService = {
  getCustomers: jest.fn().mockReturnValue(of([{ name: 'John' }])),
}

const serviceChooser = {
  getService: jest.fn().mockReturnValue(customersService),
}
```

### Different mock objects based on input

```typescript
const customersService = {
  getCustomers: jest.fn().mockImplementation((input) => {
    if (input === 'John') {
      return of([{ name: 'John' }]);
    } else {
      return of([{ name: 'Mary' }]);
    }
  }),
}
```

### Mock once

We could also add 'Once' to it, that way it would work as a mock just once, and then you would need to mock again.
That's also useful for cache testing, like imagine a scenario where the first time it shouldn't return something, but then the next one it should return something, we could do:

```typescript
const customersService = {
  getCustomers: jest.fn()
    .mockImplementationOnce(() => of([]))
    .mockImplementationOnce(() => of([{ name: 'John' }])),
}
```

### Async test with fakeAsync

```typescript
it('should validate getCustomer', fakeAsync(() => {
  //when: triggers the component method that calls the service
  tick(); // this will simulate the time passing
  //then: expect that any logic or treatment is done correctly
}));
```

- flush(): Simulates the passage of time until all pending asynchronous activities finish.
- flushMicrotasks(): Simulates the passage of time until all pending microtasks finish.
- tick(millis): Simulates the passage of time until all pending asynchronous activities finish. The microtasks queue is drained at the very start of this function and after any timer callback has been executed.
- tickMacroTasks(millis): Simulates the passage of time until all pending asynchronous activities finish. The microtasks queue is drained at the very start of this function and after any timer callback has been executed.
- discardPeriodicTasks(): Discard all remaining periodic tasks.
- flushPeriodicTasks(): Flushes all pending periodic tasks.

### Async test with async/await

```typescript
it('should validate getCustomer', async () => {
  //when: triggers the component method that calls the service
  await fixture.whenStable(); // this will wait for the promise to resolve
  //then: expect that any logic or treatment is done correctly
});
```

Because the async/await syntax is a way to write asynchronous code in a synchronous way, it doesn't work with all asynchronous code, like setTimeout, setInterval, etc.

```typescript
// won't work
public getWaitToShowSomething(): void {
  interval(1000).subscribe(() => {
    this.showSomething = true;
  });
}

it('should validate getWaitToShowSomething', async () => {
  const response = await component.getWaitToShowSomething();
  expect(response).toBe(true);
});

// will work

public getWaitToShowSomething(): Promise<boolean> {
  return new Promise((resolve) => {
    interval(1000).subscribe(() => {
      this.showSomething = true;
      resolve(true);
    });
  });
}

it('should validate getWaitToShowSomething', async () => {
  const response = await component.getWaitToShowSomething();
  expect(response).toBe(true);
});
```

### Using done as a callback

The done callback is a function that you can use to tell the test that it's done, so it can continue to the next test, it's useful when you have a asynchronous code that you can't convert into a promise or observable.

```typescript
it('should validate getCustomer', (done) => {
  customersService.getCustomers().subscribe(() => {
    //then: expect that any logic or treatment is done correctly
    done();
  });
});
```

In that case we are using the done callback to tell the test that it's done, so it can continue to the next test.If the done callback is not called, the test will fail after 5000ms.

This scenario ensures that the classic false positive of the test is not happening, where the test passes because it is not waiting for the asynchronous code to finish.

### Clear expectations

```typescript
it('should validate getCustomer', () => {
  //given: mock the return of the moethod getCustomer
  customersService.getCustomers = jest.fn().mockReturnValue(of([{ name: 'John' }]));

  //when: triggers the component method that calls the service
  component.getCustomer();

  //then: expect that any logic or treatment is done correctly
  expect(customersService.getCustomers).toHaveBeenCalledTimes(1);
});
```

```typescript
it('should validate getCustomer', () => {
  //given: mock the return of the moethod getCustomer
  customersService.getCustomers = jest.fn().mockReturnValue(of([{ name: 'John' }]));

  //when: triggers the component method that calls the service
  component.getCustomer();

  //then: expect that any logic or treatment is done correctly
  expect(customersService.getCustomers).toHaveBeenCalledWith('John');
});
```

### Use stubs

```typescript
describe('MyActivitiesFacade', () => {
  let myActivitiesFacadeSUT: MyActivitiesFacade;
  const globalStateStub = {
    userId: () => '1',
  };
  const activitiesServiceStub = {
    getByUserId: jest.fn(),
    putActivity: jest.fn(),
  };

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        MyActivitiesFacade,
        {
          provide: ActivitiesService,
          useValue: activitiesServiceStub,
        },
        {
          provide: GlobalState,
          useValue: globalStateStub,
        },
      ],
    });
    myActivitiesFacadeSUT = TestBed.inject(MyActivitiesFacade);
  });

  it('should be instantiable', () => {
    expect(myActivitiesFacadeSUT).toBeTruthy();
  });

  it('should call the service to get the activities', () => {
    // Arrange
    activitiesServiceStub.getByUserId.mockReturnValue(of([]));
    const getByUserIdSpy = jest.spyOn(activitiesServiceStub, 'getByUserId');
    // Act
    myActivitiesFacadeSUT.getMyActivities();
    // Assert
    expect(getByUserIdSpy).toHaveBeenCalled();
  });

  it('should call the state to execute the command', () => {
    // Arrange
    activitiesServiceStub.getByUserId.mockReturnValue(of([]));
    const executeSpy = jest.spyOn(
      myActivitiesFacadeSUT.getMyActivitiesState,
      'execute'
    );
    // Act
    myActivitiesFacadeSUT.getMyActivities();
    // Assert
    expect(executeSpy).toHaveBeenCalled();
  });
});
```

### Shallow render

https://medium.com/@getsaf/why-shallow-rendering-is-import-in-angular-unit-tests-84569d571b72

Source: 
- https://www.linkedin.com/pulse/how-test-angular-components-using-jest-nice-easy-wagner-caetano/
- https://albertobasalo.medium.com/unit-testing-angular-with-jest-7de62ae2acd8

# Angular unit testing with Jest

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

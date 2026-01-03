- Mixin 패턴은 여러 클래스의 기능(속성과 메서드)을 하나의 클래스에 조합하여 재사용성을 높이는 프로그래밍 패턴입니다. Mixin은 기존의 상속 방식을 대체하거나 보완하여 여러 클래스의 기능을 결합할 수 있게 해줍니다.
- TypeScript에서는 다중 상속을 지원하지 않기 때문에, Mixin 패턴을 사용하면 클래스 간의 기능을 동적으로 결합하는 방법으로 다중 상속과 비슷한 효과를 얻을 수 있습니다.
## **Mixin 패턴의 특징**
1. **코드 재사용성 증가**: 여러 클래스에서 공통적으로 사용하는 기능을 분리하여 다른 클래스와 쉽게 결합할 수 있습니다.
2. **다중 상속 대안**: TypeScript가 다중 상속을 지원하지 않지만, Mixin 패턴으로 여러 클래스를 조합할 수 있습니다.
3. **동적 클래스 조합**: 런타임 또는 설계 시점에 필요한 기능을 클래스에 동적으로 추가할 수 있습니다.

```node
/*
type Constructor<T = {}>는 TypeScript에서 **생성자 타입**을 정의하는 유틸리티 타입입니다.
이를 통해 클래스를 함수처럼 타입으로 다루고, 동적으로 클래스의 동작을 확장하거나 조합할 때 유용하게 사용할 수 있습니다.
*/
type Constructor<T = {}> = new (...args: any[]) => T;

function Mixin1<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    mixin1Method() {
      console.log('Mixin1 method');
    }
  };
}

function Mixin2<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    mixin2Method() {
      console.log('Mixin2 method');
    }
  };
}

class BaseClass {
  baseMethod() {
    console.log('Base method');
  }
}

class CombinedClass extends Mixin1(Mixin2(BaseClass)) {}

const instance = new CombinedClass();
instance.baseMethod();  // Base method
instance.mixin1Method(); // Mixin1 method
instance.mixin2Method(); // Mixin2 method
```

### **장점**

1. **유연한 기능 추가**: 클래스에 필요한 기능을 유연하게 추가할 수 있습니다.
2. **다중 상속 대체**: 단일 상속 제한을 우회하여 여러 클래스의 기능을 조합할 수 있습니다.
3. **코드 재사용성**: 공통 로직을 Mixin으로 분리하면 여러 클래스에서 재사용 가능합니다.
### **단점**
4. **클래스 복잡성 증가**: 너무 많은 Mixin을 조합하면 클래스의 구조가 복잡해질 수 있습니다.
5. **타입 안전성 감소**: TypeScript에서는 Mixin이 런타임 동작과 관련되기 때문에, 타입 정보를 명시적으로 관리해야 할 때가 있습니다.
6. **디버깅 어려움**: Mixin으로 구성된 클래스는 계층 구조가 명확하지 않아 디버깅이 어렵습니다.
## 상위 모듈은 하위 모듈의 것을 직접 가져오면안됨
- 구체적인 동작을 직접 구현하면 저수준모듈, 추상화된 로직을제공하면 고수준모듈
- 둘다 추상화에 의존해야 함
## 추상화는 세부사항에 의존해선 안됨
- 세부사항이 추상화에 의존해야함
## 클래스가 다른 클래스와 관계가 있으면 안됨
- 클래스가 다른 클래스의 작동방식을 많이 알고있음 안됨
- 종속성 (dependency) 또는 결합(coupling) 발생
- 종속성은 잠재적인 위험


## 안좋은 예시: 상위 모듈이 하위모듈을 직접 가져온다.
```js
// 저수준 클래스  
class Fan {  
    spin(): void {  
        console.log('Fan is spinning');  
    }  
  
    stop(): void {  
        console.log('Fan is stopping');  
    }  
}  
  
// 고수준 클래스  
class Switch {  
    private fan: Fan;  
  
    constructor(fan: Fan) {  
        this.fan = fan;  
    }  
  
    turnOn(): void {  
        //Fan class(저수준)코드가 Switch class(고수준)에서 그대로 실행되고 있다.  
        //Switch class(고수준)가 다른 클래스(Fan 이외의 기능)를 실행할 수 없다.  
        this.fan.spin();  
    }  
  
    turnOff(): void {  
        //Fan class(저수준)코드가 Switch class(고수준)에서 그대로 실행되고 있다.  
        //Switch class(고수준)가 다른 클래스(Fan 이외의 기능)를 실행할 수 없다.  
        this.fan.stop();  
    }  
}  
  
// 사용 예시  
const fan = new Fan();  
const fanSwitch = new Switch(fan);  
  
fanSwitch.turnOn();  // Fan is spinning  
fanSwitch.turnOff(); // Fan is stopping
```


## 좋은 예시: 상위 모듈이 interface를 거쳐서 하위 모듈을 사용한다.
```js
/*  
* 의존성 역전 법칙  
* 고수준 모듈이 저수준 모듈에 의존해서는 안된다.  
*  
* 구체적인 동작을 직접 구현하면 저수준 모듈, 추상화된 로직을 제공하면 고수준 모듈  
* */  
interface Switchable {  
    turnOn(): void;  
    turnOff(): void;  
}  
  
// 인터페이스를 구현하는 저수준 클래스  
// Fan이 Switchable 카테고리에 속하게 된다.  
class Fan implements Switchable {  
    turnOn(): void {  
        console.log('Fan is spinning');  
    }  
  
    turnOff(): void {  
        console.log('Fan is stopping');  
    }  
}  
  
// 고수준 클래스  
class Switch {  
    // Switchable 속성을 가진다.  
    // Switchable 속성을 지닌 어느 다른 클래스들도 다룰 수 있다.  
    private device: Switchable;  
  
    constructor(device: Switchable) {  
        this.device = device;  
    }  
  
    turnOn(): void {  
        //Switchable 인터페이스를 거쳐서 Switch, Fan 소통한다.  
        //Fan class 수정에 Switch class 영향 없음  
        this.device.turnOn();  
    }  
  
    turnOff(): void {  
        //Switchable 인터페이스를 거쳐서 Switch, Fan 소통한다.  
        //Fan class 수정에 Switch class 영향 없음  
        this.device.turnOff();  
    }  
}  
  
// 사용 예시  
const fan = new Fan();  
const fanSwitch = new Switch(fan);  
  
fanSwitch.turnOn();  // Fan is spinning  
fanSwitch.turnOff(); // Fan is stopping
```
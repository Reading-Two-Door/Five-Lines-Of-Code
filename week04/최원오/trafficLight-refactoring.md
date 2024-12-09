```TS
interface TrafficLightState {
  getColor(): string;
  check(car: Car): void;
  transitionTo(light: TrafficLight): void;
}

class TrafficLight {
  private state: TrafficLightState;

  constructor(initialState: TrafficLightState) {
    this.state = initialState;
  }

  setState(newState: TrafficLightState): void {
    this.state = newState;
  }

  check(car: Car): void {
    this.state.check(car);
  }

  change(): void {
    this.state.transitionTo(this);
  }

  getColor(): string {
    return this.state.getColor();
  }
}

class RedLight implements TrafficLightState {
  getColor(): string {
    return "red";
  }

  check(car: Car): void {
    car.stop();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(new GreenLight());
  }
}

class YellowLight implements TrafficLightState {
  getColor(): string {
    return "yellow";
  }

  check(car: Car): void {
    car.stop();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(new RedLight());
  }
}

class GreenLight implements TrafficLightState {
  getColor(): string {
    return "green";
  }

  check(car: Car): void {
    car.drive();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(new YellowLight());
  }
}

class Car {
  drive(): void {
    console.log("Driving");
  }

  stop(): void {
    console.log("Stopping");
  }
}

```

```TS

// 리팩터링 후
interface TrafficLightState {
  getColor(): string;
  check(car: Car): void;
  transitionTo(light: TrafficLight): void;
}

class TrafficLight {
  private state: TrafficLightState;
  private static states: Map<string, TrafficLightState> = new Map();

  static {
    TrafficLight.states.set("red", new RedLight());
    TrafficLight.states.set("yellow", new YellowLight());
    TrafficLight.states.set("green", new GreenLight());
  }

  constructor(initialState: TrafficLightState) {
    this.state = initialState;
  }

  setState(newState: TrafficLightState): void {
    this.state = newState;
  }

  getState(color: string): TrafficLightState {
    return TrafficLight.states.get(color)!;
  }

  check(car: Car): void {
    this.state.check(car);
  }

  change(): void {
    this.state.transitionTo(this);
  }

  getColor(): string {
    return this.state.getColor();
  }
}

class RedLight implements TrafficLightState {
  getColor(): string {
    return "red";
  }

  check(car: Car): void {
    car.stop();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(light.getState("green"));
  }
}

class YellowLight implements TrafficLightState {
  getColor(): string {
    return "yellow";
  }

  check(car: Car): void {
    car.stop();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(light.getState("red"));
  }
}

class GreenLight implements TrafficLightState {
  getColor(): string {
    return "green";
  }

  check(car: Car): void {
    car.drive();
  }

  transitionTo(light: TrafficLight): void {
    light.setState(light.getState("yellow"));
  }
}

class Car {
  drive(): void {
    console.log("Driving");
  }

  stop(): void {
    console.log("Stopping");
  }
}
```

## 리펙터링 요약

Context(TrafficLight)

- 모든 가능한 상태들을 알고 있고 (Map<string, TrafficLightState>) 상태 변경을 위한 인터페이스를 제공하지만 (change() 메서드) 구체적인 상태 전환 로직은 모르게 구현

각 State(RedLight 등)

- 자신의 다음 상태가 무엇인지 알고 있고 transitionTo 메서드를 통해 직접 상태 전환 로직을 구현

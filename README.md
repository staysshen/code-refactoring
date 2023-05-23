# code-refactoring

## Завдання 1. Рефакторинг на рівні окремих операторів
### Наступна функція перевіряє, чи немає підозрілих осіб у списку осіб, імена яких були "захаркоджені":

```
void checkSecurity(String[] people) {
  boolean found = false;
  for (int i = 0; i < people.Length; i++) {
    if (!found) {
      if (people[i].Equals(“Don”)) {
        sendAlert();
        found = true;
      }
      if (people[i].Equals(“John”)) {
        sendAlert();
        found = true;
      }
    }
  }
}
```

## Запахи коду:

1. Дублювання коду: Код виклику функції sendAlert() повторюється для кожного імені в списку people[i].Equals(). Це призводить до непотрібного дублювання коду.

2. Запах керування станом: Змінна found використовується для відстеження того, чи було знайдено підозрілу особу. Це може призвести до складності розуміння і підтримки коду.

## Рефакторинг

1. Використання циклу foreach: Замінимо цикл for на цикл foreach, щоб отримати простіший і більш зрозумілий код.
2. Використання колекції підозрілих осіб: Створимо окрему колекцію для підозрілих осіб і додаватимемо їх до цієї колекції, замість повторного виклику функції sendAlert().

``` 
void checkSecurity(string[] people) {
    List<string> suspiciousPeople = new List<string>() { "Don", "John" };
    foreach (string person in people)
    {
        if (suspiciousPeople.Contains(person))
        {
            sendAlert();
            break;
        }
    }
}
```


## Завдання 2. Рефакторинг на рівні даних

``` 
enum TransportType {
  eCar,
  ePlane,
  eSubmarine
};

class Transport {
  public:
    Transport(TransportType type) : m_type (type) {}
    int GetSpeed(int distance, int time) {
      if (time != 0) {
        switch(m_type) {
          case eCar:
            return distance/time;
          case ePlane:
            return distance/(time - getTakeOffTime() - getLandingTime());
          case eSubmarine:
            return distance/(time - getDiveTime() - getAscentTime());
        }
      }
    }
...
  private:
    int m_takeOffTime;
    int m_landingTime;
    int m_diveTime;
    int m_ascentTime;
    enum m_type;
};
``` 
## Запахи коду:

1. Оператор Switch: оператор switch, який використовується в методі GetSpeed, є запахом коду. Це порушує принцип «відкрито-закрито», оскільки щоразу, коли додається новий тип транспорту, оператор switch потрібно змінити. Це може призвести до проблем з обслуговуванням у міру зростання кодової бази.

2. Метод GetSpeed відповідає за виконання кількох обчислень на основі типу транспорту. Це порушує принцип єдиної відповідальності та робить метод надто складним. Краще було б делегувати обчислення окремим методам.

## Рефакторинг

1. Замість використання оператора switch ми можемо використовувати поліморфізм для інкапсуляції поведінки, специфічної для кожного типу транспорту. Цього можна досягти шляхом створення окремих класів для кожного типу транспорту, які успадковують загальний базовий клас або інтерфейс.
2. Ми можемо виділити обчислення для кожного типу транспорту в окремі методи. Це зробить код більш модульним і покращить читабельність.

```
class Transport {
public:
    virtual int GetSpeed(int distance, int time) = 0;
};

class Car : public Transport {
public:
    int GetSpeed(int distance, int time) override {
        if (time != 0) {
            return distance / time;
        }
        return 0;
    }
};

class Plane : public Transport {
public:
    int GetSpeed(int distance, int time) override {
        if (time != 0) {
            int takeOffTime = getTakeOffTime();
            int landingTime = getLandingTime();
            return distance / (time - takeOffTime - landingTime);
        }
        return 0;
    }
};

class Submarine : public Transport {
public:
    int GetSpeed(int distance, int time) override {
        if (time != 0) {
            int diveTime = getDiveTime();
            int ascentTime = getAscentTime();
            return distance / (time - diveTime - ascentTime);
        }
        return 0;
    }
};

// Usage:
Transport* transport;

switch (m_type) {
    case eCar:
        transport = new Car();
        break;
    case ePlane:
        transport = new Plane();
        break;
    case eSubmarine:
        transport = new Submarine();
        break;
    default:
        // Handle unknown transport type
        break;
}

int speed = transport->GetSpeed(distance, time);

// Clean up the allocated memory
delete transport;
```



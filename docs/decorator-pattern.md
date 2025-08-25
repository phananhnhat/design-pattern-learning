# Decorator Pattern

Là một structural pattern (mẫu cấu trúc)

Ý tưởng:
- Cho 1 interface (component) chung
- Các Decorator cũng implement interface đó, nhưng chúng bao bọc (wrap) một Component khác để mở rộng hành vi.

Dễ hình dung nhất: mua trà sữa và thêm topping (trân châu, thạch, kem cheese...). Ly trà sữa gốc không đổi, nhưng có thêm "chức năng".

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

```javascript
function coffee() {
    return { cost: 5, desc: "Coffee" };
}

// Decorator function
function withMilk(drink) {
    return {
        cost: drink.cost + 2,
        desc: drink.desc + ", Milk"
    };
}

function withSugar(drink) {
    return {
        cost: drink.cost + 1,
        desc: drink.desc + ", Sugar"
    };
}

// Demo
let myDrink = coffee();
myDrink = withMilk(myDrink);
myDrink = withSugar(myDrink);

console.log(myDrink.desc, "=", myDrink.cost);
```

```java
// Component interface
interface Drink {
    int cost();
    String desc();
}

// Concrete Component
class Coffee implements Drink {
    public int cost() { return 5; }
    public String desc() { return "Coffee"; }
}

// Base Decorator
abstract class DrinkDecorator implements Drink {
    protected Drink drink;
    public DrinkDecorator(Drink drink) {
        this.drink = drink;
    }
    public int cost() { return drink.cost(); }
    public String desc() { return drink.desc(); }
}

// Concrete Decorators
class MilkDecorator extends DrinkDecorator {
    public MilkDecorator(Drink drink) { super(drink); }
    public int cost() { return super.cost() + 2; }
    public String desc() { return super.desc() + ", Milk"; }
}

class SugarDecorator extends DrinkDecorator {
    public SugarDecorator(Drink drink) { super(drink); }
    public int cost() { return super.cost() + 1; }
    public String desc() { return super.desc() + ", Sugar"; }
}

// Demo
public class DecoratorDemo {
    public static void main(String[] args) {
        Drink myDrink = new Coffee();
        myDrink = new MilkDecorator(myDrink);
        myDrink = new SugarDecorator(myDrink);

        System.out.println(myDrink.desc() + " = " + myDrink.cost());
        // Output: Coffee, Milk, Sugar = 8
    }
}
```
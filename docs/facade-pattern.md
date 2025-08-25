# Facade Pattern

Mẫu thiết kế cấu trúc (structural design pattern) để ẩn đi sự phức tạp của nhiều module/hệ thống con, và cung cấp một interface đơn giản, thống nhất cho bên ngoài.

Ý tưởng chính:
- 👉 Ẩn đi sự phức tạp của nhiều hệ thống con (subsystems)
- 👉 Cung cấp một interface đơn giản, thống nhất cho client (người dùng code).

Giúp component gọn gàng hơn, chỉ làm việc với 1 "cửa" duy nhất thay vì chạm vào nhiều module con.

Mức độ khó (★☆☆☆☆) | Mức độ phổ biến (★★★★★)

Ví dụ: Custom hook trong React

```javascript
// hooks/useDashboardFacade.ts
import { useUser } from "./useUser";
import { useOrders } from "./useOrders";

export function useDashboardFacade(userId: string) {
  const user = useUser(userId);
  const orders = useOrders(userId);

  return {
    user,
    orders,
    isLoading: user.loading || orders.loading,
  };
}
```

```javascript
const Dashboard = ({ userId }: { userId: string }) => {
  const { user, orders, isLoading } = useDashboardFacade(userId);

  if (isLoading) return <p>Loading...</p>;

  return (
    <>
      <h1>{user.name}</h1>
      <ul>{orders.map(o => <li key={o.id}>{o.name}</li>)}</ul>
    </>
  );
};
```

Ví dụ: trong java

```java
class CPU {
    public void start() {
        System.out.println("CPU started");
    }
}

class Memory {
    public void load() {
        System.out.println("Memory loaded");
    }
}

class ComputerFacade {
    private CPU cpu;
    private Memory memory;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
    }

    public void start() {
        System.out.println("Starting computer...");
        cpu.start();
        memory.load();
        System.out.println("Computer ready!");
    }
}
```

```java
public class FacadeSimpleDemo {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();  // chỉ cần gọi 1 hàm duy nhất
    }
}
```

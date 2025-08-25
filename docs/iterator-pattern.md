# Iterator Pattern

Là một behavioral pattern (mẫu hành vi).
Cung cấp một cách duyệt tuần tự qua các phần tử trong một collection (danh sách, tập hợp, cây, …) mà không cần biết chi tiết cấu trúc bên trong của collection đó

Ý tưởng:
- Collection chỉ cần cung cấp Iterator.
- Iterator sẽ có các phương thức như: hasNext() – còn phần tử không? next() – lấy phần tử tiếp theo.

👉 Hình dung: Giống như bạn đọc sách bằng bookmark: bạn không cần biết sách đóng gáy thế nào, chỉ cần biết lật sang trang kế tiếp.

Mức độ khó (★★☆☆☆) | Mức độ phổ biến (★★★★★)

```javascript
// Tạo iterator cho 1 mảng
function createIterator(array) {
    let index = 0;
    return {
        next: () => {
            if (index < array.length) {
                return { value: array[index++], done: false };
            } else {
                return { done: true };
            }
        }
    };
}

const iterator = createIterator(["a", "b", "c"]);
let result = iterator.next();
while (!result.done) {
    console.log(result.value);
    result = iterator.next();
}
```
👉 Đây chính là nguyên tắc mà JS for...of và Generators dựa vào. Duyệt mảng mà ko cần biết bao nhiêu phần tử.
Cụ thể: Xem Symbol.iterator, Iterators and generators in JS

```java
import java.util.Iterator;
import java.util.List;
import java.util.ArrayList;

// Collection
class MyCollection<T> implements Iterable<T> {
    private List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
    }

    @Override
    public Iterator<T> iterator() {
        return items.iterator(); // dùng iterator có sẵn của List
    }
}

// Demo
public class IteratorDemo {
    public static void main(String[] args) {
        MyCollection<String> col = new MyCollection<>();
        col.add("A");
        col.add("B");
        col.add("C");

        for (String item : col) {
            System.out.println(item);
        }
        // Output: A B C
    }
}
```
👉 Trong Java, Iterator là interface chuẩn (hasNext(), next()), và Iterable có thể được duyệt bằng for-each.
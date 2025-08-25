# Composite Pattern

Là một structural pattern (mẫu cấu trúc), cho phép bạn xử lý các đối tượng đơn lẻ (leaf) và tổ hợp các đối tượng (composite) theo cùng một cách.
Nhờ đó ta có thể làm việc với một cây cấu trúc phân cấp một cách đồng nhất.

Điển hình trong: Menu/ Submenu, File/Folder trong hệ điều hành, UI Component Tree

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

```javascript
// Leaf node: một item riêng lẻ
function createLeaf(name) {
    return {
        type: "leaf",
        name,
        show: () => console.log("- " + name)
    };
}

// Composite node: có thể chứa nhiều node con
function createComposite(name) {
    const children = [];
    return {
        type: "composite",
        name,
        add: (child) => children.push(child),
        show: () => {
            console.log("+ " + name);
            children.forEach(c => c.show());
        }
    };
}

const file1 = createLeaf("File A.txt");
const file2 = createLeaf("File B.txt");
const folder = createComposite("Documents");
folder.add(file1);
folder.add(file2);
folder.show();
```

```java
import java.util.ArrayList;
import java.util.List;

interface FileSystemItem {
    void show();
}

// Leaf
class File implements FileSystemItem {
    private String name;
    public File(String name) {
        this.name = name;
    }
    @Override
    public void show() {
        System.out.println("- " + name);
    }
}

// Composite
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();

    public Folder(String name) {
        this.name = name;
    }

    public void add(FileSystemItem item) {
        children.add(item);
    }

    @Override
    public void show() {
        System.out.println("+ " + name);
        for (FileSystemItem child : children) {
            child.show();
        }
    }
}

// Demo
public class CompositeDemo {
    public static void main(String[] args) {
        File file1 = new File("File A.txt");
        File file2 = new File("File B.txt");
        Folder folder = new Folder("Documents");

        folder.add(file1);
        folder.add(file2);
        folder.show();
    }
}
```
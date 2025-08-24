# Adapter Pattern

Là một structural pattern (mẫu cấu trúc). Cho phép các interface/module không tương thích có thể làm việc cùng nhau

Ý tưởng chính:
- Cho phép các interface/module không tương thích có thể làm việc cùng nhau bằng cách dùng một Adapter (bộ chuyển đổi).

Giống như ổ cắm điện: Ổ cắm 3 chấu (Mỹ) vs phích cắm 2 chấu (Việt Nam). Adapter đứng giữa để tương thích.

Mức độ khó (★★☆☆☆) | Mức độ phổ biến (★★★★☆)

### Khi nào dùng?
- Khi bạn muốn sử dụng một class đã có sẵn nhưng interface của nó không khớp với interface bạn cần.
- Giúp tái sử dụng code cũ (legacy) hoặc thư viện bên ngoài mà không cần sửa code gốc.
- Khi bạn cần làm việc với nhiều class con có chức năng tương tự nhưng interface lại khác nhau.

### Ví dụ: JavaScript

Giả sử chúng ta có một API cũ trả về dữ liệu dưới dạng XML và một hệ thống mới chỉ làm việc với JSON. Adapter sẽ giúp chuyển đổi dữ liệu.

```javascript
// API cũ (Adaptee)
class OldCalculator {
    operations(term1, term2, operation) {
        switch (operation) {
            case 'add':return term1 + term2;
            case 'sub':return term1 - term2;
            default: return NaN;
        }
    }
}

// API mới (Target)
class NewCalculator {
    add (term1, term2) { return term1 + term2; }
    subtract (term1, term2) { return term1 - term2; }
}

// Adapter
class CalculatorAdapter {
    constructor() {
        this.calculator = new OldCalculator();
    }

    add(term1, term2) { return this.calculator.operations(term1, term2, 'add'); }

    subtract(term1, term2) { return this.calculator.operations(term1, term2, 'sub');}
}

// Hệ thống mới mong muốn sử dụng interface của NewCalculator
const oldCalc = new OldCalculator();
console.log("Old API add:", oldCalc.operations(10, 5, 'add')); // Cách dùng cũ

const adapter = new CalculatorAdapter();
// Adapter có interface giống hệt NewCalculator
console.log("Adapter add:", adapter.add(10, 5));
console.log("Adapter subtract:", adapter.subtract(10, 5));
```

### Ví dụ: Java

Trong Java, ví dụ kinh điển là chuyển đổi giữa các loại trình phát media khác nhau.

```java
// Target Interface: Giao diện mà client mong muốn sử dụng
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee Interface: Giao diện của đối tượng cần được "adapt"
interface AdvancedMediaPlayer {
    void playVlc(String fileName);
    void playMp4(String fileName);
}

// Concrete Adaptee 1
class VlcPlayer implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name: " + fileName);
    }
    @Override
    public void playMp4(String fileName) {
        // Do nothing
    }
}

// Concrete Adaptee 2
class Mp4Player implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        // Do nothing
    }
    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file. Name: " + fileName);
    }
}

// Adapter Class
class MediaAdapter implements MediaPlayer {
    AdvancedMediaPlayer advancedMusicPlayer;

    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer = new VlcPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer = new Mp4Player();
        }
    }

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer.playMp4(fileName);
        }
    }
}

// Client code
class AudioPlayer implements MediaPlayer {
    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        // Hỗ trợ sẵn mp3
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file. Name: " + fileName);
        }
        // Dùng adapter để hỗ trợ các định dạng khác
        else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media. " + audioType + " format not supported");
        }
    }
}

// --- Sử dụng ---
public class AdapterDemo {
    public static void main(String[] args) {
        AudioPlayer audioPlayer = new AudioPlayer();

        audioPlayer.play("mp3", "beyond the horizon.mp3");
        audioPlayer.play("mp4", "alone.mp4");
        audioPlayer.play("vlc", "far far away.vlc");
        audioPlayer.play("avi", "mind me.avi");
    }
}
```

### Lợi ích
- **Tăng khả năng tái sử dụng**: Cho phép sử dụng các class đã có mà không cần sửa đổi code của chúng.
- **Tách biệt code**: Client không cần biết chi tiết về cách adapter chuyển đổi interface.
- **Tuân thủ Single Responsibility Principle**: Logic chuyển đổi được gói gọn trong adapter, không làm "ô nhiễm" code của client hay adapter.


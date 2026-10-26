# NOX Runtime & Compiler Requirements for Rune (NOX_LIMITATIONS.md)

> **Hedef:** Rune kütüphanesinin tam ekran TUI (Terminal User Interface), etkileşimli klavye olay döngüsü (interactive event loop) ve maskeli parola girdisi (password echo suppression) yeteneklerini gerçek terminalde %100 yerel olarak çalıştırabilmesi için **Nox Runtime** (`noxrt.o`, `runtime/stdlib_shims/*.zig`) ve **Nox Standard Library** (`stdlib/nox/*`) katmanlarında yapılması gereken teknik eklemeler ve spesifikasyon.

---

## 1. Yönetici Özeti (Executive Summary)

Rune v1.4.4 itibarıyla; çift tamponlu diff renderer (`ScreenBuffer`), flex layout motoru (`Layout`), deklaratif widget'lar (`Block`, `Paragraph`, `TableWidget`, vb.), durum makineli ANSI kaçış sırası ayrıştırıcı (`KeyDecoder`), yığınlanmış olay kuyruğu (`InputQueue`) ve Elm/Bubbletea benzeri TUI çalışma ortamı (`TuiApp` / `AppRunner`) **%100 saf Nox** ile eksiksiz olarak inşa edilmiştir.

Ancak Nox standart kütüphanesinde terminal I/O ile ilgili temel işletim sistemi çağrıları henüz bulunmadığından:
1. **Klavye Girdisi:** Yalnızca `input() -> str` mevcut olup, satır tabanlıdır ve kullanıcı `Enter` (`\n`) tuşuna basana kadar tüm thread'i bloke eder. Ok tuşları (`↑`, `↓`), `Esc`, `Tab`, `Ctrl+C` gibi anlık tuş vuruşları yakalanamaz.
2. **Ekran Boyutu:** Terminalin sütun ve satır sayıları (`ioctl TIOCGWINSZ`) sorgulanamamaktadır.
3. **Stdout Akışı:** Yalnızca `print()` fonksiyonu mevcuttur ve her çıktının sonuna zorunlu olarak `\n` ekler; ham `write()` ve `flush()` çağrılamaz.
4. **TTY Kontrolü:** `isatty(0/1/2)` bulunmadığı için stdin/stdout'un gerçek bir terminal mi yoksa boru hattı (pipe/CI) mı olduğu işletim sisteminden sorgulanamaz.

Bu belge, Nox derleyicisi ve çalışma zamanına (`noxc` / `noxrt`) eklenecek fonksiyonların Zig ve Nox seviyesindeki imzalarını, POSIX (macOS/Linux) ve Windows gerçekleştirimlerini tanımlar.

---

## 2. İhtiyaç Duyulan Yeni Modül: `nox.terminal` (veya `nox.os` Genişletmesi)

En temiz mimari, bu fonksiyonların `stdlib/nox/terminal.nox` altında toplanması veya mevcut `stdlib/nox/os.nox` modülüne eklenmesidir.

### Önerilen Nox Standart Kütüphane API'si (`terminal.nox`)

```nox
# stdlib/nox/terminal.nox

class TerminalSize:
    cols: int
    rows: int

    def __init__(self: TerminalSize, cols: int, rows: int) -> None:
        self.cols = cols
        self.rows = rows

# 1. TTY & Yetenek Algılama
def is_tty(fd: int = 1) -> bool: ...

# 2. Terminal Boyutu (Sütun x Satır)
def get_size() -> TerminalSize: ...

# 3. Raw Mod (Karakter Karakter Anlık Okuma & No-Echo)
def enable_raw_mode() -> None: ...
def disable_raw_mode() -> None: ...
def is_raw_mode() -> bool: ...

# 4. Parola / Echo Kontrolü
def set_echo(enabled: bool) -> None: ...

# 5. Non-Blocking / Zaman Aşımlı Ham Girdi Okuma
def poll_input(timeout_ms: int) -> bool: ...
def read_raw(max_bytes: int = 256) -> str: ...

# 6. Satır Başı Eklemeden Ham Stdout Çıktısı & Boşaltma
def write_raw(content: str) -> None: ...
def flush() -> None: ...
```

---

## 3. Zig Runtime Gerçekleştirmesi (`runtime/stdlib_shims/terminal.zig`)

Nox çalışma zamanı Zig tabanlı olduğu için (`noxrt.o`), aşağıdaki sistem çağrıları C/POSIX ve Windows API'leri kullanılarak doğrudan Zig içerisinde uygulanabilir:

### 3.1. Raw Mod (`enable_raw_mode` / `disable_raw_mode`)

#### POSIX (macOS / Linux):
```zig
const std = @import("std");
const posix = std.posix;

var orig_termios: posix.termios = undefined;
var raw_mode_enabled: bool = false;

pub export fn nox_terminal_enable_raw_mode_raw() i32 {
    if (raw_mode_enabled) return 0;

    const fd = posix.STDIN_FILENO;
    orig_termios = posix.tcgetattr(fd) catch return -1;

    var raw = orig_termios;
    // Girdi bayrakları: yazılımsal akış kontrolünü ve CR->NL dönüşümünü kapat
    raw.iflag.BRKINT = false;
    raw.iflag.ICRNL = false;
    raw.iflag.INPCK = false;
    raw.iflag.ISTRIP = false;
    raw.iflag.IXON = false;

    // Çıktı bayrakları: ham çıktı (post-processing kapalı)
    raw.oflag.OPOST = false;

    // Kontrol bayrakları: 8-bit karakterler
    raw.cflag.CSIZE = .CS8;

    // Yerel bayraklar: echo kapalı, kanonik mod kapalı, sinyaller (SIGINT/SIGTSTP) kapalı
    raw.lflag.ECHO = false;
    raw.lflag.ICANON = false;
    raw.lflag.IEXTEN = false;
    raw.lflag.ISIG = false;

    // Non-blocking okuma zamanlayıcısı: VMIN = 0, VTIME = 0 (anında döner)
    raw.cc[@intFromEnum(posix.V.MIN)] = 0;
    raw.cc[@intFromEnum(posix.V.TIME)] = 0;

    posix.tcsetattr(fd, .FLUSH, raw) catch return -1;
    raw_mode_enabled = true;
    return 0;
}

pub export fn nox_terminal_disable_raw_mode_raw() i32 {
    if (!raw_mode_enabled) return 0;
    const fd = posix.STDIN_FILENO;
    posix.tcsetattr(fd, .FLUSH, orig_termios) catch return -1;
    raw_mode_enabled = false;
    return 0;
}

pub export fn nox_terminal_is_raw_mode_raw() i32 {
    return if (raw_mode_enabled) 1 else 0;
}
```

#### Windows:
```zig
const windows = std.os.windows;
var orig_console_mode: windows.DWORD = 0;

// Enable:
// GetConsoleMode(hStdin, &orig_console_mode);
// SetConsoleMode(hStdin, orig_console_mode & ~(ENABLE_ECHO_INPUT | ENABLE_LINE_INPUT | ENABLE_PROCESSED_INPUT));
// SetConsoleMode(hStdout, orig_out_mode | ENABLE_VIRTUAL_TERMINAL_PROCESSING);
```

---

### 3.2. Terminal Boyutu (`TIOCGWINSZ`)

#### POSIX:
```zig
pub export fn nox_terminal_get_width_raw() i32 {
    var ws: posix.winsize = undefined;
    const rc = std.c.ioctl(posix.STDOUT_FILENO, posix.T.IOCGWINSZ, @intFromPtr(&ws));
    if (rc == 0 and ws.ws_col > 0) {
        return @intCast(ws.ws_col);
    }
    return 80; // Güvenli yedek
}

pub export fn nox_terminal_get_height_raw() i32 {
    var ws: posix.winsize = undefined;
    const rc = std.c.ioctl(posix.STDOUT_FILENO, posix.T.IOCGWINSZ, @intFromPtr(&ws));
    if (rc == 0 and ws.ws_row > 0) {
        return @intCast(ws.ws_row);
    }
    return 24; // Güvenli yedek
}
```

#### Windows:
```zig
// GetConsoleScreenBufferInfo(hStdout, &csbi);
// width = csbi.srWindow.Right - csbi.srWindow.Left + 1;
// height = csbi.srWindow.Bottom - csbi.srWindow.Top + 1;
```

---

### 3.3. Non-blocking Stdin Okuma (`read_raw` & `poll_input`)

#### POSIX:
```zig
pub export fn nox_terminal_poll_input_raw(timeout_ms: i32) i32 {
    var pfd = [1]posix.pollfd{
        .{
            .fd = posix.STDIN_FILENO,
            .events = posix.POLL.IN,
            .revents = 0,
        },
    };
    const count = posix.poll(&pfd, timeout_ms) catch return -1;
    return if (count > 0 and (pfd[0].revents & posix.POLL.IN) != 0) 1 else 0;
}

// read_raw çağrısı: std.posix.read(posix.STDIN_FILENO, buffer)
// Okunan baytlar doğrudan Nox string nesnesi olarak döner (with_rt).
```

---

### 3.4. Ham Stdout Yazma & Boşaltma (`write_raw` & `flush`)

```zig
pub export fn nox_terminal_write_raw(ptr: [*]const u8, len: usize) void {
    const stdout = std.io.getStdOut();
    _ = stdout.write(ptr[0..len]) catch {};
}

pub export fn nox_terminal_flush_raw() void {
    // libc veya unbuffered flush
}
```

---

## 4. Rune Tarafındaki Karşılıklar (Nasıl Entegre Edilecek?)

Nox derleyicisine bu çağrılar eklendiği anda Rune tarafında yapılacak işlem son derece basittir:

1. **`AppRunner.run_interactive()`:**
   ```nox
   # tui_runtime.nox
   def run_interactive(self: AppRunner) -> None:
       try:
           nox.terminal.enable_raw_mode()
           self.driver.enter_alt_screen()
           self.driver.hide_cursor()
           self.app.init()
           self.render_frame()

           while self.app.running:
               # 1. Terminal boyutunu kontrol et (resize algılama)
               sz = nox.terminal.get_size()
               if sz.cols != self.driver.current_buffer.width or sz.rows != self.driver.current_buffer.height:
                   self.driver.resize(sz.cols, sz.rows)
                   self.step(make_resize_event(sz.cols, sz.rows))

               # 2. Non-blocking klavye girdisini oku
               if nox.terminal.poll_input(self.tick_rate_ms):
                   chunk: str = nox.terminal.read_raw(128)
                   self.queue_raw(chunk)

               self.process_input_queue()
               if not self.app.running:
                   break

               # 3. Tick döngüsü
               self.step(make_tick_event(self.tick_rate_ms))
       finally:
           self.driver.restore()
           nox.terminal.disable_raw_mode()
   ```

2. **`TerminalDriver.write_raw()`:**
   - `print(s + "\x1b[H")` hack'i kaldırılarak doğrudan `nox.terminal.write_raw(s)` ve `nox.terminal.flush()` çağrılacak.

3. **`prompt_password()`:**
   - `nox.terminal.set_echo(False)` kullanılarak şifre maskeleme (`****`) aktif edilecek.

4. **`detect_console_options()`:**
   - `is_tty(1)` ve `get_size()` kullanılarak gerçek terminal genişliği ve TTY durumu 0 konfigürasyonla tespit edilecek.

---

## 5. Doğrulama Testi (Proof-of-Concept)

Nox runtime'a bu fonksiyonlar eklendiğinde aşağıdaki minimal Nox betiği derlenip çalıştırılarak doğrulanabilir:

```nox
# test_terminal.nox
import nox.terminal
import nox.time

print("Testing raw mode... Press any key (e.g. arrow keys, 'q' to quit):")
nox.terminal.enable_raw_mode()
try:
    while True:
        if nox.terminal.poll_input(100):
            chunk: str = nox.terminal.read_raw(32)
            if len(chunk) > 0:
                if chunk == "q":
                    break
                print("Read: " + chunk)
        nox.time.sleep_ms(10)
finally:
    nox.terminal.disable_raw_mode()
    print("Raw mode disabled cleanly.")
```

---

## 6. Mevcut Durum

| Özellik | Nox Runtime (Şu Anda) | Rune Durumu | Hedef (Nox Runtime Güncellemesi Sonrası) |
| :--- | :--- | :--- | :--- |
| **Klavye Girdi Ayrıştırma** | Sadece `input()` (enter bekler) | Sentetik testler & `queue_raw` hazır | `poll_input` + `read_raw` ile %100 canlı TUI |
| **Terminal Boyutu** | Sadece `$COLUMNS` / `$LINES` env | 80x24 fallback | `get_size()` ile dinamik boyut |
| **Pencere Boyutlandırma** | `SIGWINCH` yok | `ResizeEvent` modeli hazır | Döngü içinde anlık adaptasyon |
| **Ham Stdout** | Sadece `print()` (zorunlu `\n`) | Cursor parking (`\x1b[H`) | `write_raw()` ile sıfır scroll riski |
| **Maskeli Parola** | Desteklenmiyor | `PromptError` fırlatıyor | `set_echo(False)` ile tam destek |

Sen bu fonksiyonları Nox derleyicisine / runtime'ına eklediğin anda haber verdiğinde, Rune kütüphanesini hemen yeni API'lere bağlayacak ve `v1.5.0` canlı interaktif TUI sürümünü çıkaracağız!

# Hello, Rustacean! Web Server with Rust
Nama  : Ilham Abdillah Alhamdi <br>
NPM   : 2206081194 <br>
Kelas : Advance Programming - A <br>

## Commit 1 Reflection : What is inside the `handle_connection` ?
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader: BufReader<&mut TcpStream> = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result: Result<String, std::io::Error>| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
    println!("Request: {:#?}", http_request);
}
```
Fungsi `handle_connection` berfungsi untuk menerima koneksi request TCP dari _client_. Koneksi _client_ tersebut direpresentasikan dalam objek _struct_ `TcpStream`. Kemudian, agar dapat diolah secara efisien, objek tersebut dibungkus oleh _struct_ `BufReader`. Lalu, objek `BufReader` tersebut dikonversi menjadi array/vector yang berisi baris-baris string dari koneksi _client_. Baris-baris tersebut juga difilter terlebih dahulu sehingga tidak ada baris yang kosong. Kemudian, baris-baris string tersebut dicetak ke dalam _console_. 

Dalam proses konversi tersebut, terdapat method `map` dan `take_while`. Kedua method tersebut menerima argumen berupa _closure_. Konsep _closure_ dapat ditemui dalam pembahasan _functional programming_. Singkatnya, _closure_ merupakan ekspresi _inline function_ yang dapat di-passing sebagai nilai variable atau argumen fungsi lain. Dalam Rust, _closure_ diimplementasikan menggunakan syntax `|arg: arg_type| {body/return_value}` atau `|arg: arg_type| return_value`. 
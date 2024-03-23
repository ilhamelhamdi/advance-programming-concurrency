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

## Commit 2 Reflection : Returning HTML
![Commit 2 screen capture](/assets/images/commit-2.png)
Setelah dimodifikasi, kini fungsi `handle_connection` tidak hanya dapat menerima koneksi request, tetapi juga dapat mengembalikan _response_ berupa format HTML kepada _client_ . Konten HTML tersebut dibaca dari file `hello.html` dengan menggunakan `fs::read_to_string` dan disimpan dalam variabel `contents` berupa String. 

Selain itu konten HTML, dalam _response body_ juga dituliskan _status line_ dan _content length_. _Status line_ diperlukan untuk memberikan HTTP _status code_ kepada _client_ sebagai sinyal berhasil atau tidaknya koneksi. Berikut merupakan beberapa status kode HTTP yang dapat digunakan : 2xx untuk OK, 3xx untuk Redirection, 4xx untuk Client Bad Request, dan 5xx untuk Server Error.

Adapun header `Content length: ` diperlukan agar _client_ dapat menguji validitas _response body_ dengan mengecek kesesuaian panjangnya. Kemudian _macro_ `format!` digunakan untuk melakukan _string interpolation_ untuk menggabungkan _status line_, header `Content length`, dan body dari HTML. Hasil interpolasi string ini kemudian dikirimkan kepada _client_ melalui koneksi TCP dengan menggunakan `stream.write_all()`.



## Commit 3 Reflection : Validating request and selectively responding
![Commit 3 screen capture](/assets/images/commit-3.png)

Pada commit ini, fungsi `handle_connection` dapat melakukan seleksi _request_ dari _client_. Apabila _client_ melakukan koneksi dengan _root path_, server mengembalikan `hello.html`. Jika selain itu, server mengembalikan _response_ `404.html`. Untuk melakukan seleksi _request_, server membaca baris pertama _request_ dengan menggunakan `buf_reader.lines().next().unwrap().unwrap()`. Berikut merupakan kodenya.

Sebelum refactoring
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

Dari kode di atas, kita dapat menemukan duplikasi kode, yaitu kode untuk membaca file dan membuat string _response_ serta mengirimkannya ke `stream`. Untuk itu, kita dapat melakukan refactoring sehingga mengurangi duplikasi kode.  

Setelah refactoring
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

 

## Commit 4 Reflection : Simulation slow response
![Commit 4 screen capture](/assets/images/commit-4.png)
Pada commit ini, server disimulasikan mengembalikan _response_ dengan sangat lambat. _Client_ akan menerima response yang lambat apabila melakukan koneksi dengan path `/sleep`. Namun, _response_ lambat juga tidak hanya dialami oleh _client_ yang mengakses `/sleep`, akan tetapi dialami oleh _client_ lainnya. Hal ini terjadi apabila salah satu _client_ mengakses `/sleep`. 

Karena server ini _single-threaded_, hanya satu _request_ yang akan dilayani dalam satu waktu. Saat path `/sleep` diakses, _thread_ di server akan melakukan sleep selama 10 detik. Jika hal itu terjadi, _request_ dari _client_ lain tidak akan dilayani selama _main thread_ tidak berjalan. Oleh karena itu, diperlukan _multi-threading_ untuk melayani beberapa _client_ sekaligus.



##  Commit 5 Reflection : Multithreaded Server
_Threadpool_ merupakan sekumpulan _thread_ yang telah diinisiasi dan siap untuk melayani tugas-tugas.  Ketika sebuah program menerima tugas baru, ia menugaskan salah satu dari thread-thread dalam pool untuk menangani tugas tersebut, dan thread tersebut akan menjalankan proses pemrosesan. Sementara itu, thread-thread lain dalam pool tetap tersedia untuk menangani tugas-tugas lain yang masuk, bahkan ketika thread pertama sedang sibuk menyelesaikan tugasnya. Ketika thread pertama selesai menyelesaikan tugasnya, ia kembali ke pool thread yang tersedia, siap untuk menangani tugas baru. Dengan penerapan thread pool, program dapat memproses tugas secara paralel.

Pada commit ini, implementasi _thread pool_ menggunakan struct `ThreadPool` dan`Worker` serta type `Job`. Struct `Threadpool` berfungsi untuk menyimpan dan menginisiasi seluruh objek worker sebanyak jumlah tertentu. Struct `Worker` berfungsi untuk mengirimkan kode tugas (`Job`) dari _thread pool_ ke suatu _thread_. Adapun _type_ `Job` merupakan suatu tipe data yang mengimplementasikan _traits_ yang dapat diterima oleh `thread::spawn`. Komunikasi antara `ThreadPool` dan `Worker` dilakukan dengan teknik _message passing_. Dalam hal ini, `ThreadPool` bertindak sebagai _sender_ dan `Worker` sebagai _receiver_. Ketika ada suatu _request_ masuk, `ThreadPool` mengirimkan _message_ berupa suatu `Job` ke `Worker` untuk kemudian dijalankan di dalam suatu _thread_.
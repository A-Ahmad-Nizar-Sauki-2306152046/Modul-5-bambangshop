# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. penggunaan single model struct sudah cukup karena kita hanya memiliki satu jenis entitas subscriber dengan struktur data dan perilaku (seperti menerima notifikasi) yang seragam. Penggunaan interface atau trait dalam arsitektur perangkat lunak sebenarnya ditujukan untuk mencapai polimorfisme yang semuanya perlu terikat pada satu kontrak implementasi metode pembaruan (update) yang sama. Karena aplikasi kita belum membutuhkan variasi fungsionalitas berlapis semacam itu, membuat trait saat ini justru akan menambah kompleksitas kode tanpa memberikan manfaat arsitektural yang signifikan

2. Penggunaan `DashMap` (map/dictionary) jauh lebih esensial dan rasional dibandingkan `Vec` (list) untuk kasus ini karena atribut yang unik seperti `id` dan `url` secara natural berfungsi layaknya primary key dalam operasi basis data. Jika kita menyimpan data menggunakan Vec, setiap operasi pencarian, modifikasi, atau penghapusan data spesifik akan memaksa program melakukan iterasi satu per satu (linear search) dengan kompleksitas O(n), yang akan sangat menurunkan performa ketika jumlah subscriber membengkak. Sebaliknya, struktur dictionary pada DashMap menggunakan mekanisme hashing yang memungkinkan pencarian dan manipulasi data berdasarkan key unik tersebut dieksekusi dengan kompleksitas waktu konstan atau rata-rata O(1)

3. Sebenarnya `DashMap` dan pola desain Singleton bukanlah dua konsep substitusi untuk dipilih salah satu, melainkan keduanya saling melengkapi dalam kode kita: makro `lazy_static` bertugas mengimplementasikan Singleton dengan memastikan hanya ada satu instance penyimpan data global di memori, sementara `DashMap` berfungsi untuk memastikan instance tunggal tersebut thread-safe. Karena kerangka kerja web backend beroperasi secara konkuren untuk menangani banyak rute permintaan (request) secara bersamaan, aturan borrow checker Rust yang sangat ketat melarang keras adanya global mutable state tanpa adanya mekanisme sinkronisasi untuk mencegah data race. Seandainya kita membuat Singleton manual yang hanya berisi `HashMap` standar, compiler Rust tetap akan memaksa kita untuk membungkus struktur data tersebut dengan lock seperti `Mutex` atau `RwLock`; sehingga penggunaan `DashMap` (yang di baliknya sudah mengimplementasi granular locking secara internal) sangat mempermudah pembuatan Singleton yang aman pada ekosistem multi-threading

#### Reflection Publisher-2

1. Pemisahan `Service` dan `Repository` dari entitas `Model` merupakan implementasi langsung dari SRP dalam desain perangkat lunak. Jika sebuah Model menanggung beban ganda sebagai representasi struktur data, pengatur logika bisnis, dan pengelola interaksi penyimpanan (database) sekaligus, struktur kode akan menjadi sangat kaku (tightly coupled) dan sulit untuk diuji (unit testing). Dengan memisahkannya, Model murni berfokus pada definisi struktur data, Repository bertugas secara eksklusif mengelola operasi CRUD (menjadi antarmuka dengan mekanisme penyimpanan data), dan Service bertindak sebagai orkestrator yang menangani logika bisnis utama; sehingga setiap komponen menjadi lebih terisolasi, mudah dipelihara, dan fleksibel terhadap perubahan eksternal (misalnya saat beralih teknologi basis data tanpa perlu menyentuh logika bisnis sama sekali).

2. Jika kita hanya mengandalkan Model tanpa pemisahan layer, interaksi antar model akan menciptakan kompleksitas yang sangat tinggi dan berpotensi menghasilkan anti-pattern berupa God Object atau spaghetti code. Sebagai contoh, model `Program` tidak hanya akan mendefinisikan data dirinya sendiri, tetapi juga harus memuat serangkaian logika kompleks untuk melacak dan memanggil seluruh `Subscriber`, merakit objek `Notification`, dan mengeksekusi pengiriman tersebut secara internal. Ketergantungan langsung antar model ini memicu efek domino, di mana modifikasi kecil pada struktur subscriber atau mekanisme notifikasi akan memaksa perombakan pada source code model `Program` itu sendiri, membuat sistem menjadi sangat rentan error dan menyulitkan kolaborasi tim seiring berkembangnya aplikasi.

3. Eksplorasi menggunakan Postman terbukti sangat krusial dalam memvalidasi fungsionalitas aplikasi backend saat ini karena memungkinkan simulasi dan inspeksi permintaan HTTP secara detail (seperti misal penambahan atau penghapusan subscriber via metode POST) tanpa perlu menunggu kesiapan antarmuka frontend. Untuk pengembangan Proyek Kelompok (Group Project) maupun proyek software engineering lainnya di masa depan, fitur Collections di Postman sangat menarik karena dapat digunakan sebagai dokumentasi API yang interaktif dan dapat dibagikan (di-share) agar seluruh anggota tim memiliki acuan endpoint yang sama. Selain itu, dukungan fitur Environment Variables untuk memudahkan perpindahan testing dari skala localhost ke production, serta kemampuan menyisipkan Automated Testing scripts untuk memverifikasi respons API secara otomatis, akan sangat menghemat waktu dalam fase Quality Assurance.

#### Reflection Publisher-3

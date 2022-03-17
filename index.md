# Dokumentasi Aplikasi j-lantah & mitra j-lantah

## Memulai

- [Inisialisasi flutter](#inisialisasi-flutter)
- [Inisialisasi firebase service](#inisialisasi-firebase-service)
  - [Notification Service](#notification-service)
  - [Location Service](#location-service)
- [Komponen Modular](#modular-basic)
- [Autentikasi](#autentikasi)
  - [Register](#register)
  - [Login](#login)
- [Data poin dan liter minyak](#data-poin-dan-liter-minyak)
- [Menampilkan data history](#menampilkan-data-history)
- [Menampilkan data notifikasi](#menampilkan-data-notifikasi)
- [Profile](#profile)

### Inisialisasi Flutter

Sebelum menjalankan beberapa command yang ada di flutter, pastikan sudah meng-install Android Studio & xcode beserta xcode developer tools

setelah itu selanjutnya buka command line / terminal di folder project j-lantah user / mitra j-lantah

lalu ketikan command berikut di cmd untuk menginstall semua package yang digunakan oleh project j-lantah user / mitra j-lantah :

```cmd
flutter pub get
```

setelah command di atas berhasil di eksekusi maka selanjutnya adalah menjalankan project di device android ataupun di emulator dengan mengetikan command:

```cmd
flutter run
```

jika project berhasil di jalankan maka nanti kita dapat melakukan refresh ataupun restart aplikasi dengan menekan tombol "r" untuk refresh dan "R" untuk restart, tekan tombol tersebut di dalam command line / terminal yang telah di gunakan untuk menjalankan aplikasi j-lantah user / mitra j-lantah

Selalu letakan baris kode berikut di dalam fungsi **main** di awal baris

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  //code berikutnya setelah baris ini
}
```

ini bertujuan untuk mengecek apakah semua komponen dari flutter sudah benar benar terinisialisasi.

### Inisialisasi firebase service

Pada file **main.dart** dapat ditemukan fungsi untuk melakukan inisialisasi seperti di bawah, ini berfungsi untuk menjalankan semua service firebase di dalam project

```dart
void main() async {
  ///code sebelumnya

  await Firebase.initializeApp(); //<== Inisialisasi Firebase
}
```

- #### Notification Service

  Untuk dapat menampilkan notifikasi ke dalam device, pertama kita harus mendapatkan FCM token dari Firebase, tapi sebelum itu terjadi kita harus periksa apakah permission dari user sudah berstatus "granted" atau belum.

  kita bisa lihat itu di dalam file **main.dart**

  di dalam file itu terdapat fungsi yang bernama **notifInit()**

  - Di baris awal kita menggunakan fungsi yang terera di bawah untuk membuat instance dari **FirebaseMessaging**

  ```dart
  FirebaseMessaging messaging = FirebaseMessaging.instance;
  ```

  - Lalu selanjutnya ada variable untuk membuat settings terhadap notifikasi yang akan di dapatkan nantinya.

  ```dart
  NotificationSettings settings = await messaging.requestPermission(
      alert: true,
      announcement: false,
      badge: true,
      carPlay: false,
      criticalAlert: false,
      provisional: false,
      sound: true,
    );
  ```

  setelah membuat settingan untuk notifikasi, selanjutnya saatnya untuk mendapatkan FCM token dari Firebase

  ```dart
  await notificationService.getFcmToken;
  ```

  lalu kita cek apakah permission sudah authorized, jika sudah maka simpan FCM Token ke dalam Local Storage / cache

  ```dart
  if(settings.authorizationStatus == AuthorizationStatus.authorized) {
    LocalStorageService.save("fcmToken", notificationService.token);
  }
  ```

  Full Code:

```dart
void notifInit() async {
  FirebaseMessaging messaging = FirebaseMessaging.instance;

  NotificationSettings settings = await messaging.requestPermission(
    alert: true,
    announcement: false,
    badge: true,
    carPlay: false,
    criticalAlert: false,
    provisional: false,
    sound: true,
  );

  await notificationService.getFcmToken();

  print(notificationService.token);

  LocalStorageService.save("fcmToken", notificationService.token);

  await FirebaseMessaging.instance.setForegroundNotificationPresentationOptions(
      alert: true, badge: true, sound: true);

  if (settings.authorizationStatus == AuthorizationStatus.authorized) {
    LocalStorageService.save("fcmToken", notificationService.token);
  } else if (settings.authorizationStatus == AuthorizationStatus.provisional) {
    print('User granted provisional permission');
  } else {
    print('User declined or has not accepted permission');
  }
}
```

- #### Location Service

  Location Service ini di fungsikan untuk mendapatkan data Lokasi terkini dari **Google Map API**, dapat di lihat dalam file **splashscreen.dart** ada sebuah fungsi yang di tujukan untuk membuat insialisasi terhadap Location Service.

  ```dart
  void initState() {
    locationService();

    ///dan berikutnya
  }
  ```

  di dalam fungsi ini memiliki komponen seperti

  ```dart
  _serviceEnabled = await location.serviceEnabled();
  ```

  yang di gunakan untuk memeriksa apakah location service dari device sudah nyala atau tidak.

  lalu selanjutnya ada

  ```dart
  if(!_serviceEnabled){
    _serviceEnabled = await location.requestService();
    if(!_serviceEnabled) return;
  }
  ```

  ini digunakan untuk mengecek jika service belum di nyalakan maka kita akan meminta request untuk aktivasi location service dari device.

  lalu selanjutnya ada

  ```dart
  _permissionGranted = await location.hasPermission();
  ```

  komponen ini ditujukan untuk mendapatkan permission dari user apakah mereka menyetujui akses ke location service atau tidak.

  jika akses granted maka kita harus mendapatkan location data dengan cara

  ```dart
  _locationData = await location.getLocation();
  ```

  lalu location data tersebut dapat kita gunakan untuk mendapatkan value latitude dan juga longitude yang nantinya akan di simpan ke dalam **Local Storage Service / Local Cache**.

  ```dart
  saveLatLong(_locationData.latitude, _locationData.longitude);
  ```

  lalu jika semua hal diatas berhasil di lakukan maka komponen terakhir adalah

  ```dart
  location.onLocationChanged.listen((event) {
    saveLatLong(event.latitude!, event.longitude!);
  });
  ```

  yang nantinya dimana semua perubahaan lokasi dari user akan di listen oleh fungsi **onLocationChanged** dan akan di masukan ke dalam **Local Storage Service / Local Cache** tadi.

### Komponen Modular

**Modular** adalah sebuah package yang digunakan untuk mengatur semua routing dan Navigasi di dalam App, semua route akan tersimpan di setiap masing masing module screen.

di dalam dokumentasi ini hanya akan di bahas beberapa tipe Navigasi yang menggunakan Modular, Untuk lebih lengkapnya bisa di lihat pada [Flutter Official Modular Documentation](https://modular.flutterando.com.br/docs/flutter_modular/start/).

ini beberapa jenis Navigasi yang menggunakan Modular

```dart
Modular.to.push(route);
///dengan fungsi ini kita dapat melakukan navigasi dengan menggunakan sistem Route
///yang di sediakan oleh flutter seperti MaterialPageRoute ataupun CupertinoPageRoute
///tanpa harus menggunakan Named Route.

Modular.to.pushNamed('/nama-route-kamu/');
///dengan fungsi ini kita dapat melakukan navigasi dengan menggunakan sistem named route
///sistem ini sangat memudahkan kita karena tidak perlu memanggil nama class Screen yang akan
///kita panggil untuk melakukan navigasi, cukup dengan menuliskan nama route nya saja yang sudah
///tersimpan di dalam file Module.

Modular.to.pushNamedAndRemoveUntil('/nama-route-yang-akan-di-panggil/', ModalRoute.withName('/nama-route-yang-akan-di-cek'));
///fungsi ini sangat berguna jika kita ingin melakukan navigasi ke halaman berikutnya tapi kita juga ingin
///menghapus seluruh navigasi sebelumnya hingga screen yang akan di cek dan menghasilkan value true.

Modular.to.pop();
///fungsi ini berguna untuk melakukan aksi kembali ke screen sebelumnya.

```

### Autentikasi

Semua hal yang bersangkutan dengan aktivitas autentikasi ada di dalam file **auth_screen.dart**

dan di dalam file ini terdapat beberapa komponen yang sangat penting untuk aktivitas autentikasi seperti Login dan juga Register atau pun user juga dapat melewati alur autentikasi dengan menekan tombol skip.

- #### Skip

  Dengan user memilih tombol skip maka seluruh alur autentikasi ditiadakan dan nantinya semua pemanggilan API yang membutuhkan token autentikasi akan di hentikan dan hanya API yang tidak membutuhkan token saja yang dapat berjalan seperti API News dan semua tutorial penggunaan app di menu home.

  kita harus menyimpan value **isSkip** ke dalam **Local Storage Service / Local Cache** supaya nantinya bisa di gunakan ketika screen sudah berpindah ke home screen.

  jika value **isSkip** sudah tersimpan maka proses setelah itu adalah melakukan navigasi ke halaman home.

  ```dart
  LocalStorageService.save("isSkip", true).then((value) {
    Modular.to.pushNamedAndRemoveUntil('/home/', ModalRoute.withName('/auth/'));
  })
  ```

- #### Login

  Pada file **login_screen.dart** terdapat form **Username** dan **Password**, kedua form ini masing - masing memiliki controller yang nantinya digunakan untuk menyimpan value hasil dari input user.

  Sebelum melakukan proses login, pertama sebuah fungsi validasi akan di panggil untuk mengecek apakah seluruh form yang wajib di isi sudah terisi semua tau belum, Jika sudah terisi semua maka proses login dapat di lanjutkan, jika belum maka akan memunculkan **FlushBar** warning dengan pesan yang akan mengingatkan user bahwa masih ada form yang belum di isi.

  ```dart
  if(phoneController.text.isNotEmpty && passwordController.text.isNotEmpty) {
    //proses login
  } else {
    //munculkan warning
  }
  ```

  Jika hasilnya adalah true maka proses login akan berlanjut dengan memanggil service login

  ```dart
  authServices.userLogin(context, phoneNumber: phoneController.text, password: passwordController.text);
  ```

  di dalam fungsi ini kita melakukan proses login yang membutuhkan phoneController dan passwordController yang nantinya akan di kirim ke dalam field API dan context saat ini untuk memunculkan warning dari **dio**, setelah itu API akan melakukan proses validasi dan pada akhirnya API memberikan result yang akan di gunakan untuk mengecek apakah validasi berhasil atau gagal.

  ```dart
  if(authServices.getMessage()["status"] == "success") {
    //jika status sukses
  } else {
    //jika status gagal
  }
  ```

  Jika status sama dengan sukses maka proses login akan berlanjut, di sini kita akan menyimpan semua latitude dan longitude dari lokasi user ke dalam variable yang nantinya akan di kirimkan ke backend melalui API **updateUserLocation**.

  ```dart
  double latitude = await LocalStorageService.load("latitude");
  double longitude = await LocalStorageService.load("longitude");

  await RestApiService.updateUserLocation(
    latitude: latitude.toString(),
    longitude: longitude.toString(),
  );
  ```

  jika data latitude dan longitude sudah terkirim maka selanjutnya adalah memasukan value isSkip bernilai false ke dalam **Local Storage Service / Local Cache** karena kita telah sukses melakukan proses login. setelah value isSkip berhasil di masukan maka selanjutnya adalah melakukan navigasi ke halaman selanjutnya.

  ```dart
  LocalStorageService.save("isSkip", false).then((value) {
    if(value == true){
      Modular.to.pushReplacementNamed('/auth/authSuccess');
    }
  });
  ```

  Lalu status tidak sama dengan sukses maka sebuah **FlushBar** akan muncul untuk membuat warning kepada user.

  ```dart
  UiUtils.errorMessage(
    authServices.getMessage()["message"],
    context
  );
  ```

  Kemudian jika **phoneController** dan **passwordController** di antara kedua itu ada yang kosong, maka kita munculkan warning juga kepada user.

  ```dart
  UiUtils.errorMessage(
    "Tolong isi semua form!",
    context
  );
  ```

- #### Register

  Tidak jauh berbeda dengan login, pada halaman **register_screen.dart** juga terdapat beberapa form yang digunakan untuk menginput data seperti **Nama, Nomor telepon, Password dan Konfirmasi Password** dan juga mempunyai tambahan step yaitu memasukan **kode OTP**, akan tetapi pada App **Mitra j-lantah** mempunyai step tambahan selain **kode OTP** yaitu **melengkapi profil mitra**.

  Lalu sebelum proses register dapat berjalan, sebelum itu harus di cek terlebih dahulu apakah value dari password & konfirmasi password sudah sama atau belum.

  ```dart
  if(isPasswordMatch()) {
    // jika password sama
  } else {
    // jika tidak sama
  }

  bool isPasswordMatch() {
    return passwordController.text == password2Controller.text
    ? true : false;
  }
  ```

  Jika kedua password sama, maka kita akan melanjutkan proses registernya dengan memanggil API **userRegister**.

  ```dart
  authServices.userRegister(
    name: nameController.text,
    password: passwordController.text,
    phoneNumber: phoneNumberController.text,
    context: context
  );
  ```

  fungsi ini digunakan untuk memproses semua inputan dari user yang nantinya akan dikirim ke backend untuk mendapatkan response json dan status nya.

  setelah semua data telah di dapatkan dari backend, maka selanjutnya kita harus mengecek apakah status dari backend sama dengan sukses atau gagal.

  ```dart
  if (authServices.getMessage()["status"] == "success") {
    /// jika status sukses
  } else {
    /// jika status gagal
  }
  ```

  Jika status sama dengan sukses maka proses register akan berlanjut, di sini kita akan menyimpan nomor telepon user yang nantinya dapat di gunakan di halaman berikutnya.

  ```dart
  LocalStorageService.save("username", phoneNumberController.text);
  ```

  Lalu selanjutnya kita akan melakukan navigasi ke halaman berikutnya yaitu **otp_screen.dart**.

  ```dart
  Modular.to.push(
    MaterialPageRoute(
      builder: (context) => OtpScreen(
        phoneNumber: phoneNumberController.text,
        type: "verif"  //pada class OtpScreen terdapat parameter type yang 
        ///berfungsi untuk membedakan darimana Halaman OTP dipanggil, apakah 
        ///itu dari halaman register atau dari forgot password. 
      ),
    ),
  );
  ```

  Ketika fungsi register terpanggil maka secara otomatis backend akan mengirimkan kode OTP ke nomor yang di daftarkan.

   

### Data poin dan liter minyak

test

```dart
This is for code block

void main() {

}
```

### Menampilkan data history

```dart
This is for code block

void main() {

}
```

### Menampilkan data notifikasi

```dart
This is for code block

void main() {

}
```

### Profile

```dart
This is for code block

void main() {

}
```

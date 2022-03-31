# Dokumentasi Aplikasi j-lantah & mitra j-lantah

## Memulai

- [Inisialisasi flutter](#inisialisasi-flutter)
- [Inisialisasi firebase service](#inisialisasi-firebase-service)
  - [Notification Service](#notification-service)
  - [Location Service](#location-service)
- [Komponen Modular](#komponen-modular)

### Views

- [Splashscreen](#splashscreen)
- [Auth Screen](#auth-screen)
  - [Register Screen](#register-screen)
  - [Login Screen](#login-screen)
- [Home](#home)
- [Order Screen](#order-screen)
- [Redeem Poin](#redeem-poin)

### Models

- [Register Model](#register-model)
- [Mitra Model](#mitra-model)
- [My Poin Model](#my-poin-model)
- [Profile Model](#profile-model)
- [Notification Model](#notification-model)

### Controllers

- [Autentikasi](#autentikasi)
  - [Register](#register)
  - [Login](#login)
- [Data poin dan liter minyak](#data-poin-dan-liter-minyak)
- [Menampilkan data history](#menampilkan-data-history)
- [Menampilkan data notifikasi](#menampilkan-data-notifikasi)
- [Profile](#profile)
- [Order](#order)
  - [Create order](#create-order)
  - [Find mitra](#find-mitra)
  - [Konfirmasi Mitra Sampai Lokasi](#konfirmasi-mitra-sampai-lokasi)
  - [Konfirmasi Liter Minyak](#konfirmasi-liter-minyak)

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

## Views

## Models

- ### Register Model

  Model ini di gunakan untuk melakukan serialisasi terhadap data yang di dapat ketika melakukan Register. data yang di dapat antara lain yaitu **id, nama, password, active, kode**. Dan password yang dimaksud disini adalah password untuk melakukan verifikasi nomor telepon beserta kode juga.

  ```dart
  class RegisterModel {
    String? id;
    String? nama;
    String? password;
    int? active;
    String? kode;

    RegisterModel({this.id, this.nama, this.password, this.active, this.kode});

    RegisterModel.fromJson(Map<String, dynamic> json) {
      id = json['id'];
      nama = json['nama'];
      password = json['password'];
      active = json['active'];
      kode = json['kode'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = new Map<String, dynamic>();
      data['id'] = this.id;
      data['nama'] = this.nama;
      data['password'] = this.password;
      data['active'] = this.active;
      data['kode'] = this.kode;
      return data;
    }

    @override
    String toString() {
      return "id: $id, nama: $nama, active: $active, kode: $kode";
    }
  }
  ```

- ### Mitra Model

  Model ini di gunakan untuk melakukan serialisasi terhadap data yang di dapat ketika User mencari mitra di daerah sekitarnya. data yang di dapat antara lain yaitu **id, latitude, longitude, newOrder, orderId**.

  ```dart
  class MitraModel {
    MitraModel({
      required this.data,
    });
    late final List<Data> data;

    MitraModel.fromJson(Map<String, dynamic> json) {
      data = List.from(json['data']).map((e) => Data.fromJson(e)).toList();
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['data'] = data.map((e) => e.toJson()).toList();
      return _data;
    }
  }

  class Data {
    Data({
      required this.id,
      required this.latitude,
      required this.longitude,
      required this.newOrder,
      required this.orderId,
    });
    late final String id;
    late final String latitude;
    late final String longitude;
    late final String newOrder;
    late final String orderId;

    Data.fromJson(Map<String, dynamic> json) {
      id = json['id'];
      latitude = json['latitude'];
      longitude = json['longitude'];
      newOrder = json['new_order'];
      orderId = json['order_id'];
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['id'] = id;
      _data['latitude'] = latitude;
      _data['longitude'] = longitude;
      _data['new_order'] = newOrder;
      _data['order_id'] = orderId;
      return _data;
    }
  }
  ```

- ### My Poin Model

  Model ini di gunakan untuk melakukan serialisasi terhadap data yang di dapat ketika User akan melakukan penarikan poin. data yang di dapat antara lain yaitu **userId, poin**

  ```dart
  class MyPoinModel {
    MyPoinModel({
      required this.data,
    });
    late final List<Data> data;

    MyPoinModel.fromJson(Map<String, dynamic> json) {
      data = List.from(json['data']).map((e) => Data.fromJson(e)).toList();
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['data'] = data.map((e) => e.toJson()).toList();
      return _data;
    }
  }

  class Data {
    Data({
      required this.userId,
      required this.poin,
    });
    late final String userId;
    late final int poin;

    Data.fromJson(Map<String, dynamic> json) {
      userId = json['user_id'];
      poin = json['poin'];
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['user_id'] = userId;
      _data['poin'] = poin;
      return _data;
    }
  }
  ```

- ### Profile Model

  Model ini di gunakan untuk melakukan serialisasi terhadap data yang di dapat untuk menampilkan semua detail dari user yang sedang login.

  ```dart
  class ProfileModel {
    String? id;
    String? nama;
    String? username;
    String? email;
    String? phone;
    String? jenisKelamin;
    String? usia;
    String? nik;
    String? provider;
    String? companyId;
    String? tempatLahir;
    String? tanggalLahir;
    String? role;
    int? levelUser;
    late String avatar;
    List<ProfileAlamat>? alamat;
    List<ProfileBank>? bank;
    List<Biodata>? biodata;
    List? skills;
    List<Dokumen>? dokumen;
    int? agreement;
    List<Perusahaan>? perusahaan;

    ProfileModel(
        {this.id,
        this.nama,
        this.username,
        this.email,
        this.phone,
        this.jenisKelamin,
        this.usia,
        this.provider,
        this.companyId,
        this.tempatLahir,
        this.tanggalLahir,
        this.role,
        this.levelUser,
        this.avatar = "",
        this.alamat,
        this.bank,
        this.nik,
        this.biodata,
        this.skills,
        this.dokumen,
        this.agreement,
        this.perusahaan});

    ProfileModel.fromJson(Map<String, dynamic> json) {
      id = json['id'];
      nama = json['nama'];
      username = json['username'];
      email = json['email'];
      nik = json['nik'];
      phone = json['phone'];
      jenisKelamin = json['jenis_kelamin'];
      usia = json['usia'];
      provider = json['provider'];
      companyId = json['company_id'];
      tempatLahir = json['tempat_lahir'];
      tanggalLahir = json['tanggal_lahir'];
      role = json['role'];
      levelUser = json['level_user'];
      avatar = json['avatar'] ?? "";
      if (json['perusahaan'] != null) {
        perusahaan = <Perusahaan>[];
        json['perusahaan'].forEach((v) {
          perusahaan?.add(Perusahaan.fromJson(v));
        });
      }
      if (json['alamat'] != null) {
        alamat = <ProfileAlamat>[];
        json['alamat'].forEach((v) {
          alamat?.add(ProfileAlamat.fromJson(v));
        });
      }
      if (json['bank'] != null) {
        bank = <ProfileBank>[];
        json['bank'].forEach((v) {
          bank?.add(ProfileBank.fromJson(v));
        });
      }
      if (json['biodata'] != null) {
        biodata = [];
        json['biodata'].forEach((v) {
          biodata!.add(Biodata.fromJson(v));
        });
      }
      if (json['skills'] != null) {
        skills = json['skills'];
      }
      if (json['dokumen'] != null) {
        dokumen = [];
        json['dokumen'].forEach((v) {
          dokumen!.add(Dokumen.fromJson(v));
        });
      }
      agreement = json['agreement'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = <String, dynamic>{};
      data['id'] = id;
      data['nama'] = nama;
      data['username'] = username;
      data['nik'] = nik;
      data['email'] = email;
      data['phone'] = phone;
      data['jenis_kelamin'] = jenisKelamin;
      data['usia'] = usia;
      data['provider'] = provider;
      data['company_id'] = companyId;
      data['tempat_lahir'] = tempatLahir;
      data['tanggal_lahir'] = tanggalLahir;
      data['role'] = role;
      data['level_user'] = levelUser;
      data['avatar'] = avatar;
      if (alamat != null) {
        data['alamat'] = alamat!.map((v) => v.toJson()).toList();
      }
      if (bank != null) {
        data['bank'] = bank!.map((v) => v.toJson()).toList();
      }
      if (biodata != null) {
        data['biodata'] = biodata!.map((v) => v.toJson()).toList();
      }
      if (skills != null) {
        data['skills'] = skills!.map((v) => v.toJson()).toList();
      }
      if (dokumen != null) {
        data['dokumen'] = dokumen!.map((v) => v.toJson()).toList();
      }
      data['agreement'] = agreement;
      return data;
    }
  }

  class ProfileAlamat {
    bool? isDefault;
    String? sId;
    String? alamat;
    String? namaAlamat;
    String? locationDetail;
    String? propinsi;
    String? propinsiKode;
    String? kelurahan;
    String? kelurahanKode;
    String? kecamatan;
    String? kecamatanKode;
    String? kota;
    String? kotaKode;
    String? latitude;
    String? longitude;

    ProfileAlamat(
        {this.isDefault,
        this.sId,
        this.alamat,
        this.namaAlamat,
        this.locationDetail,
        this.kelurahan,
        this.kelurahanKode,
        this.kecamatan,
        this.kecamatanKode,
        this.propinsi,
        this.propinsiKode,
        this.kota,
        this.kotaKode,
        this.latitude,
        this.longitude});

    ProfileAlamat.fromJson(Map<String, dynamic> json) {
      isDefault = json['is_default'];
      sId = json['_id'];
      alamat = json['alamat'];
      namaAlamat = json['nama_alamat'];
      locationDetail = json['location_detail'];
      kelurahan = json['kelurahan'];
      kelurahanKode = json['kelurahan_kode'];
      kecamatan = json['kecamatan'];
      kecamatanKode = json['kecamatan_kode'];
      kota = json['kota'];
      kotaKode = json['kota_kode'];
      propinsi = json['provinsi'];
      propinsiKode = json['provinsi_kode'];
      latitude = json['latitude'];
      longitude = json['longitude'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = <String, dynamic>{};
      data['is_default'] = isDefault;
      data['_id'] = sId;
      data['alamat'] = alamat;
      data['nama_alamat'] = namaAlamat;
      data['location_detail'] = locationDetail;
      data['kelurahan'] = kelurahan;
      data['kelurahan_kode'] = kelurahanKode;
      data['kecamatan'] = kecamatan;
      data['kecamatan_kode'] = kecamatanKode;
      data['kota'] = kota;
      data['kota_kode'] = kotaKode;
      data['propinsi'] = propinsi;
      data['propinsi_kode'] = propinsiKode;
      data['latitude'] = latitude;
      data['longitude'] = longitude;
      return data;
    }
  }

  class ProfileBank {
    String? sId;
    String? bankId;
    String? bankName;
    String? bankAccount;
    String? bankAccountName;
    String? bankCode;
    String? tautan;
    String? fileTabungan;

    ProfileBank(
        {this.sId,
        this.bankId,
        this.bankName,
        this.bankAccount,
        this.bankAccountName,
        this.tautan,
        this.bankCode,
        this.fileTabungan});

    ProfileBank.fromJson(Map<String, dynamic> json) {
      sId = json['_id'];
      bankId = json['bank_id'];
      bankName = json['bank_name'];
      bankAccount = json['bank_account'];
      bankAccountName = json['bank_account_name'];
      bankCode = json['bank_code'];
      tautan = json['tautan'];
      fileTabungan = json['file_tabungan'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = <String, dynamic>{};
      data['_id'] = sId;
      data['bank_id'] = bankId;
      data['bank_name'] = bankName;
      data['bank_account'] = bankAccount;
      data['bank_account_name'] = bankAccountName;
      data['bank_code'] = bankCode;
      data['tautan'] = tautan;
      data['file_tabungan'] = fileTabungan;
      return data;
    }
  }

  class Biodata {
    String? sId;
    String? fileEktp;
    String? fileSelfieEktp;
    String? tautan;

    Biodata({this.sId, this.fileEktp, this.fileSelfieEktp, this.tautan});

    Biodata.fromJson(Map<String, dynamic> json) {
      sId = json['_id'];
      fileEktp = json['file_ektp'];
      fileSelfieEktp = json['file_selfie_ektp'];
      tautan = json['tautan'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = <String, dynamic>{};
      data['_id'] = sId;
      data['file_ektp'] = fileEktp;
      data['file_selfie_ektp'] = fileSelfieEktp;
      data['tautan'] = tautan;
      return data;
    }
  }

  class Dokumen {
    String? sId;
    String? tautan;

    Dokumen({this.sId, this.tautan});

    Dokumen.fromJson(Map<String, dynamic> json) {
      sId = json['_id'];
      tautan = json['tautan'];
    }

    Map<String, dynamic> toJson() {
      final Map<String, dynamic> data = <String, dynamic>{};
      data['_id'] = sId;
      data['tautan'] = tautan;
      return data;
    }
  }
  ```

- ### Notification Model

  Model ini di gunakan untuk melakukan serialisasi terhadap data yang di dapat ketika User membuka screen Notifikasi. data yang di dapat antara lain yaitu **id, isRead, userId, role, type, orderId, title, message, createdAt**

  ```dart
  class NotificationModel {
    NotificationModel({
      required this.data,
    });
    late final List<NotificationData> data;

    NotificationModel.fromJson(Map<String, dynamic> json) {
      data = List.from(json['data']).map((e) => NotificationData.fromJson(e)).toList();
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['data'] = data.map((e) => e.toJson()).toList();
      return _data;
    }
  }

  class NotificationData {
    NotificationData({
      required this.id,
      required this.isRead,
      required this.userId,
      required this.role,
      required this.type,
      required this.orderId,
      required this.title,
      required this.message,
      required this.createdAt,
    });
    late final String id;
    late final bool isRead;
    late final String userId;
    late final String role;
    late final String type;
    late final String orderId;
    late final String title;
    late final String message;
    late final String createdAt;

    NotificationData.fromJson(Map<String, dynamic> json) {
      id = json['id'];
      isRead = json['is_read'];
      userId = json['user_id'];
      role = json['role'];
      type = json['type'];
      orderId = json.containsKey("order_id") ? json['order_id'] : "";
      title = json['title'];
      message = json['message'];
      createdAt = json['created_at'];
    }

    Map<String, dynamic> toJson() {
      final _data = <String, dynamic>{};
      _data['id'] = id;
      _data['is_read'] = isRead;
      _data['user_id'] = userId;
      _data['role'] = role;
      _data['type'] = type;
      _data['order_id'] = orderId;
      _data['title'] = title;
      _data['message'] = message;
      _data['created_at'] = createdAt;
      return _data;
    }
  }

  ```

## Controllers

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

Pada saat user melakukan login ataupun register, aplikasi secara bersamaan juga mendapatkan data profile user yang saat ini sedang login dengan menggunakan API **getProfile()** dan juga API **getMyPoin()**.

Di dalam result dari API tersebut kita mendapatkan beberapa data termasuk data Poin dan total liter minyak user.

```dart
RestApiService.getMyPoin().then((value) {
  poin = value.data["data"][0]["poin"].toString();
  setState(() {});
});

RestApiService.getProfile().then((value) {
  print(value["data"]);
  if (value["statusCode"] == 200) {
    print(value["data"]["total_liter"]);
    liter = value["data"]["total_liter"].toString();
    setState(() {});
  } else {
    UiUtils.errorMessage(value["message"], context);
  }
});
```

Jika semua data yang dibutuhkan telah di dapatkan maka kita akan menampilkan data tersebut ke dalam screen, jika data kosong atau tidak di temukan maka akan dimasukan value 0.

### Menampilkan data history

Pada file **order_history_screen.dart** terdapat sebuah fungsi yang digunakan untuk mengambil data history dari backend, data history dari backend ini memiliki dua tipe, pertama ada **On Going** dan satu lagi ada **Completed**.

fungsi untuk pemanggilan data history ini sudah di design agar dapat memasukan parameter tipe status ordernya.

```dart
Future<void> getAllOrderData(String status, int page) async {
  if (!mounted) return;
  await RestApiService.getAllOrderByDate(status, page: page.toString()).then((value) {
    if (value.statusCode == 200) {
      if (value.data["status"] == "success") {
        if (status == "ongoing") {
          ongoingData = value.data["order"];
          setState(() {});
        } else {
          completedData = value.data["order"];
          setState(() {});
        }
      } else {
        UiUtils.errorMessage(value.data["message"], context);
      }
    } else {
      UiUtils.errorMessage(value.data["message"], context);
    }
  });
}
```

Jadi, jika kita ingin mendapatkan data dengan status **On going** maka kita bisa memanggil fungsi di atas seperti ini.

```dart
await getAllOrderData("ongoing", currentOngoingPage);
```

Begitu pula dengan status **Completed**.

```dart
await getAllOrderData("completed", currentCompletedPage);
```

Dan masing masing status memiliki pagination nya sendiri supaya tidak bentrok antara pagination yang satu dengan yang lainnya.

Semua data di inisialisasikan ketika user masuk ke menu History dan juga ketika user melakukan refresh terhadap masing masing tab.

### Menampilkan data notifikasi

Untuk menampilkan data notifikasi, dapat di lihat pada file **notification_screen.dart**. Di dalam file ini kita mempunyai fungsi untuk memanggil API dari backend untuk mendapatkan data notifikasi.

```dart
Future<void> getNotification() async {
  isLoading = true;
  setState(() {});
  currentPage = 1;
  setState(() {});
  await RestApiService.getNotification(page: currentPage.toString())
      .then((value) {
    if (value.statusCode == 200) {
      notificationModel = NotificationModel.fromJson(value.data);
      isLoading = false;
      if (mounted) setState(() {});

      print(notificationModel.data.length);
    } else {
      isLoading = false;
      if (mounted) setState(() {});
      UiUtils.errorMessage(value.data["message"], context);
    }
  });
}
```

Dan semua data dari backedn akan kita simpan ke dalam **Model notifikasi** yang nantinya akan di gunakan untuk variable penampil data di dalam UI.

Sama seperti screen order history, screen notifikasi juga memiliki pagination yang sudah di atur di dalam fungsi **onLoading()**.

```dart
void onLoading() async {
  currentPage += 1; //Inti utama dalam pagination ada di bagian ini, cukup dengan menambahkan
  ///variable currentPage dengan value 1 maka nanti api akan memanggil data Notifikasi halaman berikutnya.

  print("current page: " + currentPage.toString());
  await RestApiService.getNotification(page: currentPage.toString())
      .then((value) {
    if (value.statusCode == 200) {
      print("data: " + value.data.toString());
      List<NotificationData> newNotificationData =
          List.from(value.data["data"])
              .map((e) => NotificationData.fromJson(e))
              .toList(); /// Lalu disini kita masukan data yang telah di dapat dari API ke dalam List of Model.
      print("new notification data: $newNotificationData");
      notificationModel.data.addAll(newNotificationData); ///jika fungsinya adalah onLoading maka kita lakukan penambahan data
      ///dengan fungsi addAll(data) yang disediakan oleh List.
      isLoading = false;
      if (mounted) setState(() {});

      print(notificationModel.data.length);
      _refreshController.loadComplete(); ///setelah semua proses selesai maka terakhir adalah\
      ///kita harus mengatur _refreshController agar selesai melakukan loading.
    } else {
      isLoading = false;
      if (mounted) setState(() {});
      UiUtils.errorMessage(value.data["message"], context); ///Jika hasilnya false maka kita
      ///harus menampilkan error message ke pada user
      _refreshController.loadComplete(); ///setelah semua proses selesai maka terakhir adalah\
      ///kita harus mengatur _refreshController agar selesai melakukan loading.
    }
  });
}
```

### Profile

### Order

Pada flow order, terdapat beberapa file yang akan digunakan, untuk yang pertama ada file **pickup_screen.dart**, file ini digunakan untuk membuat order penjemputan yang bisa di lihat dalam bagian [Create order](#create-order).

- #### Create order

  Pada file **pickup_screen.dart** terdapat beberapa function.

  Fungsi **getAnnouncement**, fungsi ini digunakan untuk pengumuman dari backend j-lantah.

  ```dart
  Future<void> getAnnouncement() async {
    await RestApiService.getLegalText("announcement").then((val) {
      if(val.statusCode == 200) {
        announcementText = val.data["data"]["setting_value"];
        setState(() {

        });
      } else {
        UiUtils.errorMessage(val.data["message"], context);
      }
    });
  }
  ```

  Fungsi **getPricingData**, fungsi ini digunakan untuk mengambil data harga per liter yang telah di simpan di dalam cache, data harga akan terupdate dengan sendirinya ketika terdapat perubahan dari firebase relatime database.

  ```dart
  void getPricingData() async {
    pricingData = await LocalStorageService.load("priceData");
    setState(() {});

    print("pricing data: " + pricingData.toString());
  }
  ```

  Fungsi **loadLatLong**, fungsi ini digunakan untuk mengambil data latitude dan longitude yang telah tersimpan di awal saat kita login.

  Di dalam fungsi ini juga kita akan melakukan setup titik lokasi awal pada API google map serta melakukan pengambilan detail lokasi dengan menggunakan **placemark** dan fungsi **getAddressDetails**

  ```dart
  Future loadLatLong() async {
    isLoading = true;
    setState(() {});
    await LocalStorageService.load("latitude").then((value) async {
      await LocalStorageService.load("longitude").then((value2) async {
        latLong = LatLng(value, value2);
        navigationService.setInitialLocation(latLong!);
        placemark = await placemarkFromCoordinates(value, value2);

        getAddressDetails("$value,$value2");

        print("placemark: " + placemark[0].toJson().toString());

        isLoading = false;
        setState(() {});
      });
    });
  }
  ```

  Fungsi **getAddressDetails**, fungsi ini di gunakan untuk mendapatkan detail dari alamat user, detail alamat di dapatkan dengan memasukan latitude dan longitude ke dalam API google geocoding untuk mendapatkan **place_id** nya, setelah itu kita dapatkan detail alamatnya menggunakan fungsi **getPlaceDetailFromId** dengan mengirimkan **place_id** yang kita dapatkan tadi.

  ```dart
  void getAddressDetails(String latlong) async {
    await RestApiService.getAddressDetails(latlong).then((value) {
      print(value.body);
      var extracted = json.decode(value.body);
      if (value.statusCode == 200) {
        String session = Uuid().v4();

        PlaceApiProvider(session)
            .getPlaceDetailFromId(extracted["results"][0]["place_id"])
            .then((value) {
          addressName = value.businessName;

          setState(() {});

          print("address name: " + addressName);
        });

        fullAddress = extracted["results"][0]["formatted_address"];
        print("fullAddress: " + fullAddress);
        List address = fullAddress.split(", ");

        print(addressName);
        // addressName = extracted["results"][0][""];

        setState(() {});
      }
    });
  }
  ```

  Fungsi **getPricing**, digunakan untuk mengkalkulasikan jarak harga minimum atau maximum dengan total liter yang di inputkan oleh user.

  ```dart
  Future<double> getPricing(double liter) async {
    double price = 0;

    if (liter >= pricingData[0]["min_range"] &&
        liter <= pricingData[0]["max_range"]) {
      price = pricingData[0]["price"].toDouble() * liter;
    } else if (liter >= pricingData[1]["min_range"] &&
        liter <= pricingData[1]["max_range"]) {
      price = pricingData[1]["price"].toDouble() * liter;
    }

    return price;
  }
  ```

  Fungsi **showPinsOnMap**, fungsi ini digunakan untuk menampilkan pin lokasi user di dalam map.

  ```dart
  void showPinsOnMap() {
    setState(() {
      _markers.add(
        Marker(
          markerId: MarkerId('sourcePin'),
          position: navigationService.currentLocation,
          icon: navigationService.sourceIcon,
        ),
      );
    });
  }
  ```

  Semua fungsi di atas di gunakan untuk melakukan proses **Create Order**.

  Flow order:

  - User melakukan pemilihan lokasi pada tombol **Atur Lokasi Penjemputan**.
  - User memasukan detail alamat pada form yang telah di sediakan.
  - User melakukan input total liter minyak yang akan di pickup.
  - Terakhir user menekan tombol **Cari Mitra**, di dalam tombol tersebut terdapat sebuah fungsi yang digunakan untuk melakukan membuat order baru dan mengalihkan ke halaman proses order mitra.

    ```dart
    isLoading = true;
    setState(() {});
    if (literTotal < 1.0) {
      isLoading = false;
      setState(() {});
      UiUtils.errorMessage(
          "Jumlah liter minimal 1,0", context);
    } else if (isCanOrder == false) {
      isLoading = false;
      setState(() {});
      UiUtils.errorMessage(
          "Tidak dapat membuat order, Ada order yang sedang berjalan",
          context);
    } else {
      RestApiService.createNewOrder(
        literPrice.toInt(),
        lat: latLong?.latitude.toString(),
        long: latLong?.longitude.toString(),
        quantity: literTotal,
        orderLocation: fullAddress,
        locationDetails: locationDetailsController.text,
      ).then((value) {
        isLoading = false;
        setState(() {});
        if (value["statusCode"] == 200) {
          print(value["data"]);

          LocalStorageService.save(
            "order_id",
            value["data"]["data"]["id"],
          );

          LocalStorageService.save(
            "order_poin",
            literPrice,
          );

          Modular.to.push(
            MaterialPageRoute(
              builder: (context) => SuccessPageTemplate(
                topText: "Order berhasil dibuat",
                topTextSize: 36.sp,
                pageToTransition: ProsesPickupScreen(
                  lat: latLong?.latitude,
                  long: latLong?.longitude,
                  liter: literTotal.toString(),
                  orderId: value["data"]["data"]["id"],
                  orderNo: value["data"]["data"]
                      ["order_no"],
                ),
                // bottomText:
                //     "",
              ),
            ),
          );
        } else {
          print(value["message"]);
          for (int i = 0;
              i < value["message"].data["data"].length;
              i++) {
            UiUtils.errorMessage(
                "${value['message'].data["message"]}: ${value["message"].data["data"][i]["msg"]}",
                context);
          }
        }
      });
    }
    ```

    Pada file **proses_pickup_screen.dart** terdapat beberapa langkah order yang nantinya akan terupdate sesuai state yang di dapatkan dari notifikasi ataupun dari backend.

  - #### Find mitra

    **findMitra** di sini berfungsi untuk memanggil proses pencarian mitra.

    bagian pertama di dalam fungsi **findMitra** digunakan untuk mengatur state dari order yang sedang berjalan. Pada code di bawah bisa kita lihat state pertama yang aktif atau true adalah searching yang berarti kita sedang dalam state mencari mitra.

    ```dart
    state.addAll({
      "searching": true,
      "found": false,
      "pickedup": false,
      "sending": false,
      "done": false,
    });
    ```

    Lalu selanjutnya ada pemanggilan API untuk mencari mitra, API ini memiliki beberapa parameter seperti **Order id, latitude, longitude, dan juga radius**

    ```dart
    await RestApiService.findMitra(
      orderId: value,
      lat: value2.toString(),
      long: value3.toString(),
      radius: radius,
    )
    ```

    Fungsi di atas akan mengembalikan nilai response dari backend.

    dari response yang kita dapat akan ada nilai statusCode, jika status code sama dengan 200 maka kita akan melanjutkan proses pencarian mitra.

    Jika proses pencarian mitra berlanjut, maka selanjutnya kita akan periksa apakah radius lebih dari sama dengan 10000.

    Jika lebih dari sama dengan 10000 maka aplikasi akan memunculkan popup yang akan memberitahu bahwa tidak ada Mitra terdeteksi di sekitar wilayah User.

    ```dart
    if (radius >= 10000) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (thisContext) => Dialog(
          backgroundColor: Colors.transparent,
          child: AppDialogs.textPromptDialog(
            "Cari Lagi",
            "Batal",
            "Maaf, belum ada mitra yang ambil Order Anda",
            context,
            onYes: () {
              radius = 5000;
              Modular.to.pop(thisContext);
              findMitra(widget.orderId, lat, long);
            },
            onDenied: () {
              RestApiService.cancelOrder(
                orderId: orderId,
                reason: "Tidak mendapatkan Mitra",
              ).then((value) async {
                if (value.statusCode == 200) {
                  await LocalStorageService.save("can-order", true);
                  Modular.to.popUntil(ModalRoute.withName("/home/"));
                } else {
                  UiUtils.errorMessage(value.data["message"], context);
                }
              });
            },
          ),
        ),
      );
    }
    ```

    **Catatan: Radius akan bertambah sebanyak 2000 jika dalam 5 detik tidak di temukan mitra sampai akhirnya menyentuh angka 10000.**

    ```dart
    if (value4["data"]["status"] == "failed") {
      myTimer = Timer(const Duration(seconds: 5), () {
        timerStacked += 5;
        if (timerStacked >= 30) {
          radius += 2000;
        }

        print("timerStacked: " + timerStacked.toString());

        findMitra(value, value2, value3);
      });

      isShowDriverContainer = false;
      if (mounted) setState(() {});
    }
    ```

    Jika di temukan Mitra di sekitar lokasi user maka semua data mitra akan di masukan ke dalam Model Mitra dan akan langsung mengambil data order dengan memanggil fungsi **getOrderData** untuk melakukan pengecekan status lebih lanjut.

    ```dart
    print(value4["data"]);

    mitraModel = MitraModel.fromJson(value4["data"]);
    setState(() {});

    getOrderResetTimer = Timer(Duration(seconds: 5), () {
      getOrderData(orderId);
    });
    ```

    **Catatan: Semua state kecuali find_mitra akan menyimpan sebuah value can_order bernilai true supaya nanti bisa membuat order lagi**

  - #### Konfirmasi Mitra Sampai Lokasi

    Pada saat User mendapatkan Mitra maka nanti status order akan berubah menjadi **waiting_mitra**, pada saat status tersebut aktif, sebuah tombol akan muncul untuk mengkonfirmasi apakah Mitra sudah sampai di tempat User atau belum, lalu ada juga operasi logika untuk mengecek apakah myTimer tidak sama dengan null dan mitraSearchTimer tidak sama dengan null, jika iya maka kedua timer tersebut akan di hentikan.

    Dan juga semua data mitra akan di simpan kedalam variable **driverData** supaya nanti dapat di tampilkan ke dalam UI dengan menggunakan fungsi **setMitraLocation()**.

    Selain itu kita juga akan menampilkan tombol batal supaya User dapat membatalkan order dengan alasan yang telah tersedia.

    ```dart
    if (myTimer != null) {
      myTimer!.cancel();
    }
    if (mitraSearchTimer != null) {
      mitraSearchTimer!.cancel();
    }
    LocalStorageService.save("can-order", true);
    mitraUsername = value.data["data_mitra"][0]["phone_mitra"];
    state.addAll({"found": true});
    isShowButton = true;
    driverData = value.data["data_mitra"][0];

    isShowCancelButton = true;

    print("driver data" + driverData.toString());

    setMitraLocation();
    ```

    Setelah semua itu selesai di eksekusi maka selanjutnya user akan menekan tombol **Sudah Sampai**, di dalam tombol tersebut terdapat sebuah fungsi yang akan mengeksekusi API **konfirmasi sampai**, proses pertama aplikasi akan memunculkan popup untuk menanyakan apakah Mitra sudah sesuai dengan data yang ada di UI.

    Jika Mitra sesuai dengan data yang ada di UI maka aplikasi akan mengeksekusi API **konfirmasi sampai**, jika tidak maka popup lainnya muncul untuk menanyakan apakah user ingin mencari Mitra baru atau membatalkan order yang sedang berjalan.

    ```dart
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (thisContext) {
        isLoading = false;
        return Dialog(
          backgroundColor: Colors.transparent,
          child: AppDialogs.textPromptDialog(
            "Terima",
            "Tolak",
            "Apakah Mitra yang datang sesuai dengan data Mitra?",
            context,
            onYes: () {
              RestApiService.confirmArrived(
                orderId: orderId,
              ).then(
                (value) {
                  if (value.statusCode == 200) {
                    if (value.data["status"] == "success") {
                      Navigator.pop(thisContext);
                      getOrderData(orderId);

                      isLoading = false;
                      setState(() {});
                    } else {
                      UiUtils.errorMessage(
                          value.data["message"], context);
                    }
                  } else {
                    UiUtils.errorMessage(value.data["message"], context);
                  }
                },
              );
            },
            onDenied: () {
              Navigator.pop(thisContext);
              String selectedValue = "0";
              String message = "";
              showDialog(
                context: context,
                barrierDismissible: false,
                builder: (thisSecondContext) {
                  String reason;
                  return StatefulBuilder(
                    builder: (
                      context,
                      thisSetState,
                    ) =>
                        AppDialogs.rejectMitraDialog(
                      context,
                      message: (value) {
                        print(value);
                        message = value;
                        reasonText = value;
                      },
                      onSelected: (value) {
                        selectedValue = value;
                        thisSetState(
                          () {},
                        );
                      },
                      onReject: () {
                        Navigator.pop(thisSecondContext);
                        isLoading = false;
                        setState(() {});
                      },
                      onAccept: () {
                        print("test");
                        RestApiService.rejectMitra(
                          orderId: orderId,
                          reason: message,
                        ).then((value) {
                          print(value.data);
                          if (value.statusCode == 200) {
                            if (value.data["status"] == "success") {
                              Navigator.pop(thisSecondContext);
                              isLoading = false;
                              setState(() {});
                              showDialog(
                                context: context,
                                barrierDismissible: false,
                                builder: (thisContext) => Dialog(
                                  backgroundColor: Colors.transparent,
                                  child: AppDialogs.textPromptDialog(
                                    "Cari Lagi",
                                    "Batal",
                                    "Apakah Anda ingin mencari mitra lain?",
                                    context,
                                    onYes: () {
                                      radius = 5000;
                                      Modular.to.pop(thisContext);
                                      findMitra(
                                          widget.orderId, lat, long);
                                    },
                                    onDenied: () {
                                      RestApiService.cancelOrder(
                                        orderId: orderId,
                                        reason: "Dibatalkan oleh user",
                                      ).then((value) {
                                        if (value.statusCode == 200) {
                                          getOrderData(orderId);
                                          // Modular.to.popUntil(ModalRoute.withName("/home/"));
                                        } else {
                                          UiUtils.errorMessage(
                                              value.data["message"],
                                              context);
                                        }
                                      });
                                    },
                                  ),
                                ),
                              );
                            }
                          }
                        });
                      },
                      acceptButtonText: "Tolak mitra",
                      selectedValue: selectedValue,
                      rejectionList: reasonData,
                    ),
                  );
                },
              );
            },
          ),
        );
      },
    );
    ```

  - #### Konfirmasi Liter Minyak

    Ketika status order sudah berada di **waiting_confirmation** maka terlebih dahulu app Mitra akan melakukan proses konfirmasi apakah minyak sudah sesuai aplikasi, setelah proses konfirmasi dari Mitra selesai maka app User akan memunculkan tombol **konfirmasi** yang ditujukan untuk memunculkan popup konfirmasi apakah setuju dengan Mitra atau tidak.

    ```dart
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (myContext) => AppDialogs.confirmOrder(myContext,
          richText: RichText(
            textAlign: TextAlign.center,
            text: TextSpan(
              children: [
                TextSpan(
                    text: "Jumlah Minyak Anda ",
                    style: TextStyle(color: Colors.black)),
                TextSpan(
                  text: "$liter Liter ",
                  style: TextStyle(
                    color: AppColors.orangeAccent,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                TextSpan(
                    text: "Poin Anda ",
                    style: TextStyle(color: Colors.black)),
                TextSpan(
                  text: "$poin",
                  style: TextStyle(
                    color: AppColors.orangeAccent,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
          acceptButtonText: "Konfirmasi", onAccept: () {
        isLoading = true;
        isConfirmed = true;
        setState(() {});
        RestApiService.quantityConfirm(orderId: orderId).then((thisValue) {
          print(thisValue.data);
          if (thisValue.statusCode == 200) {
            getOrderData(orderId);
            isShowButton = false;
            isShowCancelButton = false;
            isShowConfirmButton = false;
            isLoading = false;
            setState(() {});
            Navigator.pop(myContext);
          }
        });
      }),
    );
    ```

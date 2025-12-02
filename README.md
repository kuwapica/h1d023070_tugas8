# PERTEMUAN 11 - TUGAS 9
Nama : Annida Aiska Humairoh <br>
NIM : H1D023070 <br>
SHIFT : D (baru), I (awal)<br><br>
link drive untuk screenshot aplikasi: https://drive.google.com/drive/folders/1DinsnfgciFmXqanzT8IYiuD23UiR8HLD?usp=sharing 
<br><br>

## Penjelasan kode
### Proses Registrasi
<img width="378" height="504" alt="Screenshot 2025-12-02 212428" src="https://github.com/user-attachments/assets/8c8d6ff3-85ae-4ff8-84cc-552ac828658b" />
<img width="378" height="504" alt="Screenshot 2025-12-02 212359" src="https://github.com/user-attachments/assets/73f0ec1f-ca36-49a9-b554-ed5a72357cdc" />
<br><br>

- User mengisi Form Registrasi di Flutter dengan menginputkan nama, email, password, serta konfirmasi password. Lalu klik tombol Registrasi akan memicu fungsi
  ```
  RegistrasiBloc.registrasi(
  nama: _namaTextboxController.text,
  email: _emailTextboxController.text,
  password: _passwordTextboxController.text)
  ```
  kemudian cek validasi dari inputan tadi.
  <br>
- Flutter mengirim Request HTTP POST ke API Registrasi CI4<br>
  LoginBloc menggunakan helper:
  ```
  String apiUrl = ApiUrl.registrasi;
  var body = {
    "nama": nama,
    "email": email,
    "password": password
  };
  var response = await Api().post(apiUrl, body);
  ```
  Endpoint REST API: `/registrasi`
- Di sisi server (CI4), API akan<br>
  ✔ Periksa apakah email sudah terdaftar<br>
  ✔ Enkripsi password sebelum simpan ke database<br>
  ✔ Insert data user baru<br>
  ✔ Kirim respon JSON<br>
  jika berhasil maka akan ada message registrasi berhasil, jika gagal akan ada message registrasi gagal.
- Flutter Parsing JSON Response & Tampilkan Dialog.
  ```
  var jsonObj = json.decode(response.body);
  return Registrasi.fromJson(jsonObj);
  ```
  jika sukses -> tampil dialog sukses. jika gagal -> tampil dialog gagal.
- Setelah sukses, user dialihkan ke halaman login dan harus login dengan akun yang sudah dibuat. Karena: Token hanya dibuat saat login, bukan saat registrasi.

  <br><br>
### Proses Login
<img width="378" height="504" alt="Screenshot 2025-12-02 212442" src="https://github.com/user-attachments/assets/8e654921-7292-453e-b19f-ca27e4a75945" />
<img width="378" height="504" alt="Screenshot 2025-12-02 215701" src="https://github.com/user-attachments/assets/b143657e-8d59-45c5-8d49-d301c8aab44e" />
<br><br>

- User memasukkan Email & Password di Aplikasi Flutter. Lalu klik tombol Login akan memicu fungsi
  ```
  LoginBloc.login(
      email: _emailTextboxController.text,
      password: _passwordTextboxController.text,
    )
  ```
  kemudian memeriksa validasi dari inputan tadi.
  <br>
- Flutter mengirim Request HTTP POST ke Endpoint Login CI4<br>
  Dari LoginBloc:
  ```
  String apiUrl = ApiUrl.login;
  var body = {"email": email, "password": password};
  var response = await Api().post(apiUrl, body);
  ```
  Endpoint di ApiUrl.login = `/login`. Request dikirim via Helper `Api().post()`. Jika sudah login sebelumnya, request akan kirim token juga di header.
- CodeIgniter 4 Menerima Request & Validasi<br>
  API CI4 akan:<br>
  ✔ Cek apakah email ada<br>
  ✔ Cocokkan password<br>
  ✔ Jika benar akan membuatkan token (biasanya JWT)<br>
  ✔ Balikan response JSON
- Flutter Parsing Data JSON Response<br>
  pada loginbloc
  ```
  var jsonObj = json.decode(response.body);
  return Login.fromJson(jsonObj);
  ```
  Diproses menjadi Model Login.
- Token dan UserID Disimpan di SharedPreferences<br>
  Jika login sukses:
  ```
  await UserInfo().setToken(value.token.toString());
  await UserInfo().setUserID(int.parse(value.userID.toString()));
  ```
  Disimpan sebagai session untuk request berikutnya. Token dipakai sebagai Authorization Bearer Token saat akses Produk API
- Redirect Masuk Halaman Produk<br>
  karena token sudah ada maka user dianggap login dan masuk ke halaman produk.
  ```
  Navigator.pushReplacement(context,
  MaterialPageRoute(builder: (context) => const ProdukPage()));
  ```
  <br><br>
### Proses CRUD Produk
CRUD dilakukan melalui komunikasi HTTP antara Flutter dan CodeIgniter 4. Semua request menggunakan token yang diperoleh saat login akan dikirim via header Authorization. Token dikirim menggunakan helper `Api()` yang sudah dibuat.<br>
<img width="378" height="504" alt="Screenshot 2025-12-02 212512" src="https://github.com/user-attachments/assets/7ca212c2-738e-4e78-8900-1b0d78e1d32e" />
<img width="378" height="504" alt="Screenshot 2025-12-02 212504" src="https://github.com/user-attachments/assets/b1d8060f-1c67-46fb-8aea-06dc6042997d" />
<img width="378" height="504" alt="Screenshot 2025-12-02 212518" src="https://github.com/user-attachments/assets/68e48f7b-e5bd-4c25-9935-9906355a0e5f" />
<img width="378" height="504" alt="Screenshot 2025-12-02 212538" src="https://github.com/user-attachments/assets/33dacaf3-f43b-4339-84f4-e3a82c3e9088" />
<img width="378" height="504" alt="Screenshot 2025-12-02 212543" src="https://github.com/user-attachments/assets/3c203a14-e2e2-485b-ba8a-13be49a28b46" />
<img width="378" height="504" alt="Screenshot 2025-12-02 220518" src="https://github.com/user-attachments/assets/2038645c-e6c6-4e84-a16d-e3330deb4a62" />
<br><br>

- 📃Read (Menampilkan List Produk)
  Bloc yang dipanggil yaitu `ProdukBloc.getProduks()` :
  ```
  static Future<List<Produk>> getProduks() async {
    String apiUrl = ApiUrl.listProduk;
    var response = await Api().get(apiUrl);
    var jsonObj = json.decode(response.body);
    List<dynamic> listProduk = (jsonObj as Map<String, dynamic>)['data'];
    List<Produk> produks = [];
    for (int i = 0; i < listProduk.length; i++) {
      produks.add(Produk.fromJson(listProduk[i]));
    }
    return produks;
  }
  ```
  Kode request ke API:
  ```
  var response = await Api().get(ApiUrl.listProduk);
  ```
  API mengembalikan daftar produk JSON. Flutter parsing -> ditampilkan di ListView (ListProduk).<br>
  Di UI, diberikan FutureBuilder untuk menunggu respons dari server<br>
  😁 Saat token valid -> data ditampilkan<br>
  😿 Jika token invalid -> API akan menolak
- ➕Create (Tambah Produk) <br>
  Saat user submit form `ProdukBloc.addProduk(produk: createProduk)`, data dikirim dengan POST::
  ```
  var body = {
    "kode_produk": produk!.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString(),
  };

  var response = await Api().post(apiUrl, body);
  var jsonObj = json.decode(response.body);
  return jsonObj['status'];
  }
  ```
  😁 Jika berhasil -> kembali ke List Produk <br>
  😿 Jika gagal -> tampil WarningDialog
- 📝 Update (Mengubah Produk)<br>
  Saat membuka form dari detail produk -> field otomatis terisi. cek kondisi `if(widget.produk != null)` untuk edit.<br>
  Memanggil `ProdukBloc.updateProduk(produk: updateProduk)` untuk edit produk:
  ```
  static Future updateProduk({required Produk produk}) async {
    String apiUrl = ApiUrl.updateProduk(int.parse(produk.id!));
    print(apiUrl);

    var body = {
      "kode_produk": produk.kodeProduk,
      "nama_produk": produk.namaProduk,
      "harga": produk.hargaProduk.toString(),
    };
    print("Body : $body");
    var response = await Api().put(apiUrl, jsonEncode(body));
    var jsonObj = json.decode(response.body);
    return jsonObj['status'];
  }
  ```
  Request PUT ke endpoint: `var response = await Api().put(apiUrl, jsonEncode(body));`. <br>
  😸 Jika sukses maka akan redirect kembali ke List Produk.
- 🚮 Delete (Hapus Produk)<br>
  Dari halaman `produk_detail.dart` klik tombol Delete, akan tampil konfirmasi, hapus jika "Ya"
  ```
  ProdukBloc.deleteProduk(id: int.parse(widget.produk!.id!))
  ```
  Request DELETE terkirim:
  ```
  var response = await Api().delete(apiUrl);
  ```
  😁 Jika berhasil -> kembali ke list <br>
  😿 Jika gagal -> tampil WarningDialog.
  <br><br>

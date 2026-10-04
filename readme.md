# pertemuan 2

pada pertemuan 2 saya mempelajari penerapan computational thinking (abstraksi, dekomposisi, dan algoritma) pada dart menggunakan studi kasus sistem pembayaran e-wallet.

1. abstraksi: membuat enum `StatusPembayaran` untuk mendefinisikan status transaksi dan class `EWallet` untuk menyimpan data akun (nama, pin, saldo, transaksi harian, dan percobaan pin).
2. dekomposisi: memecah logika pengecekan menjadi function terpisah seperti `cekPin()`, `cekSaldo()`, dan `cekLimit()`.
3. algoritma: menggabungkan semua validasi di function `pembayaran()` agar saldo hanya terpotong jika pin benar, saldo cukup, dan transaksi tidak melebihi limit harian.

selain itu, kode ini juga diuji dengan 5 skenario (transaksi sukses, saldo kurang, lewat limit, akun terblokir jika pin salah 3 kali, dan pin salah lalu benar).

### Kode Implementasi
```dart
enum StatusPembayaran {
  berhasil,
  pinSalah,
  akunTerblokir,
  saldoTidakCukup,
  limitTercapai,
}

class EWallet {
  final String nama;
  final int pin;
  double saldo;
  double transaksiHariIni;
  int percobaanPin;

  EWallet({
    required this.nama,
    required this.pin,
    required this.saldo,
    this.transaksiHariIni = 0,
    this.percobaanPin = 0,
  });
}

StatusPembayaran cekPin(
  EWallet akun,
  List<int> inputPin,
) {
  if (akun.percobaanPin >= 3) {
    return StatusPembayaran.akunTerblokir;
  }

  for (final pinInput in inputPin) {
    if (pinInput == akun.pin) {
      akun.percobaanPin = 0;
      return StatusPembayaran.berhasil;
    }

    akun.percobaanPin++;

    if (akun.percobaanPin >= 3) {
      return StatusPembayaran.akunTerblokir;
    }
  }

  return StatusPembayaran.pinSalah;
}

bool cekSaldo(EWallet akun, double nominal) {
  return nominal <= akun.saldo;
}

bool cekLimit(
  EWallet akun,
  double nominal,
  double limitHarian,
) {
  return akun.transaksiHariIni + nominal <= limitHarian;
}

StatusPembayaran pembayaran(
  EWallet akun,
  List<int> inputPin,
  double nominal,
  double limitHarian,
) {
  final statusPin = cekPin(akun, inputPin);

  if (statusPin == StatusPembayaran.akunTerblokir) {
    return StatusPembayaran.akunTerblokir;
  }

  if (statusPin == StatusPembayaran.pinSalah) {
    return StatusPembayaran.pinSalah;
  }

  if (!cekSaldo(akun, nominal)) {
    return StatusPembayaran.saldoTidakCukup;
  }

  if (!cekLimit(akun, nominal, limitHarian)) {
    return StatusPembayaran.limitTercapai;
  }

  akun.saldo -= nominal;
  akun.transaksiHariIni += nominal;

  return StatusPembayaran.berhasil;
}

String tampilkanHasil(StatusPembayaran status) {
  return switch (status) {
    StatusPembayaran.berhasil => 'Pembayaran berhasil',
    StatusPembayaran.pinSalah => 'Gagal: PIN salah',
    StatusPembayaran.akunTerblokir =>
      'Gagal: PIN salah 3 kali, akun diblokir',
    StatusPembayaran.saldoTidakCukup =>
      'Gagal: saldo tidak mencukupi',
    StatusPembayaran.limitTercapai =>
      'Gagal: melebihi limit transaksi harian',
  };
}

void main() {
  const double limitHarian = 2000000;

  final akun1 = EWallet(
    nama: 'Budi',
    pin: 1234,
    saldo: 3000000,
    transaksiHariIni: 500000,
  );

  final akun2 = EWallet(
    nama: 'Citra',
    pin: 5678,
    saldo: 500000,
    transaksiHariIni: 500000,
  );

  final akun3 = EWallet(
    nama: 'Doni',
    pin: 1111,
    saldo: 3000000,
    transaksiHariIni: 1000000,
  );

  final akun4 = EWallet(
    nama: 'Eka',
    pin: 2222,
    saldo: 3000000,
    transaksiHariIni: 100000,
  );

  final akun5 = EWallet(
    nama: 'Fani',
    pin: 3333,
    saldo: 3000000,
    transaksiHariIni: 100000,
  );

  print('\n Skenario 1');
  print(
    tampilkanHasil(
      pembayaran(akun1, [1234], 500000, limitHarian),
    ),
  );
  print('Saldo: Rp${akun1.saldo.toStringAsFixed(0)}');

  print('\n Skenario 2');
  print(
    tampilkanHasil(
      pembayaran(akun2, [5678], 1000000, limitHarian),
    ),
  );

  print('\n Skenario 3');
  print(
    tampilkanHasil(
      pembayaran(akun3, [1111], 1500000, limitHarian),
    ),
  );

  print('\n Skenario 4');
  print(
    tampilkanHasil(
      pembayaran(akun4, [9999, 8888, 7777], 500000, limitHarian),
    ),
  );

  print('\n Skenario 5');
  print(
    tampilkanHasil(
      pembayaran(akun5, [9999, 8888, 3333], 500000, limitHarian),
    ),
  );
}
```

### Penjelasan Fungsional 

- **Abstraksi**: Meringkas kerumitan dengan `enum StatusPembayaran` untuk hasil pasti dan class `EWallet` sebagai satu cetakan identitas dan status dompet.
- **Dekomposisi**: Menghindari satu fungsi besar yang memusingkan dengan memisahkan validasi (`cekPin`, `cekSaldo`, `cekLimit`). Mengganti aturan cukup di satu fungsi.
- **Algoritma**: Alur `pembayaran()` diurutkan dengan logis dari blokir -> cek pin -> cek saldo -> cek limit -> eksekusi potong saldo. Fail fast (gagal lebih awal sebelum proses rumit).

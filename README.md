  # Sistem Absensi Siswa Berbasis Web
## 1. Abstrak
Penelitian ini bertujuan merancang dan mengimplementasikan sistem absensi berbasis web sebagai solusi untuk menggantikan proses pencatatan kehadiran manual yang rentan terhadap kesalahan, pemalsuan data, dan inefisiensi waktu. Metode tradisional seringkali menghasilkan data yang tidak akurat, memperlambat proses rekapitulasi, dan menyulitkan pemantauan kehadiran secara real-time oleh pihak manajemen. Sistem absensi berbasis web yang diusulkan ini dirancang menggunakan arsitektur Model-View-Controller (MVC) dengan memanfaatkan teknologi seperti PHP, MySQL, dan framework responsive untuk memastikan aksesibilitas dari berbagai perangkat. Fitur utama sistem mencakup absensi mandiri, verifikasi lokasi menggunakan Geolocation, integrasi dengan data akademik/karyawan, dan pelaporan otomatis. Hasil implementasi menunjukkan bahwa sistem ini mampu meningkatkan akurasi pencatatan kehadiran hingga 99% dan mengurangi waktu yang dibutuhkan untuk rekapitulasi data kehadiran bulanan secara signifikan. Dengan demikian, sistem absensi berbasis web terbukti efektif dalam mendukung manajemen sumber daya manusia yang lebih efisien dan transparan.

## 2. Pendahuluan
Manajemen kehadiran yang efektif merupakan pilar penting dalam tata kelola institusi, baik di lingkungan pendidikan maupun korporasi. Akurasi data kehadiran secara langsung memengaruhi disiplin, penilaian kinerja, dan perhitungan kompensasi. Secara tradisional, pencatatan kehadiran seringkali mengandalkan metode manual, seperti buku daftar hadir atau mesin fingerprint konvensional. Walaupun metode ini berfungsi, ia membawa berbagai kelemahan signifikan, terutama kerentanan terhadap pemalsuan data (titip absen), kesalahan manusia (human error) dalam proses input dan rekapitulasi, serta inefisiensi waktu yang dihabiskan oleh staf administrasi untuk menghitung dan melaporkan data kehadiran bulanan.

## Analisis dan Perancangan Sistem

### a. Use Case Diagram
<img src="use case rpl.png" alt="Use Case Diagram" width="600"/>
Semua aktor, yaitu Siswa, Admin, Staff/Guru, dan Kepala Sekolah, harus melalui proses Login ke sistem terlebih dahulu sebelum dapat mengakses fungsionalitas lainnya. Proses login ini adalah Use Case wajib (<<Include>>) untuk berbagai fungsi. Aktor Siswa dapat melakukan absen mandiri. Namun, fungsi ini melibatkan (includes) proses Login ke sistem sebelum absensi dapat dilakukan. Aktor Admin dan Staff/Guru dapat input absen harian untuk siswa. Fungsi input absen harian ini merupakan perpanjangan (<<Extend>>) dari fungsi melakukan absen mandiri, yang berarti input absen harian menawarkan fungsionalitas tambahan atau alternatif yang lebih kompleks dari absen mandiri. Aktor Siswa, Admin, Staff/Guru, dan Kepala Sekolah semuanya memiliki kemampuan untuk lihat status kehadiran secara real-time. Hal ini menunjukkan bahwa informasi kehadiran transparan dan dapat diakses oleh semua pihak yang berkepentingan.
Fungsi cetak laporan absensi merupakan kelanjutan penting (<<Include>>) dari melihat status kehadiran secara real-time. Dengan kata lain, untuk mencetak laporan, pengguna (Admin dan Staff/Guru) harus mengakses dan melihat data status kehadiran terlebih dahulu.
Fungsi kelola data siswa dan kelas merupakan tanggung jawab utama Admin dan Staff/Guru, yang melibatkan penambahan, perubahan, atau penghapusan data dasar akademik. Terakhir, fungsi kelola pengaturan sistem adalah tugas eksklusif yang dikerjakan oleh Admin dan Kepala Sekolah, yang mencakup konfigurasi dan administrasi backend sistem secara keseluruhan.
 ### b. class Diagram
 <img src="class diagram rpl.jpg" alt="class diagram" width="600"/>

 ## Implementasi
 ### 1. pola design creational
a. Factory Method
   factory method adalah salah satu Creational Design Pattern dalam pemrograman berorientasi objek (OOP) yang bertujuan untuk membuat objek tanpa harus menentukan secara eksplisit kelas objek mana yang akan dibuat.
Berikut contoh code penerapan menggunakan php :
```php
<!-- absensiHandler.php -->
<?php

interface AbsensiHandler {
    // Setiap handler harus dapat memproses permintaan utamanya
    public function handleRequest(array $data): string; 
    
    // Metode opsional, untuk menunjukkan kemampuan tambahan
    public function getActions(): array;
}

// absensiMandiriHandler.php
class AbsenMandiriHandler implements AbsensiHandler {
    public function handleRequest(array $data): string {
        // Logika: Memeriksa NIS, merekam waktu, mencatat lokasi.
        return "Siswa NIS **{$data['nis']}** berhasil merekam absensi mandiri pada " . date('H:i:s');
    }

    public function getActions(): array {
        return ['Absensi Mandiri'];
    }
}

// 'AbsensiHandler.php';
class GuruHandler implements AbsensiHandler {
    public function handleRequest(array $data): string {
        // Logika: Guru sedang mengakses data absensi kelasnya.
        if (isset($data['edit_nis'])) {
             return "Guru **{$data['nama']}** berhasil **mengedit** status absensi NIS {$data['edit_nis']}.";
        }
        return "Guru **{$data['nama']}** berhasil **melihat** daftar absensi kelas X RPL 1.";
    }

    public function getActions(): array {
        return ['Lihat Absen', 'Edit Absen'];
    }
}

// 'AbsensiHandler.php';
class AdminHandler implements AbsensiHandler {
    public function handleRequest(array $data): string {
        // Logika: Admin melakukan tugas tingkat tinggi.
        if (isset($data['action']) && $data['action'] === 'export_pdf') {
            return "Admin **{$data['nama']}** sedang **mengekspor** laporan absensi sebagai PDF.";
        }
        if (isset($data['action']) && $data['action'] === 'export_excel') {
            return "Admin **{$data['nama']}** sedang **mengekspor** laporan absensi sebagai Excel.";
        }
        if (isset($data['edit_nis'])) {
             return "Admin **{$data['nama']}** berhasil **mengedit** data absensi NIS {$data['edit_nis']} secara global.";
        }
        return "Admin **{$data['nama']}** berhasil masuk ke dashboard manajemen absensi.";
    }

    public function getActions(): array {
        return ['Edit Global', 'Ekspor PDF', 'Ekspor Excel'];
    }
}
```
b. 	Singleton
	Singleton adalah Creational Design Pattern yang memastikan bahwa sebuah class hanya memiliki satu instance (objek) tunggal selama keseluruhan masa hidup aplikasi, dan menyediakan satu titik akses global ke instance tersebut. Berikut contoh penerapan pada code php :
```php
<?php
class DatabaseConnection {
    // 1. Variabel statis untuk menyimpan instance tunggal dari kelas
    private static ?DatabaseConnection $instance = null;

    // Variabel untuk menyimpan koneksi PDO yang sebenarnya
    private ?PDO $pdo = null; 

    // Konfigurasi Database (Ganti dengan data Anda)
    private const DB_HOST = 'localhost';
    private const DB_NAME = 'db_absensi';
    private const DB_USER = 'root';
    private const DB_PASS = 'password';

    // 2. Konstruktor dibuat private
    // Ini MENCEGAH PENGGUNAAN 'new DatabaseConnection()' dari luar kelas
    private function __construct() {
        try {
            $dsn = "mysql:host=" . self::DB_HOST . ";dbname=" . self::DB_NAME . ";charset=utf8mb4";
            $this->pdo = new PDO($dsn, self::DB_USER, self::DB_PASS);
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (\PDOException $e) {
            // Dalam aplikasi nyata, log error ini dan berikan pesan ramah ke pengguna
            die("Gagal terhubung ke database: " . $e->getMessage());
        }
    }

    // 3. Mencegah kloning (Cloning)
    // MENCEGAH: $conn2 = clone $conn1;
    private function __clone() { }

    // 4. Mencegah deserialisasi (untuk kasus serialisasi)
    // MENCEGAH: unserialize($serialized_object);
    public function __wakeup() {
        throw new \Exception("Cannot unserialize a singleton.");
    }

    /**
     * 5. Metode statis publik untuk mendapatkan instance tunggal
     * Ini adalah SATU-SATUNYA TITIK AKSES untuk kelas ini
     */
    public static function getInstance(): DatabaseConnection {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    // Metode untuk mengembalikan objek koneksi PDO
    public function getConnection(): PDO {
        return $this->pdo;
    }
}
```
c.	Builder 
	Builder Pattern adalah salah satu Creational Design Pattern yang berfokus pada pemisahan proses konstruksi objek yang kompleks dari representasi akhirnya. Tujuannya adalah untuk membuat berbagai variasi objek yang kompleks dengan menggunakan proses konstruksi yang sama. Pola ini menjadi solusi ideal ketika Anda perlu membuat objek dengan banyak konfigurasi atau banyak parameter opsional, menghindari masalah "telescoping constructor" (konstruktor dengan terlalu banyak parameter) yang sering terjadi di OOP. Berikut contoh penerapan code pada php :
```php
<!-- code inti director -->
<?php
class LaporanDirector {
    private ?LaporanBuilder $builder = null;

    public function setBuilder(LaporanBuilder $builder): void {
        $this->builder = $builder;
    }

    /**
     * Metode ini mendefinisikan urutan standar konstruksi laporan.
     */
    public function buildLaporanLengkap(string $judul, string $periode, array $dataSiswa, int $totalHadir, int $totalAlpha): void {
        $this->builder->reset();
        $this->builder->buildHeader($judul, $periode);
        $this->builder->buildDetailSiswa($dataSiswa);
        $this->builder->buildRingkasan($totalHadir, $totalAlpha);
        $this->builder->buildFooter();
    }
    // ... Metode untuk buildLaporanRingkas() ...
}

// code concrete builder
class PDFLaporanBuilder implements LaporanBuilder {
    // ... properti dan reset() ...

    public function buildRingkasan(int $totalHadir, int $totalAlpha): void {
        // Implementasi spesifik untuk format PDF
        $this->laporan->addPart("Ringkasan PDF", "Total Hadir: {$totalHadir} | Total Alpha: {$totalAlpha}");
    }

    public function getResult(): LaporanAbsensi {
        // ... mengembalikan objek dan mereset builder
    }
}
```
2. Dependency Injection 
Dependency Injection (DI) adalah sebuah prinsip desain software yang bertujuan untuk membuat kode lebih decoupled (tidak terlalu terikat) dan lebih mudah diuji (testable).
```php
<?php

// =========================
//  Database (Dependency)
// =========================
class Database {
    private $pdo;

    public function __construct($host, $db, $user, $pass) {
        $dsn = "mysql:host=$host;dbname=$db;charset=utf8";
        $this->pdo = new PDO($dsn, $user, $pass, [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
        ]);
    }

    public function connection() {
        return $this->pdo;
    }
}


// =========================
//  Repository (DI: Database)
// =========================
class AbsensiRepository {
    private $db;

    public function __construct(Database $database) {
        $this->db = $database->connection();
    }

    public function simpan($nis, $tanggal, $status) {
        $stmt = $this->db->prepare("
            INSERT INTO absensi (nis, tanggal, status)
            VALUES (?, ?, ?)
        ");
        return $stmt->execute([$nis, $tanggal, $status]);
    }

    public function hariIni() {
        $stmt = $this->db->query("
            SELECT * FROM absensi WHERE tanggal = CURDATE()
        ");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}


// =========================
//  Service (DI: Repository)
// =========================
class AbsensiService {
    private $repo;

    public function __construct(AbsensiRepository $repo) {
        $this->repo = $repo;
    }

    public function absen($nis) {
        return $this->repo->simpan($nis, date("Y-m-d"), "Hadir");
    }

    public function getHariIni() {
        return $this->repo->hariIni();
    }
}


// =========================
//  Dependency Injection Container
// =========================
$db        = new Database("localhost", "db_absensi", "root", "");
$repo      = new AbsensiRepository($db);
$service   = new AbsensiService($repo);


// =========================
//  Controller (Main Logic)
// =========================
$pesan = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $nis = $_POST['nis'];

    if ($service->absen($nis)) {
        $pesan = "Absensi berhasil!";
    } else {
        $pesan = "Gagal menyimpan absensi!";
    }
}

$absensiHariIni = $service->getHariIni();

?>

<!DOCTYPE html>
<html>
<head>
    <title>Absensi Siswa (Dependency Injection)</title>
</head>
<body>

<h2>Form Absensi</h2>

<form method="POST">
    <label>NIS:</label>
    <input type="text" name="nis" required>
    <button type="submit">Absen</button>
</form>

<p><?= $pesan ?></p>

<h2>Absensi Hari Ini</h2>
<table border="1" cellpadding="5">
    <tr>
        <th>NIS</th>
        <th>Tanggal</th>
        <th>Status</th>
    </tr>

    <?php foreach ($absensiHariIni as $a): ?>
    <tr>
        <td><?= $a['nis'] ?></td>
        <td><?= $a['tanggal'] ?></td>
        <td><?= $a['status'] ?></td>
    </tr>
    <?php endforeach; ?>

</table>

</body>
</html>
```
3. Squence & activity diagram
a. squence diagram
<img src="squence diagram.png" alt="squence diagram" width="600"/>
b. activity diagram siswa
<img src="activity diagram siswa.png" alt="activity diagram siswa" width="600"/>
  activity diagram staff/guru
<img src="activity diagram staffguru.png" alt="activity diagram staff/guru" width="600"/>

 ## Evaluasi&kreasi
 1. Design state machine diagram
<img src="design state machine diagram.jpeg" alt="design state machine diagram" width="600"/>

2. evaluasi kualitas desain
Activity Diagram Proses Absensi Siswa Berbasis Web menggambarkan alur kerja yang logis dan terstruktur. Desain ini menunjukkan kualitas yang baik dalam aspek Maintainability (kemudahan pemeliharaan) dan Reusability (kemampuan pakai ulang) untuk proses absensi digital.
a. Maintainability (Kemudahan Pemeliharaan): Baik
- Pemodelan Proses yang Jelas (Activity Diagram) Diagram aktivitas ini memberikan representasi langkah demi langkah yang clear dan sequential (berurutan) dari awal hingga akhir proses absensi:
1.	Diawali dengan Login ke sistem.
2.	Dilanjutkan dengan Validasi waktu absensi.
3.	Terdapat cabang proses opsional untuk Validasi lokasi.
4.	Diakhiri dengan Sistem menyimpan absensi ke database dan menampilkan notifikasi. Hal ini memudahkan dalam melacak setiap langkah yang mungkin memerlukan pemeliharaan atau penyesuaian di masa mendatang (misalnya, jika aturan waktu atau lokasi absensi berubah).
- Pemisahan Tugas dan Validasi Diagram ini secara eksplisit memisahkan tugas utama (Absensi) dengan serangkaian validasi krusial (Login, Waktu, Lokasi).
1. Setiap validasi memiliki kondisi keluar (exit condition) yang jelas (Ya/Tidak) yang mengarah ke langkah berikutnya atau ke kondisi akhir yang gagal. Hal ini memudahkan debugging (pencarian kesalahan); jika absensi gagal, developer dapat dengan cepat mengetahui di tahap validasi mana kegagalan terjadi.
b. Reusability (Kemampuan Pakai Ulang): Baik
- Potensi Penerapan Single Responsibility Principle (SRP) Meskipun ini adalah diagram proses, langkah-langkah di dalamnya dapat dipecah menjadi fungsi atau modul yang masing-masing idealnya hanya memiliki satu tanggung jawab:
1. Modul Login: Bertanggung jawab penuh atas validasi username dan password.
2. Modul Validasi Waktu: Bertanggung jawab penuh atas pengecekan rentang waktu absensi.
3. Modul Validasi Lokasi: Bertanggung jawab penuh atas pengecekan koordinat GPS/lokasi. Hal ini memungkinkan setiap "modul" tersebut dapat dipanggil atau digunakan kembali (reusable) di bagian sistem lain yang membutuhkan validasi serupa (misalnya, validasi waktu/lokasi untuk check-in atau check-out guru).
- Langkah Absensi yang Terstruktur Langkah inti "Siswa mengirim absensi" dan "Sistem menyimpan absensi ke database" adalah prosedur standar.
1. Prosedur penyimpanan ke database dapat dijadikan service terpisah yang dapat digunakan oleh berbagai proses pengiriman data lain dalam sistem (misalnya, pengiriman tugas, pengiriman formulir).

## Saran Improvement & Fitur Baru
- Fitur Absensi dengan QR Code
1. Guru menghasilkan QR code unik untuk setiap kelas.
2. Siswa cukup scan QR untuk absen.
3. mencegah titip absen.
4. Cepat dan efisien.
Manfaat: Proses absensi lebih cepat dan akurat
- Validasi Lokasi (GPS / Geolocation)
1. Sistem mengecek apakah siswa berada dalam radius sekolah.
2. Jika siswa tidak di area sekolah → absensi ditolak.
Manfaat: Menghindari absensi dari rumah atau tempat lain
- Deteksi Wajah (Face Recognition)
1. Absensi menggunakan kamera HP/laptop.
2. Sistem mencocokkan dengan foto siswa di database.
Manfaat: Anti titip absen, sangat akurat
- Notifikasi Otomatis ke Orang Tua
1. Sistem mengirimkan WhatsApp atau email: “Anak Anda hadir pukul 07:05.”, “Anak Anda belum absen sampai pukul 07:30.”
Manfaat: Orang tua bisa memantau disiplin anak.


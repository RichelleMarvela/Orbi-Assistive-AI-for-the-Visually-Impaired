# Orbi — Assistive AI for the Visually Impaired

Ringkasan singkat
-----------------
Orbi adalah sebuah aplikasi asistif berbasis Python/Flask yang membantu pengguna tunanetra dengan fungsi-fungsi seperti:
- Pembacaan teks dari kamera (OCR & TTS).
- Deteksi objek otomatis (mode deteksi yang bisa dihidupkan/matikan).
- Logging aktivitas dan lokasi pengguna.
- Panel admin untuk melihat log aktivitas pengguna.

Project ini bertujuan memberikan prototipe cepat (proof-of-concept) untuk fitur-fitur assistive AI di perangkat berkemampuan kamera.

Fitur utama
-----------
- Autentikasi sederhana (register/login) dengan role admin/user.
- Database SQLite untuk menyimpan pengguna dan aktivitas (ActivityLog).
- Pembacaan teks real-time dari webcam menggunakan EasyOCR + pyttsx3.
- Mode deteksi objek yang dijalankan melalui subprocess.
- Reverse geocoding (OpenStreetMap/Nominatim) untuk menyimpan lokasi pengguna.
- Text-to-speech (pyttsx3) untuk mengumumkan hasil kepada pengguna.
- Endpoint REST untuk perintah suara, deteksi otomatis, dan logging lokasi.

Struktur proyek (ringkasan)
--------------------------
- AOL_ProjectAI_AdminUserPage/
  - app3.py            — Aplikasi Flask utama (routes, DB models, logika perintah suara)
  - readtext.py        — Skrip OCR + TTS untuk membaca teks dari webcam menggunakan EasyOCR
  - detectobject2.py   — (tidak termasuk di repo ini di preview; diasumsikan ada) skrip deteksi objek
  - templates/         — (tidak ditampilkan di preview) template HTML: login.html, register.html, admin.html, user.html
  - static/            — (opsional) file CSS/JS/assets
- README.md           — (file ini)

Teknologi & dependencies
------------------------
- Bahasa: Python 3.8+ (direkomendasikan 3.9+)
- Framework: Flask
- Database: SQLite (melalui SQLAlchemy)
- OCR: EasyOCR
- Computer Vision: OpenCV (cv2)
- Text-to-Speech: pyttsx3
- HTTP requests: requests
- Timezones: pytz
- Keamanan password: werkzeug.security (generate_password_hash, check_password_hash)
- Multiprocessing & subprocess untuk menjalankan proses latar seperti OCR/deteksi

Rekomendasi paket (requirements.txt)
------------------------------------
Berikut daftar paket yang umumnya dibutuhkan (sesuaikan versi jika perlu):

Flask
Flask_SQLAlchemy
Werkzeug
pyttsx3
opencv-python
easyocr
torch      # easyocr memerlukan PyTorch
requests
pytz

Contoh minimal requirements.txt:
```
Flask>=2.0
Flask_SQLAlchemy>=2.5
Werkzeug>=2.0
pyttsx3>=2.90
opencv-python>=4.5
easyocr>=1.6
torch>=1.8
requests>=2.25
pytz>=2021.1
```

Persyaratan sistem & catatan instalasi
--------------------------------------
- Pastikan Python 3.8+ terpasang.
- Untuk OpenCV (webcam) pastikan akses kamera diizinkan (Linux: /dev/video*, Windows: device driver).
- EasyOCR memerlukan PyTorch. Instal PyTorch sesuai OS/GPU/CPU:
  - CPU-only (contoh): pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu
  - Atau ikuti instruksi resmi di https://pytorch.org/
- pyttsx3 menggunakan engine TTS lokal (sapi5 di Windows, espeak di Linux). Pastikan engine TTS tersedia.
- Jika ingin performa OCR lebih cepat, gunakan GPU (instal versi torch yang mendukung CUDA dan easyocr GPU=True).
- Nominatim (reverse geocode) memiliki rate limit. Gunakan cache bila perlu untuk produksi.

Setup lokal (langkah demi langkah)
---------------------------------
1. Clone repo:
   git clone https://github.com/RichelleMarvela/Orbi-Assistive-AI-for-the-Visually-Impaired.git
   cd Orbi-Assistive-AI-for-the-Visually-Impaired/AOL_ProjectAI_AdminUserPage

2. Buat virtual environment dan aktifkan:
   - Linux/Mac:
     python3 -m venv venv
     source venv/bin/activate
   - Windows:
     python -m venv venv
     venv\Scripts\activate

3. Instal dependencies:
   pip install -r requirements.txt
   (Jika tidak ada file requirements.txt, instal paket yang tercantum di bagian requirements)

4. Jalankan aplikasi Flask:
   python app3.py
   - Aplikasi akan berjalan pada http://127.0.0.1:5000 dengan debug=True (default pada kode).
   - Saat pertama kali dijalankan, database SQLite (`orbi.db`) akan dibuat otomatis dan akun default dibuat:
     - admin / admin (role: admin)
     - user / user (role: user)

5. Menjalankan pembaca teks (readtext.py) secara terpisah:
   python readtext.py
   - Akan membuka webcam, tekan `c` untuk capture & mendeteksi teks, `q` untuk keluar.

Endpoint & penggunaan
----------------------
Web interface:
- /            — Halaman login.
- /register    — Halaman registrasi pengguna baru.
- /dashboard   — Redirect ke halaman sesuai role.
- /user        — Panel user.
- /admin       — Panel admin (lihat log pengguna, cari username via querystring `?username=<name>`).
- /logout      — Logout.

API endpoints (JSON):
- POST /process_voice
  - Body: JSON { "command": "<perintah suara>" }
  - Fungsi: memproses perintah suara; contoh perintah:
    - "hello" atau "hey orbi" -> sapaan
    - "detect object" -> aktifkan mode deteksi
    - "stop detecting object" -> hentikan deteksi
    - "read text" -> menjalankan `readtext.py` (subprocess)
  - Hanya dapat dipanggil oleh user yang sudah login (session-based).

- GET /auto_detect
  - Memanggil skrip deteksi objek `detectobject2.py` sebagai subprocess bila mode deteksi aktif.
  - Mengembalikan hasil deteksi (JSON).

- POST /log_location
  - Body: JSON { "latitude": <float>, "longitude": <float> }
  - Menyimpan lokasi user via reverse_geocode() ke database sebagai ActivityLog.

Contoh penggunaan curl (memerlukan cookie session dari login):
- Login via web lebih mudah. Untuk testing API tanpa sesi, Anda dapat menonaktifkan pengecekan sesi sementara (hanya untuk debugging lokal), tetapi jangan lakukan ini di produksi.

Database & model singkat
------------------------
Model User:
- id, username, password (hashed), role, logs (relationship ke ActivityLog)

Model ActivityLog:
- id, user_id, action, location, timestamp
- Properti helper: local_time mengubah UTC ke timezone Asia/Jakarta

Keamanan & produksi
-------------------
Poin penting sebelum deploy:
- Jangan gunakan app.secret_key yang hard-coded. Gunakan variabel lingkungan (ENV) untuk SECRET_KEY.
- Matikan debug=True di production.
- Gunakan server WSGI production seperti Gunicorn/uvicorn dan reverse proxy (Nginx).
- Pertimbangkan rate limiting dan autentikasi token untuk endpoint API.
- Proteksi endpoint sensitif, sanitasi input, dan validasi payload JSON.
- Tidak disarankan menyimpan password dalam plaintext — kode sudah menggunakan generate_password_hash.

Optimasi & catatan arsitektur
----------------------------
- readtext.py dijalankan sebagai proses terpisah (subprocess) dari Flask; ini membuat UI tidak terblokir.
- Untuk operasi TTS/IO berat, gunakan worker queue (RQ/Celery) di produksi.
- Deteksi objek juga dijalankan lewat subprocess; jika deteksi intensif GPU, jalankan sebagai service terpisah.

Troubleshooting umum
--------------------
- Kamera tidak terbaca: periksa device permission, coba index kamera yang berbeda (cv2.VideoCapture(0) -> 1, 2, ...).
- pyttsx3 tidak mengeluarkan suara: periksa TTS backend (Windows: SAPI5; Linux: espeak). Di beberapa lingkungan headless, TTS tidak akan bekerja.
- EasyOCR error: pastikan PyTorch terinstal dan kompatibel dengan versi easyocr. Jika terjadi error GPU, pasang versi CPU-only atau pasang torch sesuai CUDA.
- `detectobject2.py` tidak ditemukan: pastikan file exist dan return JSON yang valid bila dipanggil.

Contoh pengembangan fitur lanjutan
----------------------------------
- Integrasi model deteksi objek berbasis YOLO/TensorFlow.
- Mode continuous OCR (bukan capture manual) dengan heuristik debounce.
- Integrasi layanan cloud TTS (Google/IBM/AWS) untuk kualitas suara yang lebih baik.
- Enkripsi lokasi & data sensitif, anonymization, dan audit log.

Kontribusi
----------
- Buka issue untuk bug/fitur baru.
- Fork proyek, buat branch feature/fix, lalu buat PR.
- Sertakan deskripsi perubahan dan testing steps.

Lisensi
-------
Tambahkan file LICENSE sesuai pilihan Anda (MIT, Apache-2.0, dsb.). Saat ini repo belum menyertakan lisensi eksplisit — periksa dan tambahkan sesuai kebutuhan.

Catatan developer (spesifik kode)
--------------------------------
- app3.py:
  - Membuat DB saat app context tersedia dan otomatis menyiapkan akun `admin` dan `user` default.
  - Fungsi speak_text menggunakan multiprocessing untuk mencegah blocking saat TTS.
  - get_geolocation memanggil ipinfo.io (gratis namun ada rate limit).
  - reverse_geocode memanggil Nominatim (OpenStreetMap) — gunakan User-Agent dan patuhi usage policy.
- readtext.py:
  - Menangkap frame dari webcam, tekan `c` untuk capture, lalu OCR via EasyOCR.
  - Menambahkan rectangle & teks pada frame yang terdeteksi dan memutar TTS untuk setiap teks unik.

Contoh Quick Start singkat
--------------------------
1. Setup environment & install dependencies.
2. Jalankan `python app3.py`.
3. Buka browser ke http://127.0.0.1:5000
4. Login sebagai admin/admin atau register user baru.
5. Untuk mencoba OCR, klik perintah suara (atau jalankan readtext.py) lalu tekan `c` untuk capture.

Kontak / Credits
----------------
Dibuat oleh: RichelleMarvela (pemilik repo)
Terima kasih kepada:
- Tim EasyOCR & PyTorch
- Kontributor OpenCV, pyttsx3, SQLAlchemy

----------------------------------------
Catatan: README ini dibuat berdasarkan kode yang tersedia (app3.py dan readtext.py). Jika ada file lain (templates, detectobject2.py, static assets), tambahkan bagian khusus pada README agar dokumentasi tetap sinkron dengan isi repo.

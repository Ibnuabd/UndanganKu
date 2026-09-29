# Product Requirements Document (PRD): Platform Undangan Digital Pernikahan

## 1. Executive Summary

- **Problem Statement**: Pasangan pengantin dan vendor (*Wedding Organizer*) di Indonesia membutuhkan solusi yang cepat, elegan, dan andal untuk membuat serta mengelola undangan pernikahan digital secara online tanpa kerumitan teknis.
- **Proposed Solution**: Platform undangan digital berbasis web yang menyediakan berbagai katalog template per kategori dengan sistem *subscription tier*. Platform ini menggunakan alur pemesanan berbasis pengisian form dan konfirmasi pembayaran manual via WhatsApp untuk memudahkan komunikasi personal dengan klien.
- **Success Criteria**:
  - Mencapai **100 undangan aktif** (berhasil di-publish).
  - Waktu muat (*load time*) undangan publik kurang dari 3 detik di jaringan 4G.
  - 100% template dapat diakses dengan baik secara responsif (mobile & desktop).

## 2. User Experience & Functionality

- **User Personas**:
  - **Klien (Pasangan Pengantin / Vendor)**: Mencari, memilih template, mengisi data acara, dan mempublikasikan undangan untuk tamu mereka.
  - **Tamu Undangan**: Menerima tautan undangan publik, melihat detail acara, dan menulis ucapan di *Wall of Wishes*.
  - **Admin (Internal)**: Mengelola master data template, kategori, dan melakukan verifikasi pembayaran/aktivasi undangan.

- **User Stories**:
  - `As a [Klien], I want to [mendaftar dan login menggunakan email-password] so that [saya bisa mengelola undangan saya dengan aman].`
  - `As a [Klien], I want to [melihat katalog template berdasarkan kategori] so that [saya bisa memilih desain yang sesuai dengan tema pernikahan].`
  - `As a [Klien], I want to [mengisi form pemesanan setelah memilih template dan dialihkan ke WhatsApp] so that [saya bisa melakukan konfirmasi pesanan dan pembayaran].`
  - `As a [Klien], I want to [memasukkan detail acara ganda (akad & resepsi), galeri, love story, dan info hadiah digital] so that [tamu mendapatkan informasi yang lengkap].`
  - `As a [Tamu], I want to [membaca informasi acara dan menulis ucapan di Wall of Wishes] so that [saya bisa memberikan doa restu secara digital].`
  - `As an [Admin], I want to [melakukan CRUD pada template dan kategori] so that [platform selalu memiliki desain yang up-to-date].`

- **Acceptance Criteria**:
  - *Auth*: Sistem harus mendukung verifikasi email dan *forgot password*.
  - *Checkout Flow*: Setelah klien memilih template dan mengisi form, sistem harus men-generate tautan `wa.me` dengan pesan *pre-filled* yang berisi detail pesanan untuk dikirim ke admin.
  - *Theme Configuration*: Setiap template memiliki `themeConfig` (warna/tema) bawaan dari sistem yang **TIDAK BISA** diubah (di-override) oleh klien.
  - *Public View*: Halaman undangan publik harus di-generate di *slug* yang unik dan mendukung tampilan *desktop-responsive* serta *mobile-first*.

- **Non-Goals**:
  - Integrasi dengan *Payment Gateway* otomatis (Midtrans, Xendit, dll) belum dilakukan pada fase ini (fokus pada manual transfer bank via WhatsApp).
  - Pembuatan *marketplace* untuk desainer pihak ketiga (panel admin murni dikelola internal).

## 3. Technical Specifications

- **Architecture Overview**: 
  - **Frontend (Client & Public)**: React.js (Single Page Application untuk interaktivitas yang optimal).
  - **Backend (API & Admin)**: Laravel (PHP framework untuk pengelolaan REST API dan manajemen Admin Panel).
  - **Database**: MySQL.
- **Integration Points**: 
  - WhatsApp (Tautan langsung/Deep link menggunakan `wa.me` tanpa API resmi WABA untuk fase MVP).
  - SMTP server untuk pengiriman email transaksional (Verifikasi Email, Lupa Password).
- **Security & Privacy**: 
  - Perlindungan rute publik dan privat menggunakan Laravel Sanctum atau JWT.
  - Validasi ketat pada form untuk mencegah serangan XSS, terutama pada fitur *Wall of Wishes* (input dari tamu publik).
- **Core Data Model** (Draft Relasi Entitas MySQL):
  - **`users`**: `id`, `name`, `email`, `password`, `role` (admin/klien).
  - **`categories`**: `id`, `name`, `slug`.
  - **`templates`**: `id`, `name`, `category_id`, `theme_config` (JSON), `preview_image`, `is_active`.
  - **`invitations`**: `id`, `user_id`, `template_id`, `slug` (unik), `status` (draft/published), `payment_status`.
  - **`couples`**: `id`, `invitation_id`, `groom_name`, `bride_name`, `groom_parents`, `bride_parents`, `love_story` (text).
  - **`events`**: `id`, `invitation_id`, `type` (akad/resepsi), `event_date`, `start_time`, `end_time`, `location_name`, `address_url`.
  - **`galleries`**: `id`, `invitation_id`, `image_url`.
  - **`wishes`** (Wall of Wishes): `id`, `invitation_id`, `guest_name`, `message`, `created_at`.

## 4. Risks & Roadmap

- **Phased Rollout**:
  - **MVP (Fase 1)**: Autentikasi dasar, integrasi Laravel + React, 3-5 template awal, dan alur checkout manual via WhatsApp.
  - **v1.1 (Fase 2)**: Penambahan fitur manajemen *guest book* (buku tamu QR code), dashboard analitik (jumlah viewer).
  - **v2.0 (Fase 3)**: Integrasi *Payment Gateway* otomatis dan kustomisasi font/musik premium.

- **Technical Risks**:
  - *Drop-off Rate*: Karena konfirmasi pembayaran bersifat manual via WhatsApp, ada risiko klien membatalkan pesanan jika admin merespon terlalu lambat. **Mitigasi**: Pastikan ada jam operasional admin yang jelas atau integrasi auto-reply dasar pada WhatsApp penerima.
  - *Skalabilitas Media*: Fitur galeri foto akan memakan banyak penyimpanan (*storage*). **Mitigasi**: Mengimplementasikan pembatasan ukuran file (*max upload size*) dan kompresi gambar otomatis pada sisi Laravel sebelum disimpan ke server/S3.

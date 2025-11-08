# Dokumentasi Prisma ORM untuk Schema MySQL

Dokumentasi ini menjelaskan schema Prisma ORM yang digunakan untuk database MySQL. Schema ini mencakup berbagai model (tabel) yang merepresentasikan data global seperti tenant, user, store, produk, transaksi, dan sistem supplier hybrid. Setiap bagian mencakup nama tabel, deskripsi tabel beserta penjelasan, isi tabel (field-field dengan tipe data, constraints, dan deskripsi), serta relasi antar tabel.

Dokumentasi ini dibuat berdasarkan schema Prisma yang diberikan, dengan fokus pada struktur data, enum, dan hubungan relasional.

## Daftar Enum
Enum adalah tipe data enumerasi yang digunakan dalam model-model.

### Enum: TenantStatus
- Deskripsi: Status tenant, menunjukkan apakah tenant aktif atau tidak.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: TokenType
- Deskripsi: Jenis token untuk autentikasi dan verifikasi user.
- Isi:
  - ACCESS
  - REFRESH
  - RESET_PASSWORD
  - VERIFY_EMAIL

### Enum: StoreStatus
- Deskripsi: Status toko, menunjukkan apakah toko aktif atau tidak.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: StoreUserStatus
- Deskripsi: Status user toko, menunjukkan apakah user aktif di toko tersebut.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: TenantUserStatus
- Deskripsi: Status user tenant, menunjukkan apakah user aktif di tenant tersebut.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: ShiftDay
- Deskripsi: Hari dalam seminggu untuk jadwal shift.
- Isi:
  - MONDAY
  - TUESDAY
  - WEDNESDAY
  - THURSDAY
  - FRIDAY
  - SATURDAY
  - SUNDAY

### Enum: CashStatus
- Deskripsi: Status kas shift, menunjukkan apakah kas pending, dibuka, atau ditutup.
- Isi:
  - PENDING
  - OPENED
  - CLOSED

### Enum: UNIT
- Deskripsi: Satuan unit produk.
- Isi:
  - pcs
  - box

### Enum: VariantScope
- Deskripsi: Scope varian produk, apakah global (template superadmin) atau khusus toko.
- Isi:
  - GLOBAL
  - STORE

### Enum: VariantStatus
- Deskripsi: Status varian, menunjukkan apakah aktif atau tidak.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: IsCash
- Deskripsi: Indikator apakah metode pembayaran adalah tunai.
- Isi:
  - YES
  - NO

### Enum: TransactionStatus
- Deskripsi: Status transaksi, menunjukkan apakah pending, dibayar, atau dibatalkan.
- Isi:
  - PENDING
  - PAID
  - CANCELED

### Enum: StoreBillStatus
- Deskripsi: Status tagihan toko, menunjukkan apakah sudah dibayar atau belum.
- Isi:
  - PAID
  - UNPAID

### Enum: CustomerStatus
- Deskripsi: Status pelanggan, menunjukkan apakah aktif atau tidak.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: SupplierScope
- Deskripsi: Scope supplier, apakah global (template superadmin) atau khusus toko.
- Isi:
  - GLOBAL
  - STORE

### Enum: SupplierStatus
- Deskripsi: Status supplier, menunjukkan apakah aktif atau tidak.
- Isi:
  - ACTIVE
  - INACTIVE

### Enum: PurchaseOrderStatus
- Deskripsi: Status pesanan pembelian, menunjukkan apakah pending, sukses, dikirim, atau dibatalkan.
- Isi:
  - PENDING
  - SUCCESS
  - DELIVERED
  - CANCELED

## Daftar Model (Tabel)

### Model: Tenant
- Nama Tabel: tenants (mapped dari Tenant)
- Deskripsi: Tabel ini menyimpan informasi tenant (penyewa) yang merupakan entitas utama dalam sistem multi-tenant. Tenant mewakili perusahaan atau organisasi yang menggunakan sistem, dengan detail seperti nama, NPWP, alamat, dan status.
- Isi Tabel:

| Field        | Type          | Constraints/Defaults                  | Deskripsi                          |
|--------------|---------------|---------------------------------------|------------------------------------|
| id          | Int           | @id @default(autoincrement())         | ID unik tenant                     |
| userId      | BigInt        | @unique                               | ID user pemilik tenant             |
| name        | String        | @db.VarChar(255)                      | Nama tenant                        |
| npwp        | String?       | @db.VarChar(255)                      | Nomor NPWP (opsional)              |
| npwpImage   | String?       | @db.VarChar(255)                      | Gambar NPWP (opsional)             |
| address     | String        | @db.Text                              | Alamat tenant                      |
| provinceId  | Int           |                                       | ID provinsi                        |
| cityId      | Int           |                                       | ID kota                            |
| districtId  | Int           |                                       | ID kecamatan                       |
| villageId   | Int           |                                       | ID desa                            |
| postalCode  | String        | @db.VarChar(12)                       | Kode pos                           |
| tenantStatus| TenantStatus  | @default(ACTIVE)                      | Status tenant                      |
| createdAt   | DateTime      | @default(now())                       | Tanggal dibuat                     |
| updatedAt   | DateTime      | @updatedAt                            | Tanggal terakhir diupdate          |

- Relasi:
  - user: User (one-to-one, fields: [userId], references: [id])
  - province: Province (many-to-one, fields: [provinceId], references: [id])
  - city: City (many-to-one, fields: [cityId], references: [id])
  - district: District (many-to-one, fields: [districtId], references: [id])
  - village: Village (many-to-one, fields: [villageId], references: [id])
  - stores: Store[] (one-to-many)
  - tenantUser: TenantUser[] (one-to-many)

### Model: Role
- Nama Tabel: roles (mapped dari Role)
- Deskripsi: Tabel ini menyimpan peran (role) user dalam sistem, seperti admin, kasir, dll.
- Isi Tabel:

| Field    | Type    | Constraints/Defaults          | Deskripsi                  |
|----------|---------|-------------------------------|----------------------------|
| id      | Int     | @id @default(autoincrement()) | ID unik role               |
| roleName| String  | @unique @db.VarChar(50)       | Nama role                  |

- Relasi:
  - users: User[] (one-to-many)

### Model: User
- Nama Tabel: users (mapped dari User)
- Deskripsi: Tabel ini menyimpan informasi user sistem, termasuk autentikasi, profil, dan verifikasi.
- Isi Tabel:

| Field          | Type     | Constraints/Defaults                  | Deskripsi                          |
|----------------|----------|---------------------------------------|------------------------------------|
| id             | BigInt   | @id @default(autoincrement())         | ID unik user                       |
| name           | String?  | @db.VarChar(255)                      | Nama user (opsional)               |
| email          | String   | @unique @db.VarChar(255)              | Email user                         |
| password       | String   | @db.Char(60)                          | Password hash                      |
| imageUrl       | String?  | @db.VarChar(255)                      | URL gambar profil (opsional)       |
| phoneNumber    | String   | @db.VarChar(15)                       | Nomor telepon                      |
| isEmailVerified| Boolean  | @default(false)                       | Status verifikasi email            |
| idCard         | String?  | @db.VarChar(255)                      | Nomor KTP (opsional)               |
| idCardImage    | String?  | @db.VarChar(255)                      | Gambar KTP (opsional)              |
| npwp           | String?  | @db.VarChar(255)                      | Nomor NPWP (opsional)              |
| npwpImage      | String?  | @db.VarChar(255)                      | Gambar NPWP (opsional)             |
| googleId       | String?  | @unique                               | ID Google (opsional)               |
| roleId         | Int      |                                       | ID role                            |
| createdAt      | DateTime | @default(now())                       | Tanggal dibuat                     |
| updatedAt      | DateTime | @updatedAt                            | Tanggal terakhir diupdate          |

- Relasi:
  - role: Role (many-to-one, fields: [roleId], references: [id])
  - tokens: Token[] (one-to-many)
  - tenant: Tenant[] (one-to-many)
  - activityUsers: ActivityUser[] (one-to-many)
  - storeBills: StoreBill[] (one-to-many)
  - transactions: Transaction[] (one-to-many)
  - StoreUser: StoreUser[] (one-to-many)
  - TenantUser: TenantUser[] (one-to-many)
  - CashierCash: CashierCash[] (one-to-many)

### Model: Token
- Nama Tabel: tokens (mapped dari Token)
- Deskripsi: Tabel ini menyimpan token autentikasi untuk user, seperti access token, refresh token, dll.
- Isi Tabel:

| Field      | Type      | Constraints/Defaults | Deskripsi                     |
|------------|-----------|----------------------|-------------------------------|
| id        | Int       | @id @default(autoincrement()) | ID unik token                 |
| userId    | BigInt    |                      | ID user                       |
| token     | String    | @db.Text             | Nilai token                   |
| type      | TokenType |                      | Jenis token                   |
| expires   | DateTime  |                      | Tanggal kadaluarsa            |
| blacklisted| Boolean   | @default(false)      | Status blacklist              |
| createdAt | DateTime  | @default(now())      | Tanggal dibuat                |

- Relasi:
  - user: User (many-to-one, fields: [userId], references: [id])

### Model: StoreType
- Nama Tabel: store_types (mapped dari StoreType)
- Deskripsi: Tabel ini menyimpan tipe toko, seperti retail, grosir, dll.
- Isi Tabel:

| Field    | Type    | Constraints/Defaults          | Deskripsi                  |
|----------|---------|-------------------------------|----------------------------|
| id      | Int     | @id @default(autoincrement()) | ID unik tipe toko          |
| name    | String  | @db.VarChar(255)              | Nama tipe toko             |

- Relasi:
  - stores: Store[] (one-to-many)

### Model: Province
- Nama Tabel: provinces (mapped dari Province)
- Deskripsi: Tabel ini menyimpan data provinsi untuk alamat.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults | Deskripsi                     |
|----------|----------|----------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik provinsi              |
| name    | String   | @db.VarChar(255)     | Nama provinsi                 |
| createdAt| DateTime | @default(now())      | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt           | Tanggal terakhir diupdate     |

- Relasi:
  - cities: City[] (one-to-many)
  - stores: Store[] (one-to-many)
  - tenant: Tenant[] (one-to-many)

### Model: City
- Nama Tabel: cities (mapped dari City)
- Deskripsi: Tabel ini menyimpan data kota/kabupaten dalam provinsi.
- Isi Tabel:

| Field     | Type     | Constraints/Defaults | Deskripsi                     |
|-----------|----------|----------------------|-------------------------------|
| id       | Int      | @id @default(autoincrement()) | ID unik kota                  |
| provinceId| Int      |                      | ID provinsi                   |
| name     | String   | @db.VarChar(255)     | Nama kota                     |
| createdAt| DateTime | @default(now())      | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt           | Tanggal terakhir diupdate     |

- Relasi:
  - province: Province (many-to-one, fields: [provinceId], references: [id])
  - districts: District[] (one-to-many)
  - stores: Store[] (one-to-many)
  - tenant: Tenant[] (one-to-many)

### Model: District
- Nama Tabel: districts (mapped dari District)
- Deskripsi: Tabel ini menyimpan data kecamatan dalam kota.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults | Deskripsi                     |
|----------|----------|----------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik kecamatan             |
| cityId  | Int      |                      | ID kota                       |
| name    | String   | @db.VarChar(255)     | Nama kecamatan                |
| createdAt| DateTime | @default(now())      | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt           | Tanggal terakhir diupdate     |

- Relasi:
  - city: City (many-to-one, fields: [cityId], references: [id])
  - villages: Village[] (one-to-many)
  - stores: Store[] (one-to-many)
  - tenant: Tenant[] (one-to-many)

### Model: Village
- Nama Tabel: villages (mapped dari Village)
- Deskripsi: Tabel ini menyimpan data desa/kelurahan dalam kecamatan.
- Isi Tabel:

| Field     | Type     | Constraints/Defaults | Deskripsi                     |
|-----------|----------|----------------------|-------------------------------|
| id       | Int      | @id @default(autoincrement()) | ID unik desa                  |
| districtId| Int      |                      | ID kecamatan                  |
| name     | String   | @db.VarChar(255)     | Nama desa                     |
| createdAt| DateTime | @default(now())      | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt           | Tanggal terakhir diupdate     |

- Relasi:
  - district: District (many-to-one, fields: [districtId], references: [id])
  - stores: Store[] (one-to-many)
  - tenant: Tenant[] (one-to-many)

### Model: Store
- Nama Tabel: stores (mapped dari Store)
- Deskripsi: Tabel ini menyimpan informasi toko, termasuk lokasi, tipe, dan status. Toko terkait dengan tenant.
- Isi Tabel:

| Field            | Type       | Constraints/Defaults                  | Deskripsi                          |
|------------------|------------|---------------------------------------|------------------------------------|
| id              | Int        | @id @default(autoincrement())         | ID unik toko                       |
| storeCode       | String?    | @unique @db.VarChar(255)              | Kode unik toko (opsional)          |
| tenantId        | Int        |                                       | ID tenant                          |
| name            | String     | @db.VarChar(255)                      | Nama toko                          |
| address         | String     | @db.Text                              | Alamat toko                        |
| email           | String     | @db.VarChar(255)                      | Email toko                         |
| image           | String     | @db.VarChar(255)                      | Gambar toko                        |
| provinceId      | Int        |                                       | ID provinsi                        |
| cityId          | Int        |                                       | ID kota                            |
| districtId      | Int        |                                       | ID kecamatan                       |
| villageId       | Int        |                                       | ID desa                            |
| storeTypeId     | Int        |                                       | ID tipe toko                       |
| postalCode      | String     | @db.VarChar(12)                       | Kode pos                           |
| description     | String     | @db.Text                              | Deskripsi toko                     |
| phoneNumber     | String     | @db.VarChar(15)                       | Nomor telepon                      |
| logo            | String?    | @db.VarChar(255)                      | Logo toko (opsional)               |
| permitCertificate| String?   | @db.VarChar(255)                      | Sertifikat izin (opsional)         |
| status          | StoreStatus| @default(ACTIVE)                      | Status toko                        |
| createdAt       | DateTime   | @default(now())                       | Tanggal dibuat                     |
| updatedAt       | DateTime   | @updatedAt                            | Tanggal terakhir diupdate          |

- Relasi:
  - province: Province (many-to-one, fields: [provinceId], references: [id])
  - city: City (many-to-one, fields: [cityId], references: [id])
  - district: District (many-to-one, fields: [districtId], references: [id])
  - village: Village (many-to-one, fields: [villageId], references: [id])
  - storeType: StoreType (many-to-one, fields: [storeTypeId], references: [id])
  - tenant: Tenant (many-to-one, fields: [tenantId], references: [id])
  - customers: Customer[] (one-to-many)
  - storeBills: StoreBill[] (one-to-many)
  - activityUsers: ActivityUser[] (one-to-many)
  - transactions: Transaction[] (one-to-many)
  - StoreUser: StoreUser[] (one-to-many)
  - PurchaseOrder: PurchaseOrder[] (one-to-many)
  - WarehouseStock: WarehouseStock[] (one-to-many)
  - CashierCash: CashierCash[] (one-to-many)
  - StoreSupplier: StoreSupplier[] (one-to-many)
  - OwnedSuppliers: Supplier[] (one-to-many, relation: "SupplierOwner")

### Model: StoreUser
- Nama Tabel: store_users (mapped dari StoreUser)
- Deskripsi: Tabel junction untuk menghubungkan user dengan toko, dengan status keaktifan.
- Isi Tabel:

| Field    | Type           | Constraints/Defaults          | Deskripsi                     |
|----------|----------------|-------------------------------|-------------------------------|
| id      | BigInt         | @id @default(autoincrement()) | ID unik store user            |
| userId  | BigInt         |                               | ID user                       |
| storeId | Int            |                               | ID toko                       |
| status  | StoreUserStatus| @default(ACTIVE)              | Status user di toko           |
| createdAt| DateTime      | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime      | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - user: User? (many-to-one, fields: [userId], references: [id])
  - store: Store? (many-to-one, fields: [storeId], references: [id])
  - Shift: Shift[] (one-to-many)
  - Customer: Customer[] (one-to-many)

### Model: TenantUser
- Nama Tabel: tenant_users (mapped dari TenantUser)
- Deskripsi: Tabel junction untuk menghubungkan user dengan tenant, dengan status keaktifan.
- Isi Tabel:

| Field    | Type            | Constraints/Defaults          | Deskripsi                     |
|----------|-----------------|-------------------------------|-------------------------------|
| id      | BigInt          | @id @default(autoincrement()) | ID unik tenant user           |
| userId  | BigInt          |                               | ID user                       |
| tenantId| Int             |                               | ID tenant                     |
| status  | TenantUserStatus| @default(ACTIVE)              | Status user di tenant         |
| createdAt| DateTime       | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime       | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - tenant: Tenant (many-to-one, fields: [tenantId], references: [id])
  - users: User (many-to-one, fields: [userId], references: [id])

### Model: Shift
- Nama Tabel: shifts (mapped dari Shift)
- Deskripsi: Tabel ini menyimpan jadwal shift karyawan di toko, termasuk status kas.
- Isi Tabel:

| Field      | Type      | Constraints/Defaults          | Deskripsi                     |
|------------|-----------|-------------------------------|-------------------------------|
| id        | Int       | @id @default(autoincrement()) | ID unik shift                 |
| shiftStart| DateTime  |                               | Waktu mulai shift             |
| shiftEnd  | DateTime? |                               | Waktu akhir shift (opsional)  |
| days      | ShiftDay  |                               | Hari shift                    |
| cashStatus| CashStatus| @default(PENDING)             | Status kas shift              |
| userStoreId| BigInt   |                               | ID store user                 |
| createdAt | DateTime  | @default(now())               | Tanggal dibuat                |
| updatedAt | DateTime  | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - storeUser: StoreUser (many-to-one, fields: [userStoreId], references: [id])
  - CashierCash: CashierCash[] (one-to-many)

### Model: CashierCash
- Nama Tabel: cashier_cash (mapped dari CashierCash)
- Deskripsi: Tabel ini menyimpan data kas kasir per shift, termasuk selisih dan catatan.
- Isi Tabel:

| Field         | Type     | Constraints/Defaults          | Deskripsi                          |
|---------------|----------|-------------------------------|------------------------------------|
| id           | Int      | @id @default(autoincrement()) | ID unik kas kasir                  |
| shiftId      | Int      |                               | ID shift                           |
| userId       | BigInt   |                               | ID user                            |
| storeId      | Int      |                               | ID toko                            |
| initialCash  | BigInt   | @default(0)                   | Kas awal shift                     |
| finalCash    | BigInt?  |                               | Kas akhir shift (opsional)         |
| systemCash   | BigInt?  |                               | Kas menurut sistem (opsional)      |
| cashDifference| BigInt? |                               | Selisih kas (opsional)             |
| notes        | String?  | @db.Text                      | Catatan selisih (opsional)         |
| isBalanced   | Boolean? |                               | Status keseimbangan kas (opsional) |
| createdAt    | DateTime | @default(now())               | Tanggal dibuat                     |
| updatedAt    | DateTime | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - shift: Shift (many-to-one, fields: [shiftId], references: [id], onDelete: Cascade)
  - user: User (many-to-one, fields: [userId], references: [id])
  - store: Store (many-to-one, fields: [storeId], references: [id])

### Model: productCategory
- Nama Tabel: product_categories (mapped dari productCategory)
- Deskripsi: Tabel ini menyimpan kategori produk per toko.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik kategori              |
| storeId | Int      |                               | ID toko                       |
| name    | String   | @db.VarChar(255)              | Nama kategori                 |
| createdAt| DateTime | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - products: Product[] (one-to-many)

### Model: Product
- Nama Tabel: products (mapped dari Product)
- Deskripsi: Tabel ini menyimpan informasi produk dasar, termasuk kode, nama, harga, dan status.
- Isi Tabel:

| Field            | Type     | Constraints/Defaults                  | Deskripsi                          |
|------------------|----------|---------------------------------------|------------------------------------|
| id              | BigInt   | @id @default(autoincrement())         | ID unik produk                     |
| productCode     | String   | @unique @db.VarChar(255)              | Kode unik produk                   |
| serialNumber    | Int?     |                                       | Nomor serial (opsional)            |
| productCategoryId| Int     |                                       | ID kategori produk                 |
| name            | String   | @db.VarChar(255)                      | Nama produk                        |
| description     | String   | @db.Text                              | Deskripsi produk                   |
| discount        | Int?     |                                       | Diskon (opsional)                  |
| basePrice       | BigInt   |                                       | Harga dasar                        |
| imageUrl        | String   | @db.VarChar(255)                      | URL gambar produk                  |
| isActive        | Boolean  | @default(true)                        | Status aktif                       |
| createdAt       | DateTime | @default(now())                       | Tanggal dibuat                     |
| updatedAt       | DateTime | @updatedAt                            | Tanggal terakhir diupdate          |

- Relasi:
  - productCategory: productCategory (many-to-one, fields: [productCategoryId], references: [id])
  - variants: ProductVariant[] (one-to-many)
  - TransactionDetail: TransactionDetail[] (one-to-many)
  - PurchaseOrderItem: PurchaseOrderItem[] (one-to-many)
  - WarehouseStock: WarehouseStock[] (one-to-many)
  - SupplierProduct: SupplierProduct[] (one-to-many)

### Model: VariantType
- Nama Tabel: variant_types (mapped dari VariantType)
- Deskripsi: Tabel ini menyimpan tipe varian produk (misal: Color, Size), dengan scope global atau store.
- Isi Tabel:

| Field      | Type        | Constraints/Defaults          | Deskripsi                          |
|------------|-------------|-------------------------------|------------------------------------|
| id        | BigInt      | @id @default(autoincrement()) | ID unik tipe varian                |
| name      | String      | @db.VarChar(255)              | Nama tipe varian                   |
| scope     | VariantScope| @default(GLOBAL)              | Scope varian                       |
| storeId   | Int?        |                               | ID toko (opsional untuk store)     |
| description| String?    | @db.Text                      | Deskripsi (opsional)               |
| isActive  | Boolean     | @default(true)                | Status aktif                       |
| createdAt | DateTime    | @default(now())               | Tanggal dibuat                     |
| updatedAt | DateTime    | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - options: VariantOption[] (one-to-many)

### Model: VariantOption
- Nama Tabel: variant_options (mapped dari VariantOption)
- Deskripsi: Tabel ini menyimpan opsi varian (misal: Merah untuk Color), dengan harga tambahan.
- Isi Tabel:

| Field          | Type         | Constraints/Defaults          | Deskripsi                          |
|----------------|--------------|-------------------------------|------------------------------------|
| id            | BigInt       | @id @default(autoincrement()) | ID unik opsi varian                |
| variantTypeId | BigInt       |                               | ID tipe varian                     |
| name          | String       | @db.VarChar(255)              | Nama opsi                          |
| additionalPrice| BigInt      | @default(0)                   | Harga tambahan                     |
| status        | VariantStatus| @default(ACTIVE)              | Status opsi                        |
| createdAt     | DateTime     | @default(now())               | Tanggal dibuat                     |
| updatedAt     | DateTime     | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - variantType: VariantType (many-to-one, fields: [variantTypeId], references: [id], onDelete: Cascade)
  - productVariantOptions: ProductVariantOption[] (one-to-many)
  - transactionDetails: TransactionDetail[] (one-to-many)

### Model: ProductVariant
- Nama Tabel: product_variants (mapped dari ProductVariant)
- Deskripsi: Tabel ini menyimpan varian aktual untuk setiap produk.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | BigInt   | @id @default(autoincrement()) | ID unik varian produk         |
| productId| BigInt  |                               | ID produk                     |
| isActive| Boolean  | @default(true)                | Status aktif                  |
| createdAt| DateTime| @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime| @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - product: Product (many-to-one, fields: [productId], references: [id], onDelete: Cascade)
  - productVariantOptions: ProductVariantOption[] (one-to-many)
  - transactionDetails: TransactionDetail[] (one-to-many)

### Model: ProductVariantOption
- Nama Tabel: product_variant_options (mapped dari ProductVariantOption)
- Deskripsi: Tabel junction untuk menghubungkan varian produk dengan opsi varian.
- Isi Tabel:

| Field           | Type   | Constraints/Defaults          | Deskripsi                          |
|-----------------|--------|-------------------------------|------------------------------------|
| id             | BigInt | @id @default(autoincrement()) | ID unik junction                   |
| productVariantId| BigInt|                               | ID varian produk                   |
| variantOptionId| BigInt|                               | ID opsi varian                     |

- Relasi:
  - productVariant: ProductVariant (many-to-one, fields: [productVariantId], references: [id], onDelete: Cascade)
  - variantOption: VariantOption (many-to-one, fields: [variantOptionId], references: [id], onDelete: Cascade)

### Model: PaymentMethod
- Nama Tabel: payment_methods (mapped dari PaymentMethod)
- Deskripsi: Tabel ini menyimpan metode pembayaran, termasuk apakah tunai atau tidak.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik metode pembayaran     |
| name    | String   | @db.VarChar(255)              | Nama metode                   |
| image   | String?  | @db.VarChar(255)              | Gambar metode (opsional)      |
| isCash  | IsCash   |                               | Indikator tunai               |
| createdAt| DateTime | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - transactions: Transaction[] (one-to-many)

### Model: Tax
- Nama Tabel: taxes (mapped dari Tax)
- Deskripsi: Tabel ini menyimpan pajak per transaksi.
- Isi Tabel:

| Field        | Type   | Constraints/Defaults     | Deskripsi                     |
|--------------|--------|--------------------------|-------------------------------|
| id          | Int    | @id @default(autoincrement()) | ID unik pajak                 |
| transactionId| String | @db.VarChar(255)         | ID invoice transaksi          |
| name        | String | @db.VarChar(255)         | Nama pajak                    |
| rate        | Int    |                          | Rate pajak                    |

- Relasi:
  - transaction: Transaction (many-to-one, fields: [transactionId], references: [invoiceId])

### Model: Transaction
- Nama Tabel: transactions (mapped dari Transaction)
- Deskripsi: Tabel ini menyimpan transaksi penjualan, termasuk total, pembayaran, dan status.
- Isi Tabel:

| Field          | Type             | Constraints/Defaults                  | Deskripsi                          |
|----------------|------------------|---------------------------------------|------------------------------------|
| id            | BigInt           | @id @default(autoincrement())         | ID unik transaksi                  |
| invoiceId     | String           | @unique @db.VarChar(255)              | ID invoice unik                    |
| customerId    | Int              |                                       | ID pelanggan                       |
| userId        | BigInt           |                                       | ID user (kasir)                    |
| storeId       | Int              |                                       | ID toko                            |
| paymentMethodId| Int             |                                       | ID metode pembayaran               |
| totalAmount   | BigInt           |                                       | Total jumlah                       |
| cash          | BigInt           |                                       | Tunai yang dibayarkan              |
| change        | BigInt           |                                       | Kembalian                          |
| discount      | BigInt           | @default(0)                           | Diskon                             |
| transactionDate| DateTime        |                                       | Tanggal transaksi                  |
| status        | TransactionStatus| @default(PENDING)                     | Status transaksi                   |
| createdAt     | DateTime         | @default(now())                       | Tanggal dibuat                     |
| updatedAt     | DateTime         | @updatedAt                            | Tanggal terakhir diupdate          |

- Relasi:
  - customer: Customer (many-to-one, fields: [customerId], references: [id])
  - user: User (many-to-one, fields: [userId], references: [id])
  - store: Store (many-to-one, fields: [storeId], references: [id])
  - paymentMethod: PaymentMethod (many-to-one, fields: [paymentMethodId], references: [id])
  - tax: Tax[] (one-to-many)
  - details: TransactionDetail[] (one-to-many)

### Model: TransactionDetail
- Nama Tabel: transaction_details (mapped dari TransactionDetail)
- Deskripsi: Tabel ini menyimpan detail item dalam transaksi.
- Isi Tabel:

| Field           | Type     | Constraints/Defaults          | Deskripsi                          |
|-----------------|----------|-------------------------------|------------------------------------|
| id             | Int      | @id @default(autoincrement()) | ID unik detail transaksi           |
| transactionId  | BigInt   |                               | ID transaksi                       |
| productId      | BigInt   |                               | ID produk                          |
| productVariantId| BigInt? |                               | ID varian produk (opsional)        |
| quantity       | Int      |                               | Kuantitas                          |
| unitPrice      | BigInt   |                               | Harga per unit                     |
| subtotal       | BigInt   |                               | Subtotal                           |
| createdAt      | DateTime | @default(now())               | Tanggal dibuat                     |
| updatedAt      | DateTime | @updatedAt                    | Tanggal terakhir diupdate          |
| variantOptionId| BigInt?  |                               | ID opsi varian (opsional)          |

- Relasi:
  - transaction: Transaction (many-to-one, fields: [transactionId], references: [id])
  - product: Product (many-to-one, fields: [productId], references: [id])
  - productVariant: ProductVariant? (many-to-one, fields: [productVariantId], references: [id])
  - variantOption: VariantOption? (many-to-one, fields: [variantOptionId], references: [id])

### Model: ActivityUser
- Nama Tabel: user_activity (mapped dari ActivityUser)
- Deskripsi: Tabel ini mencatat aktivitas user di toko.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik aktivitas             |
| userId  | BigInt   |                               | ID user                       |
| storeId | Int      |                               | ID toko                       |
| action  | String   | @db.VarChar(255)              | Aksi yang dilakukan           |
| actionTime| DateTime|                               | Waktu aksi                    |

- Relasi:
  - user: User (many-to-one, fields: [userId], references: [id])
  - store: Store (many-to-one, fields: [storeId], references: [id])

### Model: StoreBill
- Nama Tabel: store_bills (mapped dari StoreBill)
- Deskripsi: Tabel ini menyimpan tagihan toko, seperti biaya bulanan.
- Isi Tabel:

| Field    | Type           | Constraints/Defaults          | Deskripsi                     |
|----------|----------------|-------------------------------|-------------------------------|
| id      | Int            | @id @default(autoincrement()) | ID unik tagihan               |
| storeId | Int            |                               | ID toko                       |
| name    | String         | @db.VarChar(255)              | Nama tagihan                  |
| status  | StoreBillStatus|                               | Status tagihan                |
| price   | Int            |                               | Harga tagihan                 |
| onDate  | DateTime       |                               | Tanggal tagihan               |
| createdAt| DateTime      | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime      | @updatedAt                    | Tanggal terakhir diupdate     |
| userId  | BigInt?        |                               | ID user (opsional)            |

- Relasi:
  - store: Store (many-to-one, fields: [storeId], references: [id])
  - User: User? (many-to-one, fields: [userId], references: [id])

### Model: Customer
- Nama Tabel: customers (mapped dari Customer)
- Deskripsi: Tabel ini menyimpan informasi pelanggan per toko.
- Isi Tabel:

| Field      | Type           | Constraints/Defaults          | Deskripsi                          |
|------------|----------------|-------------------------------|------------------------------------|
| id        | Int            | @id @default(autoincrement()) | ID unik pelanggan                  |
| storeId   | Int            |                               | ID toko                            |
| storeUserId| BigInt        |                               | ID store user                      |
| name      | String         | @db.VarChar(255)              | Nama pelanggan                     |
| phoneNumber| String?       | @db.VarChar(255)              | Nomor telepon (opsional)           |
| status    | CustomerStatus | @default(ACTIVE)              | Status pelanggan                   |
| createdAt | DateTime       | @default(now())               | Tanggal dibuat                     |
| updatedAt | DateTime       | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - store: Store (many-to-one, fields: [storeId], references: [id])
  - storeUser: StoreUser (many-to-one, fields: [storeUserId], references: [id])
  - Transaction: Transaction[] (one-to-many)

### Model: Supplier
- Nama Tabel: suppliers (mapped dari Supplier)
- Deskripsi: Tabel ini menyimpan informasi supplier, dengan scope global atau store.
- Isi Tabel:

| Field        | Type          | Constraints/Defaults          | Deskripsi                          |
|--------------|---------------|-------------------------------|------------------------------------|
| id          | Int           | @id @default(autoincrement()) | ID unik supplier                   |
| name        | String        | @db.VarChar(255)              | Nama supplier                      |
| scope       | SupplierScope | @default(GLOBAL)              | Scope supplier                     |
| storeId     | Int?          |                               | ID toko pemilik (opsional)         |
| phoneNumber | String        | @db.VarChar(20)               | Nomor telepon                      |
| email       | String?       | @db.VarChar(255)              | Email (opsional)                   |
| address     | String        | @db.Text                      | Alamat                             |
| status      | SupplierStatus| @default(ACTIVE)              | Status supplier                    |
| description | String?       | @db.Text                      | Deskripsi (opsional)               |
| contactPerson| String?      | @db.VarChar(255)              | Nama kontak (opsional)             |
| createdAt   | DateTime      | @default(now())               | Tanggal dibuat                     |
| updatedAt   | DateTime      | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - ownerStore: Store? (many-to-one, fields: [storeId], references: [id])
  - storeSuppliers: StoreSupplier[] (one-to-many)
  - SupplierProduct: SupplierProduct[] (one-to-many)

### Model: StoreSupplier
- Nama Tabel: store_suppliers (mapped dari StoreSupplier)
- Deskripsi: Tabel junction untuk menghubungkan toko dengan supplier.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik junction              |
| storeId | Int      |                               | ID toko                       |
| supplierId| Int    |                               | ID supplier                   |
| isActive| Boolean  | @default(true)                | Status aktif                  |
| createdAt| DateTime | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - store: Store (many-to-one, fields: [storeId], references: [id], onDelete: Cascade)
  - supplier: Supplier (many-to-one, fields: [supplierId], references: [id], onDelete: Cascade)
  - purchaseOrders: PurchaseOrder[] (one-to-many)

### Model: PurchaseOrder
- Nama Tabel: purchase_orders (mapped dari PurchaseOrder)
- Deskripsi: Tabel ini menyimpan pesanan pembelian dari supplier.
- Isi Tabel:

| Field          | Type               | Constraints/Defaults          | Deskripsi                          |
|----------------|--------------------|-------------------------------|------------------------------------|
| id            | Int                | @id @default(autoincrement()) | ID unik pesanan                    |
| storeSupplierId| Int               |                               | ID store supplier                  |
| storeId       | Int                |                               | ID toko                            |
| totalAmount   | Int                | @default(0)                   | Total jumlah                       |
| status        | PurchaseOrderStatus| @default(PENDING)             | Status pesanan                     |
| orderDate     | DateTime           | @default(now())               | Tanggal pesanan                    |
| notes         | String?            | @db.Text                      | Catatan (opsional)                 |
| createdAt     | DateTime           | @default(now())               | Tanggal dibuat                     |
| updatedAt     | DateTime           | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - storeSupplier: StoreSupplier (many-to-one, fields: [storeSupplierId], references: [id])
  - store: Store (many-to-one, fields: [storeId], references: [id])
  - purchaseOrderItems: PurchaseOrderItem[] (one-to-many)
  - WarehouseStock: WarehouseStock[] (one-to-many)

### Model: PurchaseOrderItem
- Nama Tabel: purchase_order_items (mapped dari PurchaseOrderItem)
- Deskripsi: Tabel ini menyimpan item dalam pesanan pembelian.
- Isi Tabel:

| Field          | Type     | Constraints/Defaults          | Deskripsi                          |
|----------------|----------|-------------------------------|------------------------------------|
| id            | Int      | @id @default(autoincrement()) | ID unik item                       |
| purchaseOrderId| Int     |                               | ID pesanan                         |
| productId     | BigInt   |                               | ID produk                          |
| quantity      | Int      |                               | Kuantitas                          |
| capitalPrice  | Int      |                               | Harga modal                        |
| expiredDate   | DateTime?|                               | Tanggal kadaluarsa (opsional)      |
| createdAt     | DateTime | @default(now())               | Tanggal dibuat                     |
| updatedAt     | DateTime | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - purchaseOrder: PurchaseOrder (many-to-one, fields: [purchaseOrderId], references: [id], onDelete: Cascade)
  - product: Product (many-to-one, fields: [productId], references: [id])

### Model: WarehouseStock
- Nama Tabel: warehouse_stocks (mapped dari WarehouseStock)
- Deskripsi: Tabel ini menyimpan stok gudang per produk dan toko.
- Isi Tabel:

| Field          | Type     | Constraints/Defaults          | Deskripsi                          |
|----------------|----------|-------------------------------|------------------------------------|
| id            | Int      | @id @default(autoincrement()) | ID unik stok                       |
| productId     | BigInt   |                               | ID produk                          |
| storeId       | Int      |                               | ID toko                            |
| purchaseOrderId| Int?    |                               | ID pesanan (opsional)              |
| stock         | Int      | @default(0)                   | Jumlah stok                        |
| createdAt     | DateTime | @default(now())               | Tanggal dibuat                     |
| updatedAt     | DateTime | @updatedAt                    | Tanggal terakhir diupdate          |

- Relasi:
  - product: Product (many-to-one, fields: [productId], references: [id])
  - store: Store (many-to-one, fields: [storeId], references: [id])
  - purchaseOrder: PurchaseOrder? (many-to-one, fields: [purchaseOrderId], references: [id])

### Model: SupplierProduct
- Nama Tabel: supplier_products (mapped dari SupplierProduct)
- Deskripsi: Tabel junction untuk menghubungkan supplier dengan produk.
- Isi Tabel:

| Field    | Type     | Constraints/Defaults          | Deskripsi                     |
|----------|----------|-------------------------------|-------------------------------|
| id      | Int      | @id @default(autoincrement()) | ID unik junction              |
| supplierId| Int     |                               | ID supplier                   |
| productId| BigInt  |                               | ID produk                     |
| createdAt| DateTime | @default(now())               | Tanggal dibuat                |
| updatedAt| DateTime | @updatedAt                    | Tanggal terakhir diupdate     |

- Relasi:
  - supplier: Supplier (many-to-one, fields: [supplierId], references: [id], onDelete: Cascade)
  - product: Product (many-to-one, fields: [productId], references: [id], onDelete: Cascade)

# 📅 Day 5 & 6 – Data Cleaning in SQL: Nashville Housing Dataset

## 🎯 Learning Goals
- Membersihkan dan mempersiapkan data untuk analisis menggunakan SQL
- Menerapkan teknik transformasi data: standar format, parsing alamat, hingga handling null & duplikat
- Memahami proses ETL (Extract, Transform, Load) level fundamental

---

## 🧹 Steps Performed

### 1. Standardisasi Format Tanggal
Mengubah `SaleDate` dari format datetime ke `DATE`:
```sql
#### 1. Standardisasi Format Tanggal
Mengubah `SaleDate` dari format datetime ke `DATE`:

ALTER TABLE NashvilleHousing
ADD SaleDateConverted DATE;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(DATE, SaleDate);
```

### 2. Mengisi Data Alamat yang Kosong
Menggunakan teknik self join berdasarkan ParcelID untuk mengisi PropertyAddress yang null:
```sql
UPDATE a
SET a.PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
  ON a.ParcelID = b.ParcelID AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL;
```

### 3. Memisahkan Alamat Menjadi Kolom Sendiri
Split PropertyAddress menjadi Address & City:
```sql
ALTER TABLE NashvilleHousing ADD PropertySplitAddress NVARCHAR(255);
UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1);

ALTER TABLE NashvilleHousing ADD PropertySplitCity NVARCHAR(255);
UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress));
```
Split OwnerAddress jadi Address, City, dan State menggunakan PARSENAME():
```sql
ALTER TABLE NashvilleHousing ADD OwnerSplitAddress NVARCHAR(255);
UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3);
-- City & State juga ditambahkan dengan cara serupa
```

### 4. Normalisasi Field SoldAsVacant
Mengubah value Y/N menjadi Yes/No:
```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
    WHEN SoldAsVacant = 'Y' THEN 'Yes'
    WHEN SoldAsVacant = 'N' THEN 'No'
    ELSE SoldAsVacant
END;
```

### 5. Menghapus Duplikat
Menggunakan ROW_NUMBER() untuk mencari dan menyaring duplikat:
```sql
WITH RowNumCTE AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference
        ORDER BY UniqueID) row_num
    FROM NashvilleHousing
)
DELETE FROM RowNumCTE WHERE row_num > 1;
```

### 6. Menghapus Kolom Tidak Terpakai
Untuk membersihkan struktur tabel dari kolom yang tidak relevan:
```sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
```

---

## 🧠 Key Takeaways
- Teknik JOIN, CASE, SUBSTRING, dan PARSENAME() sangat powerful dalam data transformation
- Menggunakan CTE (WITH) bikin query jadi lebih readable
- Data cleaning adalah bagian penting sebelum analisis dan visualisasi

---

## 🛠️ Tools Used
- Microsoft SQL Server
- SSMS (SQL Server Management Studio)
- Dataset: Nashville Housing (CSV file)

---

## 📌 Next Steps
- Export data bersih ke file baru atau database untuk visualisasi lebih lanjut
- Implementasikan cleaning ini di dataset real lain seperti ecommerce, sales, dll
- Pelajari advanced SQL: CTE rekursif, window functions lanjutan, indexing


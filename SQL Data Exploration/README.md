# 📊 Day 1 – SQL Data Exploration (COVID Dataset)

🧠 **Modul/Referensi:** Project SQL – COVID 19 Data Exploration (Inspired by Alex The Analyst)

---

## 🎯 Tujuan Belajar
Mulai eksplorasi data dengan SQL dari yang paling dasar sampai membuat data siap visualisasi. Fokus utama:
- Query dasar (`SELECT`, `WHERE`, `ORDER BY`)
- Perbandingan `total_cases` vs `total_deaths`
- Analisis tingkat infeksi per populasi
- Menggunakan `GROUP BY` dan `MAX()` untuk insight agregat
- CTE (Common Table Expressions) untuk logika query yang lebih bersih
- Temporary Table dan View sebagai persiapan ke tahap visualisasi

---

## 🔍 Insight Utama
- Negara dengan tingkat infeksi tertinggi bukan berarti tertinggi juga dalam kematian.
- Kita bisa ukur death rate cukup mudah pakai `(total_deaths / total_cases) * 100`.
- Menggabungkan dataset kematian & vaksinasi kasih insight soal progress vaksin di tiap negara.
- CTE dan Temp Table membantu buat data transformasi sebelum visualisasi.

---

## 🛠️ Tools Digunakan
- **SQL Server Management Studio (SSMS)**
- **Database:** `PortfolioProject..CovidDeaths` & `PortfolioProject..CovidVaccinations`

---

## 🧪 Highlight Query
```sql
-- Melihat kemungkinan kematian akibat Covid
SELECT location, date, total_cases, total_deaths, 
       (total_deaths/total_cases)*100 AS death_percentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2;

-- CTE: Rolling vaksinasi per negara
WITH PopvsVac AS (
  SELECT dea.location, dea.date, dea.population, vac.new_vaccinations,
         SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.date) AS RollingPeopleVaccinated
  FROM PortfolioProject..CovidDeaths dea
  JOIN PortfolioProject..CovidVaccinations vac
    ON dea.location = vac.location AND dea.date = vac.date
  WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM PopvsVac;

# Requirements Document

## Introduction

Fitur chart "Akumulasi Waktu Aktif Kerja Online Agen" pada dashboard analytics menampilkan data yang tidak akurat dan tidak konsisten. Chart ini menunjukkan nilai waktu online yang sangat tinggi (97-98 jam) yang tidak realistis untuk data harian atau mingguan, dan beberapa agen menampilkan nilai 0 padahal seharusnya memiliki aktivitas.

## Glossary

- **Dashboard**: Halaman utama yang menampilkan ringkasan performa CS
- **Online Hours Map**: Objek JavaScript yang menyimpan data waktu online per agen (agentId sebagai key, jam sebagai value)
- **Chart**: Visualisasi grafik menggunakan Chart.js
- **Agent/CS**: Customer Service yang menangani chat pelanggan
- **Working Hours**: Waktu aktif kerja agen saat online di sistem

## Requirements

### Requirement 1

**User Story:** Sebagai pengguna dashboard, saya ingin melihat akumulasi waktu kerja online yang akurat untuk setiap agen, sehingga saya dapat mengevaluasi kehadiran dan keaktifan mereka secara tepat.

#### Acceptance Criteria

1. WHEN the dashboard loads THEN the system SHALL display online hours data that reflects realistic working time values
2. WHEN online hours data is zero for an agent THEN the system SHALL display "-" instead of showing a bar with zero value
3. WHEN all agents have zero online hours THEN the system SHALL display a message indicating no data is available
4. THE system SHALL format online hours values with one decimal place precision
5. THE system SHALL ensure online hours values do not exceed 24 hours per day for daily reports

### Requirement 2

**User Story:** Sebagai administrator, saya ingin data online hours dapat diperbarui melalui file upload atau API, sehingga data selalu sinkron dengan sistem backend.

#### Acceptance Criteria

1. WHEN an Excel file containing online hours data is uploaded THEN the system SHALL parse and update the onlineHoursMap
2. WHEN the Excel file does not contain online hours column THEN the system SHALL use default zero values for all agents
3. WHEN the date filter changes THEN the system SHALL recalculate and display the filtered online hours data
4. THE system SHALL validate that online hours values are non-negative numbers
5. THE system SHALL aggregate online hours data by agent when multiple date entries exist

### Requirement 3

**User Story:** Sebagai developer, saya ingin struktur data online hours konsisten dengan data lainnya, sehingga lebih mudah untuk maintenance dan debugging.

#### Acceptance Criteria

1. THE system SHALL store online hours data in the same structure as other metrics (part of rawData array)
2. WHEN aggregating data by agent THEN the system SHALL include online hours in the aggregation object
3. THE system SHALL use the same filtering logic for online hours as for other metrics
4. THE system SHALL maintain data integrity between online hours and other agent metrics
5. THE system SHALL provide clear variable naming that indicates the purpose of online hours data

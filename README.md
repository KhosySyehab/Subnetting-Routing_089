# 🖧 Tugas Komunikasi Data & Jaringan Komputer
**Semester Gasal 2025/2026**  
**Topik:** Subnetting (VLSM & CIDR)  
**Nama:** Muhammad Khosyi Syehab  
**NRP:** 5027241089  
**Base Network:** `10.129.0.0` (karena 089 mod 256 = 129)

---

## Deskripsi Umum

Yayasan Pendidikan ARA akan membangun jaringan untuk beberapa unit kerja yang tersebar di **Kantor Pusat** dan **Kantor Cabang (Bidang Pengawas Sekolah)**.  
Setiap unit memiliki jumlah host berbeda, sehingga diperlukan **perhitungan subnetting dengan VLSM dan CIDR** agar alokasi IP efisien dan tidak boros.

---

## Struktur Jaringan

### **Kantor Pusat**
| Ruang | Jumlah Host Aktif |
| :--- | ---: |
| Sekretariat | 380 |
| Bidang Kurikulum | 220 |
| Bidang Guru & Tendik | 95 |
| Bidang Sarana & Prasarana | 45 |
| Bidang Pengawas Sekolah (HQ) | 18 |
| Server & Admin | 6 |

### **Kantor Cabang**
| Ruang | Jumlah Host Aktif |
| :--- | ---: |
| Bidang Pengawas Sekolah (Branch) | 18 |

---

## Hasil Perhitungan Subnetting (VLSM)

| No | Subnet | Network | Prefix | Mask | Range Host | Broadcast | Gateway |
|:-:|:-|:-|:-:|:-|:-|:-|:-|
| 1 | Sekretariat | 10.129.0.0 | /23 | 255.255.254.0 | 10.129.0.1 – 10.129.1.254 | 10.129.1.255 | 10.129.0.1 |
| 2 | Kurikulum | 10.129.2.0 | /24 | 255.255.255.0 | 10.129.2.1 – 10.129.2.254 | 10.129.2.255 | 10.129.2.1 |
| 3 | Guru & Tendik | 10.129.3.0 | /25 | 255.255.255.128 | 10.129.3.1 – 10.129.3.126 | 10.129.3.127 | 10.129.3.1 |
| 4 | Sarana Prasarana | 10.129.3.128 | /26 | 255.255.255.192 | 10.129.3.129 – 10.129.3.190 | 10.129.3.191 | 10.129.3.129 |
| 5 | Pengawas Sekolah (HQ) | 10.129.3.192 | /27 | 255.255.255.224 | 10.129.3.193 – 10.129.3.222 | 10.129.3.223 | 10.129.3.193 |
| 6 | Pengawas Sekolah (Branch) | 10.129.3.224 | /27 | 255.255.255.224 | 10.129.3.225 – 10.129.3.254 | 10.129.3.255 | 10.129.3.225 |
| 7 | Server & Admin | 10.129.4.0 | /29 | 255.255.255.248 | 10.129.4.1 – 10.129.4.6 | 10.129.4.7 | 10.129.4.1 |
| 8 | Link Core–Branch | 10.129.4.8 | /30 | 255.255.255.252 | 10.129.4.9 – 10.129.4.10 | 10.129.4.11 | 10.129.4.9 |

---

## CIDR (Supernetting Result)

Seluruh jaringan hasil VLSM dapat diagregasi menjadi blok besar berikut:

| Aggregated Network | Jumlah IP | Prefix | Host Range | Keterangan |
|:-|:-|:-:|:-|:-|
| 10.129.0.0 | 1024 | /22 | 10.129.0.0 – 10.129.3.255 | Gabungan Sekretariat–Pengawas |
| 10.129.4.0 | 16 | /28 | 10.129.4.0 – 10.129.4.15 | Gabungan Server + Link P2P |


---

## Topologi Jaringan

**Perangkat yang digunakan (Cisco Packet Tracer):**

| Lokasi | Perangkat | Jumlah |
|:--|:--|:--:|
| Router Pusat | Cisco 2911 | 1 |
| Router Cabang | Cisco 1911 | 1 |
| Switch | Cisco 2960-24TT | 7 |
| PC | PC-PT | 14 |
| Kabel | Straight-Through, Serial DCE/DTE | - |

Topologi (ilustrasi di CPT):  
📸 _Lihat screenshot pada file Excel (Sheet “Topologi”)_

---

## ⚙️ Konfigurasi IP Router (contoh)

### Router_Pusat (Cisco 2911)
```
enable
conf t
interface g0/0
ip address 10.129.0.1 255.255.254.0
no shutdown

interface g0/1
ip address 10.129.2.1 255.255.255.0
no shutdown

interface g0/2
ip address 10.129.3.1 255.255.255.128
no shutdown

interface g0/3
ip address 10.129.3.129 255.255.255.192
no shutdown

interface g1/0
ip address 10.129.3.193 255.255.255.224
no shutdown

interface g1/1
ip address 10.129.4.1 255.255.255.248
no shutdown

interface s0/0/0
ip address 10.129.4.9 255.255.255.252
clock rate 64000
no shutdown
exit

ip route 10.129.3.224 255.255.255.224 10.129.4.10
```

### Router_Cabang (Cisco 1911)
```
enable
conf t
interface s0/0/0
ip address 10.129.4.10 255.255.255.252
no shutdown

interface g0/0
ip address 10.129.3.225 255.255.255.224
no shutdown

ip route 0.0.0.0 0.0.0.0 10.129.4.9
exit
```

---

## Pengujian (Testing)

| Tujuan | Cara Uji | Hasil |
|:--|:--|:--|
| PC di Sekretariat ping ke Gateway | `ping 10.129.0.1` | ✅ Reply |
| PC Cabang ping ke Server | `ping 10.129.4.2` | ✅ Reply |
| PC Kurikulum ke PC Sarpras | `ping <IP PC lain>` | ✅ Reply |
| Router ke Router (S0/0/0) | `ping 10.129.4.10` | ✅ Reply |

---

## Output yang Dikumpulkan

| File | Deskripsi |
|:--|:--|
| `Tugas1_Jarkom_Khosyi_089.pkt` | File topologi Cisco Packet Tracer |
| `VLSM_CIDR.xlsx` | Tabel subnetting (VLSM, CIDR, supernetting) |
| `link_github.txt` | Berisi URL repository GitHub ini |

---

## Author

- **Nama:** Muhammad Khosyi Syehab  
- **NRP:** 5027241089  
- **Mata Kuliah:** Komunikasi Data & Jaringan Komputer  
- **Semester:** Gasal 2025/2026

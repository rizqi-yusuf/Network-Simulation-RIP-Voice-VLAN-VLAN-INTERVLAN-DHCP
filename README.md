<div align="center">
  <h1>🌐 Simulasi Jaringan Enterprise Lengkap: 4 Router, 4 Switch, VoIP Multi-Site, Wireless, VLAN & DHCP</h1>

  <p>
    <b>Blueprint Lengkap: Konfigurasi CLI Penuh untuk 4 Router, 4 Switch, Dynamic Routing (RIPv2), DHCP Pool, dan Layanan Telepon Multi-Site.</b>
  </p>

  [🇬🇧 English](README.md) | **🇮🇩 Bahasa Indonesia**
</div>

<br />

## 📖 Gambaran Proyek
Repositori ini menyediakan dokumentasi dan panduan konfigurasi yang lengkap, langkah demi langkah, untuk topologi jaringan enterprise kompleks dengan 4 router dan 4 switch yang dibangun di Cisco Packet Tracer. Setiap perintah konfigurasi untuk seluruh perangkat jaringan dibuka sepenuhnya di sini sebagai bahan referensi dan pembelajaran yang komprehensif.

---

## 🗺️ Arsitektur & Topologi Jaringan

Topologi ini tersusun dari beberapa zona yang terhubung lewat link WAN serial dan jalur trunk/access berkecepatan tinggi. Jaringannya menangani data yang tersegmentasi, mobilitas wireless, server farm terpusat, dan endpoint VoIP multi-site.

<div align="center">

<img width="886" height="366" alt="Topologi Jaringan" src="https://github.com/user-attachments/assets/c2c3a93e-fb82-4179-a5fc-29e6c645597e" />

</div>

---

## 📋 Tabel Lengkap Subnet, VLAN & Ekstensi VoIP

| Zona / Site | Perangkat | VLAN ID | Subnet / Network | Ekstensi VoIP | Alokasi & Tujuan |
| :--- | :--- | :---: | :--- | :---: | :--- |
| **Site 1 (Kiri Atas)** | Switch 1 & Router 1 | VLAN 10 | `192.70.30.0/24` | **101 – 102** | Access Switch & IP Phone (Kiri Atas) |
| **Site 2 (Kanan Atas)** | Switch 2 & Router 2 | VLAN 20 | `192.80.30.0/24` | **103 – 104** | Access Switch & IP Phone (Kanan Atas) |
| **Site 3 (Kiri Bawah)** | Switch 3 & Router 3 | VLAN 30 | `192.30.30.0/24` | **105 – 106** | Access Switch & IP Phone (Kiri Bawah) |
| **Site 4 (Site Server)** | Switch 4 & Router 4 | VLAN 60 | `192.60.30.0/24` | N/A | Server Farm Web & DNS terpusat |
| **LAN Data** | Workstation | VLAN 10 | `192.10.30.0/24` | N/A | Klien workstation statis & dinamis |
| **LAN Wireless** | Access Point | VLAN 20 | `192.20.30.0/24` | N/A | DHCP pool untuk perangkat mobile |
| **Backbone WAN** | Interface Serial | N/A | Beberapa `/30` | N/A | Link serial point-to-point (`Se0/3/0`, `Se0/3/1`) |

---

## 📝 Konfigurasi Perangkat Lengkap (4 Switch & 4 Router)

---

### BAGIAN A: KONFIGURASI SWITCH (Switch 1 sampai Switch 4)

#### 1. Switch 1 (Site Kiri Atas - Mendukung Ekstensi 101-102)
```text
enable
configure terminal
hostname Switch1

vlan 10
 name VOICE_TOPLEFT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 10
 no shutdown
```

#### 2. Switch 2 (Site Kanan Atas - Mendukung Ekstensi 103-104)
```text
enable
configure terminal
hostname Switch2

vlan 20
 name VOICE_TOPRIGHT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
```

#### 3. Switch 3 (Site Kiri Bawah - Mendukung Ekstensi 105-106)
```text
enable
configure terminal
hostname Switch3

vlan 30
 name VOICE_BOTLEFT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 30
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 no shutdown
```

#### 4. Switch 4 (Site Server & Data)
```text
enable
configure terminal
hostname Switch4

vlan 10
 name DATA_LAN
vlan 60
 name SERVER

interface GigabitEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 60
 no shutdown
```

---

### BAGIAN B: KONFIGURASI ROUTER (Router 1 sampai Router 4)

#### 1. Router 1 (Gateway Kiri Atas - VoIP 101-102)
```text
enable
configure terminal
hostname Router1

! --- Link WAN Serial ---
interface Serial0/3/0
 ip address 192.66.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Sub-interface Inter-VLAN & DHCP ---
interface FastEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.70.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R1
 network 192.70.30.0 255.255.255.0
 default-router 192.70.30.1
 option 150 ip 192.70.30.1

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.70.30.1 port 2000

ephone-dn 1
 number 101
ephone-dn 2
 number 102

telephony-service
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.70.30.0
 network 192.66.30.0
 no auto-summary
```

#### 2. Router 2 (Gateway Kanan Atas - VoIP 103-104)
```text
enable
configure terminal
hostname Router2

! --- Link WAN Serial ---
interface Serial0/3/0
 ip address 192.66.30.2 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 192.46.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Sub-interface Inter-VLAN & DHCP ---
interface FastEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.80.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R2
 network 192.80.30.0 255.255.255.0
 default-router 192.80.30.1
 option 150 ip 192.80.30.1

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.80.30.1 port 2000

ephone-dn 1
 number 103
ephone-dn 2
 number 104

telephony-service
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.80.30.0
 network 192.66.30.0
 network 192.46.30.0
 no auto-summary
```

#### 3. Router 3 (Gateway Kiri Bawah - VoIP 105-106 & DHCP Wireless)
```text
enable
configure terminal
hostname Router3

! --- Link WAN Serial ---
interface Serial0/3/1
 ip address 192.46.30.2 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 192.56.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Sub-interface, LAN Data & DHCP Wireless ---
interface FastEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.30.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R3
 network 192.30.30.0 255.255.255.0
 default-router 192.30.30.1
 option 150 ip 192.30.30.1

ip dhcp pool WIRELESS_POOL_R3
 network 192.20.30.0 255.255.255.0
 default-router 192.20.30.1
 dns-server 192.60.30.2

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.30.30.1 port 2000

ephone-dn 1
 number 105
ephone-dn 2
 number 106

telephony-service
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.30.30.0
 network 192.20.30.0
 network 192.46.30.0
 network 192.56.30.0
 no auto-summary
```

#### 4. Router 4 (Gateway Site Server)
```text
enable
configure terminal
hostname Router4

! --- Link WAN Serial & LAN Server ---
interface Serial0/3/1
 ip address 192.56.30.2 255.255.255.252
 no shutdown

interface FastEthernet 0/0
 ip address 192.60.30.1 255.255.255.0
 no shutdown

ip dhcp pool SERVER_DATA_POOL
 network 192.10.30.0 255.255.255.0
 default-router 192.10.30.1
 dns-server 192.60.30.2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.56.30.0
 network 192.60.30.0
 network 192.10.30.0
 no auto-summary
```

---

## 🧪 Prosedur Verifikasi & Pengujian

* **Validasi Port Switch:** Jalankan `show vlan brief` dan `show interfaces trunk` di keempat switch untuk memastikan enkapsulasi trunk dan penempatan port access sudah benar.
* **Verifikasi Tabel Routing:** Jalankan `show ip route` di keempat router untuk memastikan sinkronisasi jalur lengkap lewat RIPv2.
* **Uji Panggilan VoIP:** Hubungi ekstensi **101** dari ekstensi **103** atau **105** untuk menguji routing panggilan multi-site lintas batas router yang berbeda.

## 👤 Author

* **Nama:** Rizqi Maulana Yusuf
* **GitHub:** [@rizqi-yusuf](https://github.com/rizqi-yusuf)

<div align="center">

⭐ **Kalau ini membantu kamu, jangan lupa kasih bintang di repo ini!** ⭐

</div>

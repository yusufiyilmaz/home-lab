# Kurulum Notları

## Sanal makineler

| Makine | RAM | Disk | Ağ |
|---|---|---|---|
| Ubuntu Server 24.04 | 4 GB | 25 GB | NAT Network |
| Kali Linux | 4 GB | — | NAT Network |

## Ağ

VirtualBox'ta **NAT Network** (`10.0.2.0/24`) oluşturuldu. Bu ağdaki makineler:
- Birbirini görebilir (saldırı senaryoları için gerekli).
- Kurulum için internete çıkabilir.
- Ev ağındaki diğer cihazlardan erişilemez (izolasyon).

## Ubuntu hedef

- Kurulumda **OpenSSH server** seçildi.
- Dil **English** bırakıldı; log mesajlarının tutarlı (İngilizce) olması için önemli.
- Saat dilimi düzeltildi:
  ```bash
  sudo timedatectl set-timezone Europe/Istanbul
  ```

## Host'tan yönetim

Host (Windows) makineden Ubuntu'ya kolay erişim için VirtualBox NAT Network'e bir port yönlendirme eklendi: host'un `2222` portu → Ubuntu'nun `22` portu. Yalnızca host'un kendisinden (`127.0.0.1`) erişilebilir.

```bash
ssh -p 2222 yusuf@localhost
```

Log analiz için host'a çekilir:

```bash
scp -P 2222 yusuf@localhost:/var/log/auth.log lab-auth.log
```

## Snapshot

Her senaryodan önce temiz bir snapshot alınır, senaryodan sonra o ana dönülür. Böylece lab her zaman bilinen temiz bir durumdan başlar.

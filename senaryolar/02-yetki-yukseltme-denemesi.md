# Senaryo 2: Başarısız Yetki Yükseltme Denemesi ve Tespiti

**Amaç:** Sınırlı yetkili bir hesaba erişen bir saldırganın yönetici (root) yetkisi almaya çalışmasını üretmek ve bu denemenin `auth.log`'da nasıl göründüğünü, nasıl tespit edildiğini belgelemek.

**Ön koşul:** [Senaryo 1](01-ssh-brute-force.md) ile saldırgan `testuser` hesabına erişti. Bu hesabın sudo (yönetici) yetkisi yok.

**İlgili MITRE ATT&CK tekniği:**
- T1548.003 — Abuse Elevation Control Mechanism: Sudo and Sudo Caching
- (hedeflenen) T1003.008 — OS Credential Dumping: /etc/passwd and /etc/shadow

## Saldırı

Saldırgan, `testuser` olarak giriş yaptıktan sonra parola hash'lerinin tutulduğu `/etc/shadow` dosyasını yönetici yetkisiyle okumaya çalıştı:

```bash
sudo cat /etc/shadow
```

`testuser` sudoers listesinde olmadığı için işlem reddedildi. Yani saldırı başarısız oldu; amacımız bu başarısız denemenin bıraktığı izi incelemek.

## Logda görünümü

```text
sudo: testuser : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/testuser ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
```

Bu tek satır üç önemli bilgi taşıyor:
- **Kim:** `testuser`
- **Ne yapmaya çalıştı:** root olarak `/etc/shadow` okumak
- **Sonuç:** `user NOT in sudoers` — reddedildi

## Tespit

```bash
python analyzer.py lab-auth.log
```

Çıktı (ilgili kısım):

```text
== Şüpheli / reddedilen sudo ===================================
  2026-10-16T15:55:33  testuser REDDEDİLDİ (user NOT in sudoers): /usr/bin/cat /etc/shadow

== MITRE ATT&CK (6 teknik) =====================================
  ...
  T1548.003  Abuse Elevation Control Mechanism: Sudo and Sudo Caching  [Privilege Escalation] ×1
```

Araç, reddedilen `sudo` denemesini yakaladı ve **T1548.003** tekniğine eşledi.

## Neden "başarısız" tespiti de önemli?

Saldırı başarısız olsa bile bu iz değerlidir:
- Normal bir kullanıcı `/etc/shadow` okumaya çalışmaz. Bu satır tek başına bir **uyarı işareti**dir.
- Bir hesabın yetki yükseltmeye çalışması, o hesabın ele geçirilmiş olabileceğini gösterir.
- Bu yüzden araç, reddedilen denemeyi "çalışmış komut" gibi değil, ayrı bir **deneme** (T1548.003) olarak işaretler; gerçekte çalışmadığı için `/etc/shadow` okuma (T1003.008) tekniğine eşlenmez.

## Senaryo 1 ile birlikte: tam saldırı zinciri

İki senaryo birlikte okunduğunda saldırının aşamaları ortaya çıkıyor:

1. **Erişim:** Brute force ile `testuser` hesabına giriş (Senaryo 1)
2. **Yetki yükseltme denemesi:** `/etc/shadow` okuma girişimi, reddedildi (Senaryo 2)

Araç her iki aşamayı da tek bir `auth.log` dosyasından, MITRE ATT&CK'e eşleyerek raporladı.

## Temizlik

Senaryo sonrası hedef makine temiz kurulum snapshot'ına döndürülür.

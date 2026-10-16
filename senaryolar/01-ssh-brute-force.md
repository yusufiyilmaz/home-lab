# Senaryo 1: SSH Brute Force ve Tespiti

**Amaç:** Bir saldırganın SSH üzerinden parola denemesi yapıp hesaba eriştiği durumu üretmek ve bu izlerin `auth.log`'da nasıl göründüğünü, nasıl tespit edildiğini belgelemek.

**Ortam:** Kali (`10.0.2.4`, saldırgan) → Ubuntu Server (`10.0.2.5`, hedef). İzole NAT Network.

**İlgili MITRE ATT&CK teknikleri:**
- T1110.001 — Brute Force: Password Guessing
- T1078.003 — Valid Accounts: Local Accounts
- T1021.004 — Remote Services: SSH

## Hazırlık

Hedef Ubuntu'da, senaryo için gerçek hesaplara dokunmadan ayrı bir test hesabı açıldı:

```bash
sudo adduser testuser
```

## Saldırı

Kali'den hedefe, test hesabına karşı art arda başarısız parola denemeleri, ardından doğru parolayla bir giriş yapıldı. Bu, zayıf parolalı bir hesaba yönelik tipik bir parola deneme saldırısını temsil eder.

Amaç bir sistemi ele geçirmek değil, bu davranışın logda bıraktığı izi incelemektir.

## Logda görünümü

Hedefteki `/var/log/auth.log` dosyasında oluşan satırlar (kısaltılmış):

```text
sshd[1253]: Failed password for testuser from 10.0.2.4 port 43726 ssh2
sshd[1253]: Failed password for testuser from 10.0.2.4 port 43726 ssh2
sshd[1263]: Failed password for testuser from 10.0.2.4 port 51372 ssh2
sshd[1263]: Accepted password for testuser from 10.0.2.4 port 51372 ssh2
```

Desen: aynı kaynaktan (`10.0.2.4`) kısa sürede çok sayıda başarısız deneme, ardından başarılı bir giriş.

## Tespit

Log, host makineye çekilip [auth-log-analyzer](https://github.com/yusufiyilmaz/auth-log-analyzer) ile analiz edildi:

```bash
python analyzer.py lab-auth.log
```

Çıktı (ilgili kısım):

```text
== !! OLASI ELE GEÇİRME — başarısız denemelerden sonra başarılı giriş
  2026-10-16T15:19:16  testuser@10.0.2.4  (password), öncesinde 5 hata [BRUTE-FORCE IP]

== Brute-force tespitleri (1) ==================================
  10.0.2.4   toplam 5 | pencerede tepe 5 | 2026-10-16T15:18:28 → 2026-10-16T15:19:00
      denenen: testuser
```

Araç iki şeyi yakaladı:
1. **Brute force:** `10.0.2.4`'ten 5 başarısız deneme, 10 dakikalık pencere içinde.
2. **Olası ele geçirme:** Bu denemelerden sonra gelen başarılı giriş.

## Çıkarılan ders

İlk analizde meşru bir giriş de (parolanın bir kez yanlış yazıldığı) yanlışlıkla "olası ele geçirme" olarak işaretlendi. Bu bir **yanlış pozitif** (false positive) örneğiydi.

Bunun üzerine araca bir eşik eklendi: bir girişin "olası ele geçirme" sayılması için öncesinde en az 3 başarısız deneme aranıyor ve yalnızca giriş öncesi pencere içindeki hatalar sayılıyor. Böylece tek bir yanlış parola alarm üretmiyor, gerçek saldırı ise hâlâ yakalanıyor.

Bu, SOC çalışmasının temel bir parçasıdır: tespit kurallarını yanlış pozitifleri azaltacak şekilde ayarlamak.

## Temizlik

Senaryo sonrası hedef makine temiz kurulum snapshot'ına döndürülür ve test hesabı ortadan kalkar.

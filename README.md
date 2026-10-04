# Linux File System Monitoring and Logging Service

Linux'ta bir klasörü izleyip içindeki dosya değişikliklerini (oluşturma, değiştirme, silme, taşıma) bir log dosyasına yazan küçük bir Python servisi. systemd ile arka planda sürekli çalışacak şekilde kuruluyor.

## Nasıl çalışıyor

`file_watcher.py`, [watchdog](https://github.com/gorakhargosh/watchdog) kütüphanesinin `Observer` sınıfıyla izlenen klasörü alt klasörleriyle birlikte dinliyor. Klasör olaylarını atlıyor, her dosya olayını bir satırlık JSON olarak log dosyasına ekliyor:

```json
{"event_type": "modified", "src_path": "/home/ubuntu/bsm/test/notlar.txt", "timestamp": "2024-12-20 14:03:51"}
```

Aynı satır standart çıktıya da yazıldığı için servis olarak çalışırken `journalctl` ile de görülebiliyor.

## Kurulum

İzlenen klasör ve log dosyası `file_watcher.py`'nin başındaki iki sabitte tanımlı:

```python
LOG_FILE = "/home/ubuntu/bsm/logs/changes.json"
WATCH_DIR = "/home/ubuntu/bsm/test"
```

```bash
pip install watchdog
mkdir -p /home/ubuntu/bsm/logs /home/ubuntu/bsm/test
python3 file_watcher.py
```

Servis olarak kurmak için `file_watcher.service` içindeki `User`, `Group` ve `ExecStart` yolunu kendi sisteminize göre düzenleyin. Servisin kullanıcısının log klasörüne yazma izni olmalı.

```bash
sudo cp file_watcher.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now file_watcher
journalctl -u file_watcher -f
```

`Restart=always` sayesinde servis çökerse systemd onu yeniden başlatıyor.

## Eksikler

- Yollar kodun içinde sabit, yapılandırma dosyası ya da komut satırı parametresi yok.
- Taşıma olaylarında sadece kaynak yol yazılıyor, hedef yol yazılmıyor.
- Log dosyası döndürülmüyor, zamanla büyüyor.

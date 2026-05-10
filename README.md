# MCD-123 Android TV Stick — Setup e Guia Rápido

## Resumo (Português)
Guia de configuração e otimização para o stick Android MCD-123 (Android 13.1). Inclui procedimentos: conexão ADB, forçar banda 2.4GHz, persistência de BSSID, desabilitar bloatware, e diagnósticos úteis.

### Dispositivo
- Modelo: MCD-123
- Android: 13.1
- Recomendação: preferir 2.4GHz para estabilidade de rede

## 1. Conexão ADB via USB
- Verificar ADB: `adb version`
- Listar dispositivos: `adb devices` (aceitar autorização no stick)

## 2. Wi‑Fi — Forçar banda 2.4GHz (solução)
- Identificar BSSID 2.4GHz e 5GHz do roteador
- Exportar/editar `WifiConfigStore.xml` via adb para fixar BSSID 2.4GHz
- Script de boot (fixbssid) disponível para reaplicar no boot
- Observação: solução definitiva—desativar 5GHz no roteador se possível

## 3. Remoção/Desabilitar Bloatware
- Comandos `pm disable-user` listados para aplicações de fábrica
- Proceder com cautela: desabilitar um pacote por vez e testar reboot

## 4. Syslog e Logs (notebook openSUSE)
- Habilitar syslog-ng: `sudo systemctl enable --now syslog-ng`
- Logs em `/var/log/messages`

## 5. Hardware e Firmware
- SoC: Allwinner H616 (sun50iw9p1)
- Firmware: build não oficial — atualizar envolve risco (bootloader bloqueado)

## Diagnóstico útil
Comandos úteis como `adb shell ping`, `dumpsys wifi`, `pm list packages -d`, etc.

---

# MCD-123 Android TV Stick — Setup and Quick Guide

## Summary (English)
Setup and optimization guide for the MCD-123 Android TV stick (Android 13.1). Covers: ADB connection, forcing 2.4GHz Wi‑Fi band, BSSID persistence, disabling bloatware, and diagnostics.

### Device
- Model: MCD-123
- Android: 13.1
- Recommendation: prefer 2.4GHz for stable network connectivity

## 1. ADB over USB
- Check ADB: `adb version`
- List devices: `adb devices` (accept authorization on the stick)

## 2. Wi‑Fi — Forcing 2.4GHz band
- Identify router BSSIDs for 2.4GHz and 5GHz
- Export/edit `WifiConfigStore.xml` via adb to set 2.4GHz BSSID
- Boot script (`fixbssid`) included to reapply on startup
- Note: permanent fix is disabling 5GHz on the router if possible

## 3. Disable Bloatware
- Use `pm disable-user` for unwanted system apps
- Proceed cautiously: disable one package at a time and reboot to test

## 4. Syslog and Logs (notebook openSUSE)
- Enable syslog-ng: `sudo systemctl enable --now syslog-ng`
- Logs located at `/var/log/messages`

## 5. Hardware and Firmware
- SoC: Allwinner H616 (sun50iw9p1)
- Firmware: unofficial builds; flashing risks bricking the device

## Useful diagnostics
Examples: `adb shell ping`, `dumpsys wifi`, `pm list packages -d`, `dumpsys activity services`.


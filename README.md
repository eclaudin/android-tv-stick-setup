# MCD-123 Android TV Stick — Setup e Otimizações

## Dispositivo
- **Modelo:** MCD-123
- **Android:** 13.1
- **Serial:** 0c001052258408f2314
- **MAC Wi-Fi:** 78:1e:b8:1d:dd:d8
- **Notebook:** openSUSE Tumbleweed (kernel 6.19.12)

---

## 1. Conexão ADB via USB

O stick não tem porta USB de dados — a conexão é feita pelo cabo USB-A que fica na lateral (usado normalmente para pen drive/teclado).

```bash
# Verificar se ADB está instalado
adb version

# Listar dispositivos (aceitar autorização na tela do stick)
adb devices
```

Ao conectar pela primeira vez, aparece um popup no stick pedindo autorização — confirmar "Sempre permitir".

---

## 2. Wi-Fi — Forçar banda 2.4GHz

O roteador é um **MitraStar GPT-2742GX4X5v6-SV** (Claro Brasil) que não permite separar os SSIDs por banda. O stick conectava no 5GHz (canal 36, freq 5180MHz) com latência absurda (~4000ms) e quedas a cada ~4 minutos.

**Solução:** Usar o modo IoT do roteador (desativa o 5GHz por 10 minutos) para forçar o stick a conectar no 2.4GHz, depois fixar o BSSID no config.

### BSSIDs do roteador
- **2.4GHz:** `e8:45:8b:b8:a2:20`
- **5GHz:** `e8:45:8b:b8:a2:27`

### Fixar BSSID 2.4GHz no stick
```bash
# Baixar config Wi-Fi do stick
adb shell su -c "cat /data/misc/wifi/WifiConfigStore.xml" > /tmp/WifiConfigStore.xml

# Substituir BSSID nulo pelo BSSID do 2.4GHz
sed -i 's|<null name="BSSID" />|<string name="BSSID">e8:45:8b:b8:a2:20</string>|' /tmp/WifiConfigStore.xml

# Enviar de volta para o stick
adb push /tmp/WifiConfigStore.xml /sdcard/WifiConfigStore.xml
adb shell su -c "cp /sdcard/WifiConfigStore.xml /data/misc/wifi/WifiConfigStore.xml"

# Reiniciar Wi-Fi
adb shell su -c "svc wifi disable" && sleep 2 && adb shell su -c "svc wifi enable"
```

> **Atenção:** O Android ignora o BSSID fixado e faz roaming automático para o 5GHz sempre que ele estiver disponível — mesmo com SSID broadcast desabilitado, pois a rede fica salva no dispositivo. A solução definitiva é **desativar a banda 5GHz no roteador**. Com o 5GHz fora, o stick permanece estável no 2.4GHz com sinal excelente (RSSI ~-17).

### Verificar se está no 2.4GHz
```bash
adb shell dumpsys wifi | grep -a "mWifiInfo"
# Deve mostrar Frequency: 2447MHz e BSSID: e8:45:8b:b8:a2:20
```

### Persistência após queda de energia

O Android pode sobrescrever o `WifiConfigStore.xml` ao reiniciar, zerando o BSSID. Para corrigir isso automaticamente no boot, foi instalado um serviço de sistema:

**`/system/bin/fixbssid`** — script que verifica e reaplicar o BSSID se necessário:
```sh
#!/system/bin/sh
sleep 10
BSSID="e8:45:8b:b8:a2:20"
CONF="/data/misc/wifi/WifiConfigStore.xml"
if grep -q '<null name="BSSID" />' "$CONF"; then
    svc wifi disable
    sed -i 's|<null name="BSSID" />|<string name="BSSID">'"$BSSID"'</string>|' "$CONF"
    svc wifi enable
fi
```

**`/system/etc/init/fixbssid.rc`** — registra o serviço para rodar no boot:
```
on property:sys.boot_completed=1
    start fixbssid

service fixbssid /system/bin/fixbssid
    disabled
    oneshot
    user root
    group shell
    seclabel u:r:shell:s0
```

Para instalar (requer `/system` gravável via `adb remount`):
```bash
adb root && adb remount
adb push fixbssid /system/bin/fixbssid
adb push fixbssid.rc /system/etc/init/fixbssid.rc
adb shell chmod +x /system/bin/fixbssid
```

---

## 3. Bloatware desabilitado

### Apps de fábrica chineses
```bash
for pkg in \
  com.ik.installxapk \
  com.android.netservice \
  com.factorytools.factorystability \
  com.softwinner.agingdragonbox \
  com.softwinner.dragonbox \
  com.softwinner.awlogsettings; do
    adb shell pm disable-user --user 0 "$pkg"
done
```

### Serviços desnecessários no TV stick
```bash
for pkg in \
  com.android.phone \
  com.android.server.telecom \
  com.android.providers.telephony \
  com.android.mms.service \
  com.android.simappdialog \
  com.android.se \
  com.android.printspooler \
  com.android.bips \
  com.android.printservice.recommendation \
  com.android.dreams.web \
  com.android.hotspot2 \
  com.softwinner.miracastReceiver \
  com.android.bookmarkprovider \
  com.android.music \
  com.android.gallery3d \
  com.android.egg; do
    adb shell pm disable-user --user 0 "$pkg"
done
```

Para reabilitar algum se necessário:
```bash
adb shell pm enable --user 0 <package>
```

---

## 4. syslog-ng no notebook (openSUSE)

O syslog-ng estava instalado mas inativo:

```bash
sudo systemctl enable --now syslog-ng
# Logs disponíveis em /var/log/messages
```

---

## 5. Hardware e Firmware

### Especificações do chip
- **SoC:** Allwinner H616 (`sun50iw9p1`)
- **CPU:** ARMv8 64-bit (4x Cortex-A53)
- **Android:** 13.1 — patch de segurança: 2019-12-05 (desatualizado)
- **Build:** `titan_p1-userdebug 10 QP1A.191105.004 test-keys` — firmware não oficial
- **Bootloader:** bloqueado (`ro.boot.flash.locked=1`, `verifiedbootstate=green`)

### Atualização de firmware / ROM customizada
Não há caminho oficial para atualização. O H616 tem comunidade ativa no XDA Developers — pesquisar por:
- `Allwinner H616 Android 13 ROM`
- `sun50iw9p1 custom ROM`
- Dispositivos compatíveis: X96 Max+, Tanix TX6S, TX9S

> **Atenção:** qualquer flash de ROM exige desbloquear o bootloader. Com bootloader bloqueado, o único caminho é via `dd` direto na partição usando root — risco de brick.

---

## 6. Diagnóstico útil

```bash
# Verificar conectividade
adb shell ping -c 3 8.8.8.8

# Ver latência para o gateway
adb shell ping -c 3 192.168.15.1

# Ver frequência Wi-Fi atual
adb shell dumpsys wifi | grep -a "mWifiInfo"

# Ver apps desabilitados
adb shell pm list packages -d

# Ver processos em background
adb shell dumpsys activity services | grep ServiceRecord
```

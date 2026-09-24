# Hailo PCIe driver: Raspberry Pi 5 workaround

Záznam lokální úpravy ovladače pro sestavu s Frigate na Raspberry Pi 5.

## Zaznamenaná konfigurace

- Hailo PCIe driver: `v4.21.0`
- Kernel: `6.18.50+rpt-rpi-2712`
- Patch: `hailo-rpi5-driver.patch`
- Zdroj: https://github.com/hailo-ai/hailort-drivers/tree/v4.21.0

Patch je určen pro zdrojové soubory Hailo PCIe driveru `v4.21.0`. Jeho aplikovatelnost na tuto verzi byla ověřena pomocí `git apply --check`.

## Změny

1. `linux/pcie/src/pcie.c`: výchozí velikost stránky deskriptoru je `min((u32)PAGE_SIZE, 4096u)`. Explicitní parametr `force_desc_page_size` má nadále vlastní větev zpracování.
2. Stejný soubor: `hailo_get_allocation_mode()` vynutí `HAILO_ALLOCATION_MODE_DRIVER`, vypíše hlášku a vrátí úspěch. Tím se obchází původní automatický výběr i zpracování parametru pro volbu alokace.
3. `linux/vdma/memory.c`: přidává `<linux/mm.h>` a `mmap_read_lock()` / `mmap_read_unlock()` bezprostředně kolem `find_vma()`.

Jde o archivaci konkrétního workaroundu, nikoli potvrzení obecné opravy ovladače. Zámek je uvolněn ještě před následným používáním ukazatele `vma`; tento patch proto sám o sobě nepotvrzuje bezpečnost všech přístupů k VMA při souběžných změnách mapování paměti.

## Aplikování na čisté zdroje

Použijte samostatnou čistou kopii zdrojů. Na již upraveném Raspberry Pi patch znovu neaplikujte.

```bash
git clone --branch v4.21.0 --depth 1 https://github.com/hailo-ai/hailort-drivers.git hailort-drivers-patched
cd hailort-drivers-patched
git apply --check /cesta/k/hailo-rpi5-driver.patch
git apply /cesta/k/hailo-rpi5-driver.patch
git diff --check
git diff
```

Pro kompilaci a instalaci postupujte podle dokumentace Hailo pro danou verzi a použijte hlavičky cílového kernelu. Konkrétní instalační postup ani funkčnost na hardwaru nebyly v rámci této archivace ověřeny.

Přidanou hlášku lze po načtení upraveného modulu hledat v kernel logu:

```text
Probing: Forcing driver allocated vdma buffers (RPi kernel workaround)
```

## Ověření

- `git diff --check`: úspěšné.
- `git apply --check` proti původním souborům v4.21.0: úspěšné.
- Kontrola zpětného aplikování patche: úspěšná.
- Kompilace, instalace a provoz s Frigate: neověřeno v rámci kontroly tohoto patche.

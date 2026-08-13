<img src="https://github.com/user-attachments/assets/4313b6fc-50c4-4b73-bc7e-6ac6a48fcd1a" />

Inferno2pipe per OpenWrt 24.10 con GitHub Actions

Scopo

Questo repository minimale compila Inferno2pipe come pacchetto OpenWrt IPK. Usa l'SDK ufficiale della release e del target scelti, il supporto Rust del feed ufficiale OpenWrt e il sorgente Inferno fissato al tag v0.5.4, commit 04c0efe05aff5e3c90a22160c7f2da24160c3293.

La build riguarda Inferno2pipe. Non comprende il plugin ALSA alsa_pcm_inferno, un servizio init, la configurazione del firewall o il demone PTP/Statime.

1. Creazione del repository

Creare un repository GitHub vuoto e copiarvi questi file mantenendo le directory:

```text
.github/workflows/build-inferno-openwrt.yml
inferno2pipe/Makefile
.gitignore
README.md
```

Eseguire quindi:

```sh
git add .
git commit -m "Build Inferno2pipe for OpenWrt 24.10"
git push
```

2. Individuazione di target e subtarget

Usare esattamente i due nomi di directory pubblicati sotto:

```text
https://downloads.openwrt.org/releases/24.10.7/targets/<target>/<subtarget>/
```

Esempi:

```text
x86/64
ath79/generic
bcm27xx/bcm2711
```

Sul dispositivo sono utili anche:

```sh
ubus call system board
cat /etc/openwrt_release
```

Non confondere target/subtarget con l'architettura dei pacchetti mostrata da `opkg print-architecture`.

3. Avvio della build

Aprire il repository su GitHub, selezionare Actions, poi "Build Inferno2pipe for OpenWrt 24.10" e "Run workflow". Inserire:

```text
openwrt_version: 24.10.7
target: x86
subtarget: 64
```

Il workflow accetta solo release con formato 24.10.N. Per un router diverso, cambiare target e subtarget con i valori individuati al punto precedente.

4. Operazioni effettuate dal workflow

1. Scarica Inferno v0.5.4 al commit fissato e inizializza i submodule ai commit registrati a monte.
2. Scarica l'SDK OpenWrt dalla directory ufficiale del target.
3. Ricava il nome e lo SHA-256 dell'SDK da `sha256sums` e verifica l'archivio prima di estrarlo.
4. Aggiorna soltanto il feed packages ufficiale e il feed locale.
5. Installa il supporto Rust del feed, usa `Cargo.lock` con `--locked` e compila Inferno2pipe con il cross-toolchain dell'SDK.
6. Pubblica l'IPK, `SHA256SUMS` e `BUILDINFO.txt` come artifact GitHub Actions.
7. Esegue le action di terze parti soltanto a revisioni SHA complete e usa il permesso minimo `contents: read`.

La prima esecuzione può essere lunga: l'SDK non contiene rustc e il target Rust viene costruito dal feed OpenWrt. Il job ha un limite di sei ore. Lo spazio e il tempo necessari variano in funzione dell'architettura e del runner.

5. Download e verifica

Al termine della run, scaricare l'artifact denominato, ad esempio:

```text
inferno2pipe-openwrt-24.10.7-x86-64
```

Estrarlo e verificare:

```sh
sha256sum -c SHA256SUMS
```

6. Installazione sul dispositivo

Verificare prima che release, target e architettura del dispositivo coincidano con quelli registrati in `BUILDINFO.txt`. Quindi:

```sh
scp inferno2pipe_*.ipk root@ROUTER:/tmp/
ssh root@ROUTER 'opkg install /tmp/inferno2pipe_*.ipk'
```

Controllare l'eseguibile:

```sh
inferno2pipe --help
```

7. Aggiornamento di Inferno

Per usare una release diversa occorre modificare in modo coerente:

1. `INFERNO_VERSION` e `INFERNO_COMMIT` nel workflow;
2. `PKG_VERSION` nel Makefile;
3. se cambiano dipendenze o struttura del workspace, il Makefile OpenWrt.

Usare un commit completo, non un branch mobile. Verificare sempre che i submodule risultino inizializzati e che `Cargo.lock` sia presente.

8. Limiti architetturali e operativi

Il feed Rust di OpenWrt 24.10.7 abilita aarch64, arm, i386, loongarch64, mips, mips64, mips64el, mipsel, powerpc, powerpc64, riscv64 e x86_64. Un target OpenWrt fuori da questo elenco non è compilabile con questo Makefile senza aggiungere supporto Rust.

Inferno è dichiarato a monte sperimentale e non ufficiale. La documentazione corrente richiede inoltre una corretta sincronizzazione PTP e specifiche aperture UDP. Applicare eventuali regole firewall soltanto sull'interfaccia o VLAN AoIP e limitare le sorgenti, invece di esporre i servizi su WAN.

L'autore segnala possibili implicazioni brevettuali nella distribuzione dei binari. Prima di distribuire il pacchetto oltre l'uso interno, effettuare la valutazione giuridica indicata dal progetto a monte.

Fonti tecniche

- Inferno: https://github.com/teodly/inferno
- Tag Inferno v0.5.4: https://github.com/teodly/inferno/tree/v0.5.4
- SDK OpenWrt 24.10.7 x86/64 e checksum: https://downloads.openwrt.org/releases/24.10.7/targets/x86/64/
- Revisioni dei feed OpenWrt 24.10.7: https://downloads.openwrt.org/releases/24.10.7/targets/x86/64/feeds.buildinfo
- Supporto architetture Rust del feed fissato: https://github.com/openwrt/packages/blob/0c8f5f2ed91600b04ba639ef9c4c5ced3286cb3b/lang/rust/rust-values.mk
- Artifact GitHub Actions: https://docs.github.com/actions/using-workflows/storing-workflow-data-as-artifacts

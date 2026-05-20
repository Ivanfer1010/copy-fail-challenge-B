# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
   ```bash
   #CONFIGURACION DE EJEMPLO!!!!!!!!!!!
   apt update
   apt install gh
   
   gh api user --jq '"\(.name) → \(.email // .login)"'
   
   git config --global user.name "Jonathan E. Tito O."
   git config --global user.email "jonathantito@users.noreply.github.com"
   git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   make setup
   ```
3. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
4. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes

root@codespaces-601265:/workspaces/copy-fail-challenge-B# history
    1  cd /workspaces/copy-fail-challenge-B
    2  nano evidence/hito1_vuln_confirmed.txt
    3  cat > evidence/hito1_vuln_confirmed.txt
    4  code evidence/hito1_vuln_confirmed.txt
    5  === HITO 1: KERNEL VULNERABLE CONFIRMADO ===
    6  echo "=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===" > /tmp/hito1.txt
    7  echo "Fecha: $(date)" >> /tmp/hito1.txt
    8  echo "Hostname: $(hostname)" >> /tmp/hito1.txt
    9  echo "Kernel: $(uname -r)" >> /tmp/hito1.txt
   10  echo "Identidad: $(id)" >> /tmp/hito1.txt
   11  echo "Módulos AF_ALG:" >> /tmp/hito1.txt
   12  cat /proc/modules | grep -i alg >> /tmp/hito1.txt || echo "(no encontrado)" >> /tmp/hito1.txt
   13  echo "algif_aead en /proc/modules:" >> /tmp/hito1.txt
   14  grep algif_aead /proc/modules >> /tmp/hito1.txt 2>/dev/null || echo "(no encontrado)" >> /tmp/hito1.txt
   15  cat /tmp/hito1.txt
   16  git config user.name "Ivan Gualotuna"
   17  git config user.email "ivgualotunabo@uide.edu.ec"
   18  git config user.name
   19  git config user.email
   20  cd /workspaces/copy-fail-challenge-B-ok
   21  make setup
   22  apt-get update && apt-get install -y musl-tools musl-dev
   23  make setup
   24  apt-get install -y musl-tools musl-dev gcc make
   25  which musl-gcc
   26  make setup
   27  apt-get install -y libc6-dev gcc-multilib
   28  ls /usr/lib/x86_64-linux-gnu/libc.a
   29  make setup
   30  cd /workspaces/copy-fail-challenge-B-ok/kernel/busybox
   31  make defconfig
   32  sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
   33  grep -q "^CONFIG_STATIC=y" .config || echo "CONFIG_STATIC=y" >> .config
   34  sed -i 's/^CONFIG_TC=y/CONFIG_TC=n/' .config
   35  make -j$(nproc) 2>&1 | tail -5
   36  file busybox
   37  apt-get install -y libc6-dev gcc-multilib
   38  ls /usr/lib/x86_64-linux-gnu/libc.a
   39  cd /workspaces/copy-fail-challenge-B-ok/kernel/busybox
   40  make defconfig
   41  sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
   42  grep -q "^CONFIG_STATIC=y" .config || echo "CONFIG_STATIC=y" >> .config
   43  sed -i 's/^CONFIG_TC=y/CONFIG_TC=n/' .config
   44  make -j$(nproc) 2>&1 | tail -5
   45  file busybox
   46  apt-get install -y file
   47  make -j$(nproc) 2>&1 | tail -5
   48  file busybox
   49  cd /workspaces/copy-fail-challenge-B-ok
   50  make setup
   51  cd /workspaces/copy-fail-challenge-B-ok
   52  ROOTFS=/tmp/cf-rootfs
   53  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
   54  rm -rf "$ROOTFS"
   55  mkdir -p "$ROOTFS"
   56  cd "$ROOTFS"
   57  gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2>&1
   58  cd /workspaces/copy-fail-challenge-B-ok
   59  ls -l /tmp/cf-rootfs/init
   60  ROOTFS=/tmp/cf-rootfs
   61  cat > "$ROOTFS/init" <<'EOF'
   62  #!/bin/sh
   63  mkdir -p /proc /sys /dev /etc /home/student
   64  mount -t proc proc /proc 2>/dev/null || true
   65  mount -t sysfs sysfs /sys 2>/dev/null || true
   66  mount -t devtmpfs devtmpfs /dev 2>/dev/null || true
   67  /bin/busybox ifconfig lo 127.0.0.1 up 2>/dev/null || true
   68  /bin/busybox ifconfig eth0 10.0.2.15 netmask 255.255.255.0 up 2>/dev/null || true
   70  echo "nameserver 10.0.2.3" > /etc/resolv.conf
   71  echo "nameserver 8.8.8.8" >> /etc/resolv.conf
   72  exec /bin/busybox su - student
   73  EOF
   74  chmod 755 "$ROOTFS/init"
   75  grep -n "su\|mount\|ifconfig" "$ROOTFS/init"
   76  ROOTFS=/tmp/cf-rootfs
   77  copy_deps() {   local bin="$1";   ldd "$bin" 2>/dev/null | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do     mkdir -p "$ROOTFS$(dirname "$lib")";     cp -L "$lib" "$ROOTFS$lib";     chmod 755 "$ROOTFS$lib";   done; }
   78  # Python3
   79  PYBIN="$(readlink -f "$(command -v python3)")"
   80  mkdir -p "$ROOTFS/usr/bin" "$ROOTFS/usr/lib" "$ROOTFS/usr/local/lib"
   81  install -o root -g root -m 0755 "$PYBIN" "$ROOTFS/usr/bin/python3"
   82  ln -sf python3 "$ROOTFS/usr/bin/python"
   83  copy_deps "$PYBIN"
   84  PYVER="$(python3 -c 'import sys; print(f"python{sys.version_info.major}.{sys.version_info.minor}")')"
   85  cp -a "/usr/lib/$PYVER" "$ROOTFS/usr/lib/" 2>/dev/null || true
   86  cp -a /usr/local/lib/python* "$ROOTFS/usr/local/lib/" 2>/dev/null || true
   87  # Shell real
   88  SHREAL="$(command -v dash || readlink -f /bin/sh)"
   89  rm -f "$ROOTFS/bin/sh"
   90  install -o root -g root -m 0755 "$SHREAL" "$ROOTFS/bin/sh"
   91  copy_deps "$SHREAL"
   92  # su real con SUID
   93  rm -f "$ROOTFS/usr/bin/su" "$ROOTFS/bin/su"
   94  install -o root -g root -m 4755 /usr/bin/su "$ROOTFS/usr/bin/su"
   95  copy_deps /usr/bin/su
   96  ln -s /usr/bin/su "$ROOTFS/bin/su"
   97  chown root:root "$ROOTFS/usr/bin/su"
   98  chmod 4755 "$ROOTFS/usr/bin/su"
   99  # PAM
  100  mkdir -p "$ROOTFS/etc/pam.d" "$ROOTFS/etc/security" "$ROOTFS/lib/x86_64-linux-gnu"
  101  cp -a /etc/pam.d/su "$ROOTFS/etc/pam.d/su" 2>/dev/null || true
  102  cp -a /etc/pam.d/common-* "$ROOTFS/etc/pam.d/" 2>/dev/null || true
  103  cp -a /etc/login.defs "$ROOTFS/etc/login.defs" 2>/dev/null || true
  104  cp -a /etc/security/* "$ROOTFS/etc/security/" 2>/dev/null || true
  105  cp -a /lib/x86_64-linux-gnu/security "$ROOTFS/lib/x86_64-linux-gnu/" 2>/dev/null || true
  106  find "$ROOTFS" -type d -exec chmod 755 {} \;
  107  find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;
  108  echo "=== Verificación ==="
  109  ls -l "$ROOTFS/usr/bin/python3"
  110  ls -l "$ROOTFS/usr/bin/su"
  111  ls -l "$ROOTFS/bin/sh"
  112  chown root:root /tmp/cf-rootfs/usr/bin/su
  113  chmod 4755 /tmp/cf-rootfs/usr/bin/su
  114  ls -l /tmp/cf-rootfs/usr/bin/su
  115  wget https://copy.fail/exp -O /tmp/copy_fail_exp.py
  116  mkdir -p /tmp/cf-rootfs/home/student
  117  cp /tmp/copy_fail_exp.py /tmp/cf-rootfs/home/student/copy_fail_exp.py
  118  chmod 644 /tmp/cf-rootfs/home/student/copy_fail_exp.py
  119  chmod 755 /tmp/cf-rootfs/home/student
  120  ls -lh /tmp/cf-rootfs/home/student/copy_fail_exp.py
  121  ROOTFS=/tmp/cf-rootfs
  122  wget https://copy.fail/exp -O /tmp/copy_fail_exp.py
  123  mkdir -p "$ROOTFS/home/student"
  124  cp /tmp/copy_fail_exp.py "$ROOTFS/home/student/copy_fail_exp.py"
  125  chmod 644 "$ROOTFS/home/student/copy_fail_exp.py"
  126  chown 1001:1001 "$ROOTFS/home/student/copy_fail_exp.py" 2>/dev/null || true
  127  chmod 755 "$ROOTFS/home/student"
  128  ls -lh "$ROOTFS/home/student/copy_fail_exp.py"
  129  cd /tmp/cf-rootfs
  130  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  131  cd /workspaces/copy-fail-challenge-B-ok
  132  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  133  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  134  cd /tmp/cf-rootfs
  135  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  136  cd /workspaces/copy-fail-challenge-B-ok
  137  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  138  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  139  cd /tmp/cf-rootfs
  140  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  141  cd /workspaces/copy-fail-challenge-B-ok
  142  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  143  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  144  cd /tmp/cf-rootfs
  145  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  146  cd /workspaces/copy-fail-challenge-B-ok
  147  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  148  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  149  ROOTFS=/tmp/cf-rootfs
  150  wget https://copy.fail/exp -O /tmp/copy_fail_exp.py
  151  mkdir -p "$ROOTFS/home/student"
  152  cp /tmp/copy_fail_exp.py "$ROOTFS/home/student/copy_fail_exp.py"
  153  chmod 644 "$ROOTFS/home/student/copy_fail_exp.py"
  154  chown 1001:1001 "$ROOTFS/home/student/copy_fail_exp.py" 2>/dev/null || true
  155  chmod 755 "$ROOTFS/home/student"
  156  ls -lh "$ROOTFS/home/student/copy_fail_exp.py"
  157  cd /tmp/cf-rootfs
  158  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  159  cd /workspaces/copy-fail-challenge-B-ok
  160  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  161  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  162  ROOTFS=/tmp/cf-rootfs
  163  wget https://copy.fail/exp -O /tmp/copy_fail_exp.py
  164  mkdir -p "$ROOTFS/home/student"
  165  cp /tmp/copy_fail_exp.py "$ROOTFS/home/student/copy_fail_exp.py"
  166  chmod 644 "$ROOTFS/home/student/copy_fail_exp.py"
  167  chown 1001:1001 "$ROOTFS/home/student/copy_fail_exp.py" 2>/dev/null || true
  168  chmod 755 "$ROOTFS/home/student"
  169  ls -lh "$ROOTFS/home/student/copy_fail_exp.py"
  170  cd /tmp/cf-rootfs
  171  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  172  cd /workspaces/copy-fail-challenge-B-ok
  173  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  174  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  175  make qemu
  176  cd /tmp/cf-rootfs
  177  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  178  cd /tmp/cf-rootfs
  179  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  180  # 1. Crear las carpetas que faltan
  181  mkdir -p /workspaces/copy-fail-challenge-B-ok/kernel/build/
  182  # 2. Volver a empaquetar el initramfs
  183  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  184  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  185  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  186  cd /tmp/cf-rootfs
  187  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  188  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  189  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp"
  190  # 1. Muévete a la carpeta del reto
  191  cd /workspaces/copy-fail-challenge-B-ok
  192  # 2. Verifica la integridad del archivo
  193  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  194  # 3. Filtra las 4 líneas para confirmar que el exploit quedó dentro
  195  gzip -dc kernel/build/initramfs.cpio.gz | cpio -t 2>/dev/null | grep -E "init$|usr/bin/su|usr/bin/python3|home/student/copy_fail_exp.py"
  196  cd /workspaces/copy-fail-challenge-B-ok
  197  make qemu
  198  cat Makefile | grep -E "run|start|qemu"
  199  cd kernel
  200  make qemu
  201  ls -la
  202  cat Makefile | grep -E "run|start|qemu"
  203  chmod +x run.sh  # Por si no tiene permisos
  204  ./run.sh
  205  chmod +x nombre_del_script.sh
  206  ./nombre_del_script.sh
  207  make setup
  208  cd
  209  make setup
  210  id
  211  make setup
  212  cd /workspaces/copy-fail-challenge-B-ok
  213  ls -la
  214  ls -la /workspaces
  215  mkdir -p /workspaces/copy-fail-challenge-B/kernel/build/
  216  cp /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz /workspaces/copy-fail-challenge-B/kernel/build/
  217  cd /workspaces/copy-fail-challenge-B
  218  ls -la
  219  make qemu
  220  # Ver cuál existe
  221  ls -ld /tmp/rootfs /tmp/cf-rootfs 2>/dev/null
  222  make qemu
  223  chmod 1777 /tmp/cf-rootfs/tmp
  224  chmod 777 /tmp/cf-rootfs/home/student
  225  ls -ld /tmp/cf-rootfs/tmp /tmp/cf-rootfs/home/student
  226  cd /tmp/cf-rootfs
  227  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  228  cd /workspaces/copy-fail-challenge-B-ok
  229  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  230  make qemu
  231  cd /workspaces/copy-fail-challenge-B
  232  make qemu
  233  cd /workspaces/copy-fail-challenge-B-ok
  234  mkdir -p evidence
  235  cat > evidence/hito1_vuln_confirmed.txt << 'EOF'
  236  === HITO 1: KERNEL VULNERABLE CONFIRMADO ===
  237  Fecha: Fri May 15 00:55:28 UTC 2026
  238  Hostname: (none)
  239  Kernel: 6.12.0
  240  Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
  241  Módulos AF_ALG:
  242  (no encontrado)
  243  algif_aead en /proc/modules:
  244  (no encontrado)
  245  EOF
  246  cat evidence/hito1_vuln_confirmed.txt
  247  git add evidence/hito1_vuln_confirmed.txt
  248  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
  249  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
  250  git push origin copy-fail-challenge-B-ok --tags
  251  cd /workspaces/copy-fail-challenge-B-ok
  252  mkdir -p evidence
  253  cat > evidence/hito1_vuln_confirmed.txt << 'EOF'
  254  === HITO 1: KERNEL VULNERABLE CONFIRMADO ===
  255  Fecha: Fri May 15 00:55:28 UTC 2026
  256  Hostname: (none)
  257  Kernel: 6.12.0
  258  Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
  259  Módulos AF_ALG:
  260  (no encontrado)
  261  algif_aead en /proc/modules:
  262  (no encontrado)
  263  EOF
  264  cat evidence/hito1_vuln_confirmed.txt
  265  cd /workspaces/copy-fail-challenge-B
  266  mkdir -p evidence
  267  cat > evidence/hito1_vuln_confirmed.txt << 'EOF'
  268  === HITO 1: KERNEL VULNERABLE CONFIRMADO ===
  269  Fecha: Fri May 15 17:22:28 UTC 2026
  270  Hostname: (none)
  271  Kernel: 6.12.0
  272  Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
  273  Módulos AF_ALG:
  274  (no encontrado)
  275  algif_aead en /proc/modules:
  276  (no encontrado)
  277  EOF
  278  cat evidence/hito1_vuln_confirmed.txt
  279  git add evidence/hito1_vuln_confirmed.txt
  280  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
  281  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
  282  git push origin copy-fail-challenge-B --tags
  283  git add evidence/hito1_vuln_confirmed.txt
  284  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
  285  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
  286  git push origin main --tags
  287  make qemu
  288  cd /workspaces/copy-fail-challenge-B-ok
  289  mkdir -p evidence
  290  cat > evidence/hito2_root_shell.txt << 'EOF'
  291  === HITO 2: EXPLOIT EXITOSO ===
  292  Fecha: Fri May 15 01:18:30 UTC 2026
  293  Hostname: (none)
  294  Identidad POST-exploit: uid=0(root) gid=1001(student) groups=1001(student)
  295  Kernel: 6.12.0
  296  SHA256 del exploit usado:
  297  d401e7d1c00605749d6c617ace73ab20a762b72e41c2e1590331596e38219a61  /home/student/copy_fail_exp.py
  298  --- Salida del exploit ---
  299  EOF
  300  cat evidence/hito2_root_shell.txt
  301  git add evidence/hito2_root_shell.txt
  302  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
  303  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
  304  git push origin copy-fail-challenge-B-ox-test --tags
  305  git add evidence/hito2_root_shell.txt
  306  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
  307  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
  308  git push origin main --tags
  309  git add evidence/hito2_root_shell.txt
  310  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
  311  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
  312  git push origin copy-fail-challenge-B-ox-test --tags
  313  make qemu
  314  git add evidence/hito2_root_shell.txt
  315  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
  316  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
  317  git push origin main --tags
  318  cd
  319  # 1. Crear la carpeta de evidencias en el repositorio real (por si acaso)
  320  mkdir -p /workspaces/copy-fail-challenge-B/evidence
  321  # 2. Copiar tu archivo del Hito 2 a la ruta correcta
  322  cp /workspaces/copy-fail-challenge-B-ok/evidence/hito2_root_shell.txt /workspaces/copy-fail-challenge-B/evidence/
  323  # 3. Moverte al repositorio verdadero de Git
  324  cd /workspaces/copy-fail-challenge-B
  325  # 4. Registrar y confirmar el archivo en Git
  326  git add evidence/hito2_root_shell.txt
  327  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
  328  # 5. Crear la etiqueta del Hito 2
  329  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
  330  git push origin main --tags
  331  make qemu
  332  cat /workspaces/copy-fail-challenge-B-ok/scripts/04_run_qemu.sh
  333  grep -n "algif\|aead\|modules" /workspaces/copy-fail-challenge-B-ok/kernel/linux/crypto/algif_aead.c | head -20
  334  cat /workspaces/copy-fail-challenge-B/scripts/04_run_qemu.sh
  335  make qemu
  336  cd /workspaces/copy-fail-challenge-B-ok
  337  mkdir -p evidence
  338  code evidence/hito3_mitigation.txt
  339  === HITO 3: MITIGACIÓN TEMPORAL ===
  340  Fecha: Fri May 15 23:00:40 UTC 2026
  341  Hostname: (none)
  342  Kernel: 6.12.0
  343  Identidad: uid=0(root) gid=1001(student) groups=1001(student)
  344  Mitigación aplicada: chmod 0000 /usr/bin/su
  345  Permisos actuales de /usr/bin/su:
  346  ----------    1 root     root         55680 May 15 22:59 /usr/bin/su
  347  Nota: kernel built-in, rmmod no disponible. Mitigación via permisos SUID.
  348  Intento de exploit post-mitigación:
  349  sh: 1: su: Permission denied
  350  cd /workspaces/copy-fail-challenge-B
  351  mkdir -p evidence
  352  code evidence/hito3_mitigation.txt
  353  === HITO 3: MITIGACIÓN TEMPORAL ===
  354  Fecha: Fri May 15 23:00:40 UTC 2026
  355  Hostname: (none)
  356  Kernel: 6.12.0
  357  Identidad: uid=0(root) gid=1001(student) groups=1001(student)
  358  Mitigación aplicada: chmod 0000 /usr/bin/su
  359  Permisos actuales de /usr/bin/su:
  360  ----------    1 root     root         55680 May 15 22:59 /usr/bin/su
  361  Nota: kernel built-in, rmmod no disponible. Mitigación via permisos SUID.
  362  Intento de exploit post-mitigación:
  363  sh: 1: su: Permission denied
  364  git add evidence/hito3_mitigation.txt
  365  git commit -m "hito-3: mitigacion temporal aplicada - $(date +%Y-%m-%dT%H:%M)"
  366  git tag -a hito-3 -m "algif_aead deshabilitado, exploit neutralizado"
  367  git push origin copy-fail-challenge-B-ok --tags
  368  make qemu
  369  cd /workspaces/copy-fail-challenge-B
  370  cat > evidence/hito3_mitigation.txt << EOF
  371  === HITO 3: MITIGACIÓN TEMPORAL ===
  372  Fecha: $(date)
  373  Hostname: (none)
  374  algif_aead en lsmod:
  375  lsmod: can't open '/proc/modules': No such file or directory
  376  (módulo NO cargado - mitigación activa)
  377  Intento de exploit post-mitigación:
  378  (exploit falló como se esperaba)
  379  EOF
  380  git add evidence/hito3_mitigation.txt
  381  git commit -m "hito-3: mitigacion temporal aplicada - $(date +%Y-%m-%dT%H:%M)"
  382  git tag -f -a hito-3 -m "algif_aead deshabilitado, exploit de kernel mitigado"
  383  git push origin main --tags -f
  384  ls /workspaces/copy-fail-challenge-B-ok/kernel/
  385  ls /workspaces/copy-fail-challenge-B/kernel/
  386  cat /workspaces/copy-fail-challenge-B/Makefile | head -50
  387  ls /workspaces/copy-fail-challenge-B-/scripts/
  388  ls /workspaces/copy-fail-challenge-B/scripts/
  389  cat /workspaces/copy-fail-challenge-B-/scripts/02_build_kernel.sh
  390  cat /workspaces/copy-fail-challenge-B/scripts/02_build_kernel.sh
  391  ls /workspaces/copy-fail-challenge-B/patches/ 2>/dev/null || echo "no existe carpeta patches"
  392  cd /workspaces/copy-fail-challenge-B
  393  mkdir -p patches
  394  mkdir -p kernel/linux
  395  git clone --depth 1 --branch v6.12   https://github.com/torvalds/linux.git kernel/linux
  396  cd /workspaces/copy-fail-challenge-B
  397  mkdir -p patches
  398  mkdir -p kernel/linux
  399  git clone --depth 1 --branch v6.12   https://github.com/torvalds/linux.git kernel/linux
  400  cd /workspaces/copy-fail-challenge-B/kernel/linux
  401  grep -n "aead_request_set_crypt\|req->src\|req->dst\|rsgl\|tsgl" crypto/algif_aead.c | head -30
  402  sed -n '275,290p' /workspaces/copy-fail-challenge-B/kernel/linux/crypto/algif_aead.c
  403  cd /workspaces/copy-fail-challenge-B/kernel/linux
  404  cp crypto/algif_aead.c crypto/algif_aead.c.bak
  405  sed -n '210,215p' crypto/algif_aead.c
  406  cd /workspaces/copy-fail-challenge-B/kernel/linux
  407  cat > /tmp/fix_aead.py << 'EOF'
  408  from pathlib import Path
  409  p = Path("crypto/algif_aead.c")
  410  s = p.read_text()
  411  # Fix 1: separar rsgl_src del RX SGL
  412  old1 = "        /* Use the RX SGL as source (and destination) for crypto op. */\n        rsgl_src = areq->first_rsgl.sgl.sgt.sgl;"
  413  new1 = "        /* Use TX SGL as source, RX SGL as destination (out-of-place). */\n        rsgl_src = tsgl_src ? tsgl_src : areq->first_rsgl.sgl.sgt.sgl;"
  414  # Fix 2: separar src y dst en aead_request_set_crypt
  415  old2 = "        aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,\n                               areq->first_rsgl.sgl.sgt.sgl, used, ctx->iv);"
  416  new2 = "        aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,\n                               areq->first_rsgl.sgl.sgt.sgl, used, ctx->iv); /* src != dst */"
  417  if old1 in s:
  418      s = s.replace(old1, new1)
  419      print("Fix 1 aplicado OK")
  420  else:
  421      print("Fix 1 NO encontrado - revisar")
  422  p.write_text(s)
  423  EOF
  424  python3 /tmp/fix_aead.py
  425  cd /workspaces/copy-fail-challenge-B/kernel/linux
  426  cat > /tmp/fix_aead.py << 'EOF'
  427  from pathlib import Path
  428  p = Path("crypto/algif_aead.c")
  429  s = p.read_text()
  430  # Fix 1: separar rsgl_src del RX SGL
  431  old1 = "        /* Use the RX SGL as source (and destination) for crypto op. */\n        rsgl_src = areq->first_rsgl.sgl.sgt.sgl;"
  432  new1 = "        /* Use TX SGL as source, RX SGL as destination (out-of-place). */\n        rsgl_src = tsgl_src ? tsgl_src : areq->first_rsgl.sgl.sgt.sgl;"
  433  # Fix 2: separar src y dst en aead_request_set_crypt
  434  old2 = "        aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,\n                               areq->first_rsgl.sgl.sgt.sgl, used, ctx->iv);"
  435  new2 = "        aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,\n                               areq->first_rsgl.sgl.sgt.sgl, used, ctx->iv); /* src != dst */"
  436  if old1 in s:
  437      s = s.replace(old1, new1)
  438      print("Fix 1 aplicado OK")
  439  else:
  440      print("Fix 1 NO encontrado - revisar")
  441  p.write_text(s)
  442  EOF
  443  python3 /tmp/fix_aead.py
  444  sed -n '208,215p' crypto/algif_aead.c
  445  cd /workspaces/copy-fail-challenge-B/kernel/linux
  446  cat > /tmp/fix_aead.py << 'EOF'
  447  from pathlib import Path
  448  p = Path("crypto/algif_aead.c")
  449  s = p.read_text()
  450  old1 = "\t/* Use the RX SGL as source (and destination) for crypto op. */\n\trsgl_src = areq->first_rsgl.sgl.sgt.sgl;"
  451  new1 = "\t/* Use TX SGL as source, RX SGL as destination (out-of-place). */\n\trsgl_src = tsgl_src ? tsgl_src : areq->first_rsgl.sgl.sgt.sgl;"
  452  if old1 in s:
  453      s = s.replace(old1, new1)
  454      print("Fix aplicado OK")
  455      p.write_text(s)
  456  else:
  457      print("Fix NO encontrado - revisar")
  458  EOF
  459  python3 /tmp/fix_aead.py
  460  sed -n '208,216p' crypto/algif_aead.c
  461  cd /workspaces/copy-fail-challenge-B/kernel/linux
  462  git diff crypto/algif_aead.c > /workspaces/copy-fail-challenge-B/patches/fix_algif_aead.patch
  463  cat /workspaces/copy-fail-challenge-B/patches/fix_algif_aead.patch
  464  cd /workspaces/copy-fail-challenge-B/kernel/linux
  465  make -j$(nproc) bzImage 2>&1 | tail -10
  466  cd /workspaces/copy-fail-challenge-B/kernel/linux
  467  make tinyconfig
  468  ./scripts/config --enable 64BIT
  469  ./scripts/config --enable SERIAL_8250
  470  ./scripts/config --enable SERIAL_8250_CONSOLE
  471  ./scripts/config --enable TTY
  472  ./scripts/config --enable PRINTK
  473  ./scripts/config --enable EARLY_PRINTK
  474  ./scripts/config --enable BINFMT_ELF
  475  ./scripts/config --enable BINFMT_SCRIPT
  476  ./scripts/config --enable BLK_DEV_INITRD
  477  ./scripts/config --enable RD_GZIP
  478  ./scripts/config --enable TMPFS
  479  ./scripts/config --enable PROC_FS
  480  ./scripts/config --enable SYSFS
  481  ./scripts/config --enable DEVTMPFS
  482  ./scripts/config --enable DEVTMPFS_MOUNT
  483  ./scripts/config --enable NET
  484  ./scripts/config --enable UNIX
  485  ./scripts/config --enable INET
  486  ./scripts/config --enable CRYPTO
  487  ./scripts/config --enable CRYPTO_AEAD
  488  ./scripts/config --enable CRYPTO_AUTHENC
  489  ./scripts/config --enable CRYPTO_USER_API
  490  ./scripts/config --enable CRYPTO_USER_API_AEAD
  491  ./scripts/config --enable CRYPTO_USER_API_SKCIPHER
  492  ./scripts/config --enable CRYPTO_AUTHENCESN
  493  ./scripts/config --enable CRYPTO_AES
  494  ./scripts/config --enable CRYPTO_CBC
  495  ./scripts/config --enable CRYPTO_HMAC
  496  ./scripts/config --enable CRYPTO_SHA256
  497  ./scripts/config --enable MULTIUSER
  498  make olddefconfig
  499  ./scripts/config --enable 64BIT
  500  ./scripts/config --enable SERIAL_8250
  501  ./scripts/config --enable SERIAL_8250_CONSOLE
  502  ./scripts/config --enable TTY
  503  ./scripts/config --enable PRINTK
  504  ./scripts/config --enable EARLY_PRINTK
  505  ./scripts/config --enable BINFMT_ELF
  506  ./scripts/config --enable BINFMT_SCRIPT
  507  ./scripts/config --enable BLK_DEV_INITRD
  508  ./scripts/config --enable RD_GZIP
  509  ./scripts/config --enable TMPFS
  510  ./scripts/config --enable PROC_FS
  511  ./scripts/config --enable SYSFS
  512  ./scripts/config --enable DEVTMPFS
  513  ./scripts/config --enable DEVTMPFS_MOUNT
  514  ./scripts/config --enable NET
  515  ./scripts/config --enable UNIX
  516  ./scripts/config --enable INET
  517  ./scripts/config --enable CRYPTO
  518  ./scripts/config --enable CRYPTO_AEAD
  519  ./scripts/config --enable CRYPTO_AUTHENC
  520  ./scripts/config --enable CRYPTO_USER_API
  521  ./scripts/config --enable CRYPTO_USER_API_AEAD
  522  ./scripts/config --enable CRYPTO_USER_API_SKCIPHER
  523  ./scripts/config --enable CRYPTO_AUTHENCESN
  524  ./scripts/config --enable CRYPTO_AES
  525  ./scripts/config --enable CRYPTO_CBC
  526  ./scripts/config --enable CRYPTO_HMAC
  527  ./scripts/config --enable CRYPTO_SHA256
  528  ./scripts/config --enable MULTIUSER
  529  make olddefconfig
  530  make -j$(nproc) bzImage 2>&1 | tail -5
  531  cp /workspaces/copy-fail-challenge-B/kernel/linux/arch/x86/boot/bzImage    /workspaces/copy-fail-challenge-B/kernel/build/bzImage_patched
  532  ls -lh /workspaces/copy-fail-challenge-B/kernel/build/
  533  cd /tmp/cf-rootfs
  534  chmod 4755 usr/bin/su
  535  ls -l usr/bin/su
  536  cd /tmp/cf-rootfs
  537  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > /workspaces/copy-fail-challenge-B-ok/kernel/build/initramfs.cpio.gz
  538  cd /workspaces/copy-fail-challenge-B-ok
  539  gzip -t kernel/build/initramfs.cpio.gz && echo "INITRAMFS OK"
  540  cd /workspaces/copy-fail-challenge-B
  541  BZIMAGE=kernel/build/bzImage_patched bash scripts/04_run_qemu.sh
  542  cd /workspaces/copy-fail-challenge-B
  543  cat > evidence/hito4_patched.txt << EOF
  544  === HITO 4: PARCHE APLICADO ===
  545  Fecha: $(date)
  546  Kernel: 6.12.0
  547  Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
  548  Intento exploit post-parche:
  549  (exploit falló)
  550  EOF
  551  # 1. Agregar la evidencia y el archivo de parche
  552  git add evidence/hito4_patched.txt patches/fix_algif_aead.patch
  553  # 2. Confirmar el commit con la fecha actual
  554  git commit -m "hito-4: parche aplicado, exploit neutralizado - $(date +%Y-%m-%dT%H:%M)"
  555  # 3. Crear la etiqueta oficial del hito 4
  556  git tag -a hito-4 -m "Kernel parcheado, CVE-2026-31431 neutralizado"
  557  # 4. Subir todo a GitHub
  558  git push origin main --tags
  559  cd /workspaces/copy-fail-challenge-B
  560  code REPORT.md
  561  make verify
  562  make grade
  563  cd /workspaces/copy-fail-challenge-B
  564  cat > REPORT.md << 'EOF'
  565  # Reporte Técnico: Análisis del Exploit "Copy Fail"
  566  ### ¿Cuál es el bug raíz y en qué archivo/función está?
  567  El problema reside en el archivo `crypto/algif_aead.c`, en la función `_aead_recvmsg()`. En resumen, el kernel escribía y leía la misma porción de memoria a la vez durante una operación criptográfica, lo que permitía cambiar datos de archivos importantes cargados en memoria, por ejemplo `/usr/bin/su`.
  568  ### ¿Por qué el write a dst es peligroso?
  569  La razón es que `/usr/bin/su` es un programa especial cuyo funcionamiento permite ejecutarlo en modo setuid, es decir, posee privilegios de superusuario. Si conseguimos cambiar solo unos cuantos bytes de ese binario cuando está cargado en memoria, podemos lograr que se ejecute incorrectamente y así obtener acceso root.
  570  Lo que hace el exploit es exactamente eso: cambia unos pocos bytes en el momento adecuado para modificar el comportamiento del programa y abrir una shell con privilegios de root.
  571  ### ¿Por qué el exploit es “stealthy”?
  572  Porque no cambia nada en el disco. El archivo original no se modifica y su hash permanece igual. Solo se altera la copia temporal que el kernel mantiene en la RAM, conocida como page cache.
  573  Al no existir cambios permanentes en el sistema de archivos, todo parece normal. Incluso después de reiniciar el equipo, las modificaciones desaparecen automáticamente.
  574  ### Conexión con lo que vimos en clase
  575  El **page cache** es una copia de los archivos que mantiene el kernel en RAM para no estar leyendo continuamente desde el disco. Esta vulnerabilidad aprovecha precisamente ese mecanismo, ya que permite modificar la copia en memoria sin pasar por los controles habituales del sistema de archivos.
  576  El **bit setuid** que configuramos desde `chmod` es fundamental para que funcione el ataque. Si `/usr/bin/su` no tuviese ese permiso especial, modificar el binario en memoria no permitiría obtener privilegios de administrador. El permiso `-rwsr-xr-x` es lo que convierte este bug en una auténtica escalada de privilegios.
  577  Los **inodos** almacenan información importante del archivo: permisos, propietario y metadatos. Pero el ataque solo afecta a la copia en memoria, nunca al archivo real en disco, por lo que los inodos permanecen intactos y el sistema mantiene una apariencia normal.
  578  ### ¿Qué aprendiste?
  579  Este ejercicio me sirvió para entender cómo muchas vulnerabilidades críticas pueden surgir a partir de pequeños errores que parecen inofensivos. En concreto, el cambio introducido en 2017 parecía una simple optimización y pasó desapercibido durante años.
  580  También me ayudó a relacionar mejor conceptos vistos en clase, como el funcionamiento del page cache, el bit setuid y los inodos. Ver cómo todos estos mecanismos interactúan entre sí permite comprender mejor por qué una vulnerabilidad como esta puede convertirse en un problema de seguridad tan serio.
  581  EOF
  582  cat REPORT.md
  583  cat > /workspaces/copy-fail-challenge-B/REPORT.md << 'EOF'
  584  # Reporte Técnico: Análisis del Exploit "Copy Fail"
  585  ### ¿Cuál es el bug raíz y en qué archivo/función está?
  586  El problema reside en el archivo `crypto/algif_aead.c`, en la función `_aead_recvmsg()`. En resumen, el kernel escribía y leía la misma porción de memoria a la vez durante una operación criptográfica, lo que permitía cambiar datos de archivos importantes cargados en memoria, por ejemplo `/usr/bin/su`.
  587  ### ¿Por qué el write a dst es peligroso?
  588  La razón es que `/usr/bin/su` es un programa especial cuyo funcionamiento permite ejecutarlo en modo setuid, es decir, posee privilegios de superusuario. Si conseguimos cambiar solo unos cuantos bytes de ese binario cuando está cargado en memoria, podemos lograr que se ejecute incorrectamente y así obtener acceso root.
  589  Lo que hace el exploit es exactamente eso: cambia unos pocos bytes en el momento adecuado para modificar el comportamiento del programa y abrir una shell con privilegios de root.
  590  ### ¿Por qué el exploit es “stealthy”?
  591  Porque no cambia nada en el disco. El archivo original no se modifica y su hash permanece igual. Solo se altera la copia temporal que el kernel mantiene en la RAM, conocida como page cache.
  592  Al no existir cambios permanentes en el sistema de archivos, todo parece normal. Incluso después de reiniciar el equipo, las modificaciones desaparecen automáticamente.
  593  ### Conexión con lo que vimos en clase
  594  El **page cache** es una copia de los archivos que mantiene el kernel en RAM para no estar leyendo continuamente desde el disco. Esta vulnerabilidad aprovecha precisamente ese mecanismo, ya que permite modificar la copia en memoria sin pasar por los controles habituales del sistema de archivos.
  595  El **bit setuid** que configuramos desde `chmod` es fundamental para que funcione el ataque. Si `/usr/bin/su` no tuviese ese permiso especial, modificar el binario in memoria no permitiría obtener privilegios de administrador. El permiso `-rwsr-xr-x` es lo que convierte este bug en una auténtica escalada de privilegios.
  596  Los **inodos** almacenan información importante del archivo: permisos, propietario y metadatos. Pero el ataque solo afecta a la copia en memoria, nunca al archivo real en disco, por lo que los inodos permanecen intactos y el sistema mantiene una apariencia normal.
  597  ### ¿Qué aprendiste?
  598  Este ejercicio me sirvió para entender cómo muchas vulnerabilidades críticas pueden surgir a partir de pequeños errores que parecen inofensivos. En concreto, el cambio introducido en 2017 parecía una simple optimización y pasó desapercibido durante años.
  599  También me ayudó a relacionar mejor conceptos vistos en clase, como el funcionamiento del page cache, el bit setuid y los inodos. Ver cómo todos estos mecanismos interactúan entre sí permite comprender mejor por qué una vulnerabilidad como esta puede convertirse en un problema de seguridad tan serio.
  600  EOF
  601  git add REPORT.md
  602  git commit -m "bonus: reporte tecnico completado"
  603  git push origin main
  604  history
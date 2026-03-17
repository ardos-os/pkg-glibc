pkgname=glibc-ardos
pkgver=2.41 # Versão estável recomendada
pkgrel=1
pkgdesc="GNU C Library - Ardos OS Core (Debloated)"
arch=('x86_64')
url='https://www.gnu.org/software/libc'
license=('GPL' 'LGPL')
# Note: Removido lib32 e gd (memusagestat não é necessário no SO final)
makedepends=('git' 'python')
options=('!lto' '!emptydirs' 'staticlibs' "!buildflags")
source=("git+https://sourceware.org/git/glibc.git#tag=glibc-${pkgver}")
sha256sums=('SKIP')

prepare() {
  mkdir -p glibc-build
  cd glibc
  
  # Aqui é onde você poderá aplicar patches para o Ardos Registry no futuro
  # Exemplo: patch -p1 < ../ardos-registry-nss.patch
}

build() {
  cd glibc-build

  # FLAGS de Otimização Ardos
  # -enable-kernel=6.1: Garante uso de syscalls modernas, removendo código legado
  # -disable-nscd: Você gerenciará cache/registry de outra forma
  local _configure_flags=(
      --prefix=/usr
      --libdir=/usr/lib
      --libexecdir=/usr/lib
      --with-headers=/usr/include
      --enable-bind-now
      --enable-fortify-source
      --enable-kernel=6.1
      --enable-multi-arch
      --enable-stack-protector=strong
      --disable-nscd
      --disable-profile
      --disable-werror
      --enable-cet
  )

  echo "slibdir=/usr/lib" >> configparms
  echo "rtlddir=/usr/lib" >> configparms
  echo "sbindir=/usr/bin" >> configparms

  ../glibc/configure "${_configure_flags[@]}"

  make -O
}

package() {
  cd glibc-build
  make DESTDIR="${pkgdir}" install

  # --- ARDOS DEBLOAT SECTION ---
  
  # 1. Remover documentação e internacionalização não solicitada
  rm -rf "${pkgdir}"/usr/share/{info,man,doc,gettext}
  rm -rf "${pkgdir}"/usr/bin/xtrace # Script de debug perl
  
  # 2. Remover Headers (O Ardos-Packer não precisa deles na imagem final)
  # Nota: Eles estarão no .tar.zst temporário, mas você pode removê-los aqui
  # para que nem cheguem ao sysroot do SquashFS.
  rm -rf "${pkgdir}"/usr/include

  # 3. Remover bibliotecas estáticas (manter apenas as .so dinâmicas)
  find "${pkgdir}"/usr/lib -name "*.a" -delete

  # 4. Remover binários de suporte desnecessários
  # tzselect e zic já estão no tzdata; pldd/sln são raramente usados em prod
  rm -f "${pkgdir}"/usr/bin/{tzselect,zdump,zic,pldd,sln}

  # 5. Ardos Registry Readiness: nsswitch.conf
  # No futuro, o Ardos Registry injetará configurações aqui ou via biblioteca customizada
  # Por agora, garantimos um arquivo limpo
  install -dm755 "${pkgdir}"/etc
  cat > "${pkgdir}"/etc/nsswitch.conf <<EOF
passwd: files
group: files
shadow: files
hosts: files dns
networks: files
protocols: files
services: files
ethers: files
rpc: files
netgroup: files
EOF

  # Remover cache de bibliotecas (o initramfs/tibs rodará ldconfig se necessário)
  rm -f "${pkgdir}"/etc/ld.so.cache
}
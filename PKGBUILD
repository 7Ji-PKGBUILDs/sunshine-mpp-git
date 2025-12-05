# Maintainer: Grey Christoforo <first name at last name dot net>

_sunshine_base=26e5f2f610cf6c33ed1acf0b67c2fb8d1f383a18
_sunshine_branch=mpp
pkgname=sunshine-mpp-git
pkgver=r2952.7c6e43fb
pkgrel=1
pkgdesc="Game Stream server for Moonlight, latest git"
arch=('aarch64' 'x86_64')
url=https://github.com/LizardByte/Sunshine
license=('GPL3')
install=sunshine-mpp-git.install
options=(!lto)

depends=(
avahi
boost-libs
curl
libayatana-appindicator
libevdev
#libmfx
libnotify
libpulse
libva
libvdpau
libx11
libxcb
libxfixes
libxrandr
libxtst
miniupnpc
numactl
openssl
opus
udev
mpp
)
makedepends=(
git
boost
cmake
ninja
nodejs
npm
clang
)
makedepends_x86_64=("nasm")

optdepends=(
'cuda: NvFBC capture support'
'libcap'
'libdrm'
)

provides=(sunshine)
conflicts=(sunshine sunshine-git sunshine-bin)

source=(
"git+https://github.com/LizardByte/Sunshine.git"
"git+https://github.com/nyanmisaka/ffmpeg-rockchip.git#branch=7.1"
 mpp.patch::https://github.com/hbiyik/Sunshine/compare/${_sunshine_base}...${_sunshine_branch}.patch
)

sha256sums=('SKIP'
            'SKIP'
            'SKIP')

pkgver() {
  cd $srcdir/Sunshine
  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd $srcdir/Sunshine
  git submodule update --init --recursive --depth=1 --jobs=4 -f
  patch -p1 -N -i ../mpp.patch
}

build() {
  _stage_dir=$srcdir/Sunshine/third-party/build-deps/dist/Linux-${CARCH}
  export PKG_CONFIG_PATH="${_stage_dir}/lib/pkgconfig"
  sed -i -E "s|prefix\=.+|prefix\=$_stage_dir|" ${PKG_CONFIG_PATH}/*.pc
  cp $PKG_CONFIG_PATH/x265.pc $PKG_CONFIG_PATH/x264.pc
  sed -i -E "s|x265|x264|" ${PKG_CONFIG_PATH}/x264.pc
  

  cd $srcdir/ffmpeg-rockchip
  ./configure \
    --disable-all \
    --disable-autodetect \
    --disable-iconv \
    --enable-amf \
    --enable-encoder=h264_amf,hevc_amf,av1_amf \
    --enable-encoder=h264_vaapi,hevc_vaapi,av1_vaapi \
    --enable-encoder=h264_rkmpp,hevc_rkmpp \
    --enable-encoder=h264_v4l2m2m \
    --enable-encoder=libsvtav1 \
    --enable-encoder=libx264,libx265 \
    --enable-libsvtav1 \
    --enable-libx264 \
    --enable-libx265 \
    --enable-vaapi \
    --enable-v4l2_m2m \
    --enable-rkmpp \
    --enable-version3 \
    --enable-libdrm \
    --prefix=$_stage_dir \
    --extra-cflags="-I$_stage_dir/include" \
    --extra-ldflags="-L$_stage_dir/lib" \
    --extra-libs="-lpthread -lm" \
    --pkg-config-flags="--static" \
    --enable-avcodec \
    --enable-swscale \
    --enable-gpl \
    --enable-static \
    --pkg-config=pkg-config
  make install

  cd $srcdir/Sunshine
  npm install

  cmake -B build_dir -S . -W no-dev -G Ninja \
    -D FFMPEG_PREPARED_BINARIES="$_stage_dir" \
    -D FFMPEG_PLATFORM_LIBRARIES="rockchip_mpp $_stage_dir/lib/libx264.a $_stage_dir/lib/libx265.a $_stage_dir/lib/libSvtAv1Enc.a -lnuma" \
    -D CMAKE_BUILD_TYPE=Release \
    -D CMAKE_INSTALL_PREFIX=/usr \
    -D SUNSHINE_EXECUTABLE_PATH=/usr/bin/sunshine \
    -D SUNSHINE_ASSETS_DIR="share/sunshine" \
    -D SUNSHINE_ENABLE_X11=ON \
    -D CUDA_FAIL_ON_MISSING=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_DOCS=0

	# for debugging
    # -D CMAKE_BUILD_TYPE=Debug \
    # -D CMAKE_C_FLAGS_DEBUG="-g -O0" \
    # -D CMAKE_CXX_FLAGS_DEBUG="-g -O0" \

  # due to excessive memory consumption limit jobs
  cmake --build build_dir -j4
}

package() {
  cd $srcdir/Sunshine
  DESTDIR="${pkgdir}" cmake --install build_dir
}

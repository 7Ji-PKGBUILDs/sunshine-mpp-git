# Maintainer: Grey Christoforo <first name at last name dot net>

_sunshine_base=0107ca44d7ecce3ce1c46102decdf891d81e6816
_sunshine_branch=mpp
pkgname=sunshine-mpp-git
pkgver=r2524.8810c5cc
pkgrel=1
pkgdesc="Game Stream server for Moonlight, latest git"
arch=('x86_64' 'aarch64')
url=https://github.com/LizardByte/Sunshine
license=('GPL3')
install=sunshine-mpp-git.install

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
optdepends=(
'cuda: NvFBC capture support'
'libcap'
'libdrm'
)

provides=(sunshine)
conflicts=(sunshine sunshine-git sunshine-bin)

source=(
"git+https://github.com/LizardByte/Sunshine.git#branch=master"
git+https://github.com/flatpak/flatpak-builder-tools.git
git+https://github.com/flathub/shared-modules
"git+https://github.com/LizardByte/build-deps.git#commit=b2459cc3b1a8a6727c87b04d8c82f7a435b35165"
git+https://github.com/LizardByte/doxyconfig.git
git+https://github.com/google/googletest
git+https://github.com/games-on-whales/inputtino.git
git+https://github.com/moonlight-stream/moonlight-common-c.git
git+https://github.com/sleepybishop/nanors.git
git+https://github.com/FFmpeg/nv-codec-headers
git+https://github.com/LizardByte/nvapi-open-source-sdk
git+https://gitlab.com/eidheim/Simple-Web-Server.git
git+https://github.com/michaeltyson/TPCircularBuffer.git
git+https://github.com/LizardByte/tray
git+https://github.com/LizardByte/Virtual-Gamepad-Emulation-Client.git
git+https://gitlab.freedesktop.org/wayland/wayland-protocols.git
git+https://gitlab.freedesktop.org/wlroots/wlr-protocols.git
git+https://github.com/cgutman/enet.git
"git+https://github.com/nyanmisaka/ffmpeg-rockchip.git#branch=6.1"
git+https://github.com/GPUOpen-LibrariesAndSDKs/AMF
git+https://code.videolan.org/videolan/x264.git
git+https://bitbucket.org/multicoreware/x265_git.git
git+https://gitlab.com/AOMediaCodec/SVT-AV1.git
mpp.patch::https://github.com/hbiyik/Sunshine/compare/${_sunshine_base}...${_sunshine_branch}.patch
)

sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            '2dccb55c71c0966a7d07e2474f79987db14a0d655df6a6c1bb4063786bf64d02')

pkgver() {
  cd $srcdir/Sunshine
  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd $srcdir/build-deps
  git rm ffmpeg_sources/ffmpeg -f
  rm -rf ffmpeg_sources/ffmpeg
  ln -sf $srcdir/ffmpeg-rockchip ffmpeg_sources/ffmpeg
  git config submodule.ffmpeg_sources/AMF.url "${srcdir}/AMF"
  git config submodule.ffmpeg_sources/SVT-AV1.url "${srcdir}/SVT-AV1"
  git config submodule.ffmpeg_sources/nv-codec-headers.url "${srcdir}/nv-codec-headers"
  git config submodule.ffmpeg_sources/x264.url "${srcdir}/x264"
  git config submodule.ffmpeg_sources/x265_git.url "${srcdir}/x265_git"        
  git -c protocol.file.allow=always submodule init 
  git -c protocol.file.allow=always submodule update -f
  
  cd $srcdir/Sunshine
  git rm third-party/build-deps -f
  rm -rf third-party/build-deps
  git config submodule.third-party/flatpak-builder-tools.url "${srcdir}/flatpak-builder-tools"
  git config submodule.third-party/shared-modules.url "${srcdir}/shared-modules"
  git config submodule.third-party/doxyconfig.url "${srcdir}/doxyconfig"
  git config submodule.third-party/googletest.url "${srcdir}/googletest"
  git config submodule.third-party/inputtino.url "${srcdir}/inputtino"
  git config submodule.third-party/moonlight-common-c.url "${srcdir}/moonlight-common-c"
  git config submodule.third-party/tray.url "${srcdir}/tray"
  git config submodule.third-party/Virtual-Gamepad-Emulation-Client.url "${srcdir}/Virtual-Gamepad-Emulation-Client"
  git config submodule.third-party/wayland-protocols.url "${srcdir}/wayland-protocols"
  git config submodule.third-party/wlr-protocols.url "${srcdir}/wlr-protocols"
  git -c protocol.file.allow=always submodule init
  git -c protocol.file.allow=always submodule update -f
  patch -p1 -N -i ../mpp.patch

  cd $srcdir/Sunshine/third-party/moonlight-common-c
  git config submodule.enet.url "${srcdir}/enet"
  git -c protocol.file.allow=always submodule init
  git -c protocol.file.allow=always submodule update -f
}

build() {
  _stage_dir=$srcdir/build-deps/staging
  mkdir -p $_stage_dir/include/AMF

  export CFLAGS="${CFLAGS/-Werror=format-security/}"
  export CXXFLAGS="${CXXFLAGS/-Werror=format-security/}"
  export PKG_CONFIG_PATH=$_stage_dir/lib/pkgconfig

  cd $srcdir/build-deps
  cmake -B build_dir -S . -W no-dev -G Ninja \
    -D CMAKE_C_COMPILER=$(which gcc) \
    -D CMAKE_CXX_COMPILER=$(which g++) \
    -D CMAKE_AR=$(which ar) \
    -D CMAKE_RANLIB=$(which ranlib) \
    -D CMAKE_INSTALL_PREFIX=$_stage_dir
  cmake --build build_dir
  cmake --install build_dir

  cp -av "$srcdir/build-deps/ffmpeg_sources/AMF/amf/public/include/." "$_stage_dir/include/AMF"
  
  cd $srcdir/build-deps/ffmpeg_sources/SVT-AV1
  cmake -B build_dir -S . -W no-dev -G Ninja \
    -DCMAKE_INSTALL_PREFIX="$_stage_dir" \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_APPS=OFF \
    -DBUILD_DEC=OFF \
    -DENABLE_AVX512=ON \
    -DBUILD_SHARED_LIBS=OFF
  cmake --build build_dir
  cmake --install build_dir
  
  cd $srcdir/build-deps/ffmpeg_sources/x264
  ./configure \
    --prefix="$_stage_dir" \
    --disable-cli \
    --enable-static
  make install
  
  cd $srcdir/build-deps/ffmpeg_sources/x265_git
  cmake -B build_dir -S source -W no-dev -G Ninja \
    -DCMAKE_INSTALL_PREFIX="$_stage_dir" \
    -DCMAKE_BUILD_TYPE=Release \
    -DENABLE_CLI=OFF \
    -DENABLE_SHARED=OFF \
    -DSTATIC_LINK_CRT=ON
  cmake --build build_dir
  cmake --install build_dir
  
  cd $srcdir/build-deps/ffmpeg_sources/nv-codec-headers
  sed -i 's/PREFIX =/PREFIX ?=/g' Makefile
  PREFIX=$_stage_dir make install

  cd $srcdir/build-deps/ffmpeg_sources/ffmpeg
  ./configure \
    --disable-all \
    --disable-autodetect \
    --disable-iconv \
    --enable-amf \
    --enable-cuda \
    --enable-cuda_llvm \
    --enable-encoder=h264_amf,hevc_amf,av1_amf \
    --enable-encoder=h264_nvenc,hevc_nvenc,av1_nvenc \
    --enable-encoder=h264_vaapi,hevc_vaapi,av1_vaapi \
    --enable-encoder=h264_rkmpp,hevc_rkmpp \
    --enable-encoder=h264_v4l2m2m \
    --enable-encoder=libsvtav1 \
    --enable-encoder=libx264,libx265 \
    --enable-libsvtav1 \
    --enable-libx264 \
    --enable-libx265 \
    --enable-vaapi \
    --enable-ffnvcodec \
    --enable-nvenc \
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
    -D FFMPEG_PLATFORM_LIBRARIES="numa -lrockchip_mpp $_stage_dir/lib/libx264.a $_stage_dir/lib/libx265.a $_stage_dir/lib/libSvtAv1Enc.a" \
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

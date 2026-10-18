pkgname=psmoveserviceex-git
pkgver=r2067.f49e85ee
pkgrel=1
pkgdesc="A background service that communicates with the psmove and stores pose and button data (Fork)"
arch=('x86_64')
url="https://github.com/Timocop/PSMoveServiceEx"
license=('Apache-2.0')
depends=('boost-libs' 'libusb' 'opencv' 'webkit2gtk-4.1' 'bluez-libs' 'sdl2')
makedepends=('git' 'cmake' 'clang' 'boost' 'eigen' 'protobuf' 'ccache')
provides=('psmoveservice' 'psmoveserviceex')
conflicts=('psmoveservice' 'psmoveservice-git' 'psmoveserviceex')
source=("${pkgname%-git}::git+${url}.git")
sha256sums=('SKIP')

pkgver() {
  cd "${pkgname%-git}"
  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd "${pkgname%-git}"
  git reset --hard HEAD
  git submodule foreach --recursive git reset --hard HEAD
  git submodule update --init --recursive

  sed -i 's/PSM_RELEASE_VERSION_/PSM_PROTOCOL_VERSION_/g' cmake/Version.cmake
  sed -i 's/set(EIGEN3_VERSION_OK FALSE)/set(EIGEN3_VERSION_OK TRUE)/g' thirdparty/eigen/cmake/FindEigen3.cmake
  sed -i -E 's/set\(Boost_USE_STATIC_(LIBS|RUNTIME)[ \t]+ON\)/set(Boost_USE_STATIC_\1 OFF)/g' cmake/ThirdParty.cmake
  sed -i 's/set(CMAKE_CXX_STANDARD 14)/set(CMAKE_CXX_STANDARD 17)/g' cmake/Environment.cmake
  sed -i 's/set(CMAKE_CXX_FLAGS "-std=c++17")/set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -DBOOST_BIND_GLOBAL_PLACEHOLDERS")/g' cmake/Environment.cmake
  sed -i 's/-stdlib=libc++//g' cmake/Environment.cmake
  sed -i '/add_library(PSMoveProtocol/a target_compile_features(PSMoveProtocol PUBLIC cxx_std_17)' src/psmoveprotocol/CMakeLists.txt
  sed -i '/add_executable(PSMoveService/a target_include_directories(PSMoveService PRIVATE /usr/include/opencv4)' src/psmoveservice/CMakeLists.txt
  sed -i '/add_executable(PSMoveConfigTool/a target_include_directories(PSMoveConfigTool PRIVATE /usr/include/opencv4)' src/psmoveconfigtool/CMakeLists.txt

  find . -name "CMakeLists.txt" -exec sed -i -E 's/cmake_minimum_required\(VERSION [0-9]+\.[0-9]+(\.[0-9]+)?/cmake_minimum_required(VERSION 3.5/g' {} +
  find . -name "CMakeLists.txt" -exec sed -i -E 's/set\(CMAKE_CXX_STANDARD [0-9]+\)/set(CMAKE_CXX_STANDARD 17)/g' {} +
  find . -name "CMakeLists.txt" -exec sed -i 's/-std=c++11/-std=c++17/g; s/-std=c++14/-std=c++17/g' {} +
  find . -type f \( -name "CMakeLists.txt" -o -name "*.cmake" \) -exec sed -i -E 's/-std=c\+\+1[14]/-std=c++17/g; s/-stdlib=libc\+\+//g' {} +

  : > src/tests/CMakeLists.txt

  sed -i 's/IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin"/IF (${CMAKE_SYSTEM_NAME} MATCHES "Linux" OR ${CMAKE_SYSTEM_NAME} MATCHES "Darwin"/' cmake/ThirdParty.cmake
  sed -i '/libusb_claim_interface(handle_, 0)/i \    libusb_set_auto_detach_kernel_driver(handle_, 1);' thirdparty/PS3EYEDriver/src/ps3eye.cpp
  sed -i '/libusb_claim_interface(libusb_device_state->device_handle/i \            libusb_set_auto_detach_kernel_driver(libusb_device_state->device_handle, 1);' src/psmoveservice/Device/USB/LibUSBApi.cpp

  find thirdparty/glm -type f \( -name "*.inl" -o -name "*.hpp" \) -exec sed -i 's/\bregister //g' {} +

  find thirdparty/Boost.Application -type f -exec sed -E -i \
    -e 's|<boost/asio/io_service.hpp>|<boost/asio/io_context.hpp>|g' \
    -e 's/\bio_service\b/io_context/g' \
    -e 's/\b_([1-9])\b/boost::placeholders::_\1/g' {} +

  sed -i 's/class io_service;/class io_context; using io_service = io_context;/g' src/psmoveservice/Server/ServerNetworkManager.h
  sed -i '1s|^|#include <boost/asio.hpp>\n|' src/psmoveservice/Server/PSMoveService.cpp
  sed -i 's/asio::mutable_buffers_1 buf/auto buf/g' src/psmoveservice/Server/ServerNetworkManager.cpp
  sed -i -E 's/m_tcp_acceptor\.(get_io_service\(\)|get_executor\(\))/m_io_service/g' src/psmoveservice/Server/ServerNetworkManager.cpp

  sed -i '1s|^|#include <boost/asio.hpp>\nnamespace boost { namespace asio { using io_service = io_context; } }\n|' src/psmoveclient/ClientNetworkManager.cpp
  sed -i 's/resolver\.resolve(tcp::resolver::query(tcp::v4(), m_server_host, m_server_port))/resolver.resolve(tcp::v4(), m_server_host, m_server_port).begin()/g' src/psmoveclient/ClientNetworkManager.cpp
  sed -i 's/tcp::resolver::iterator/tcp::resolver::results_type::iterator/g' src/psmoveclient/ClientNetworkManager.cpp
  sed -i 's/asio::mutable_buffers_1 buffer =/auto buffer =/g' src/psmoveclient/ClientNetworkManager.cpp

  sed -i 's/Gamepad_init(gamepad_api_xinput_only);/Gamepad_init();/g' src/psmoveservice/Device/Manager/ControllerManager.cpp
  local win32_shim='#ifndef _WIN32_COMPAT_DEFS\n#define _WIN32_COMPAT_DEFS\ntypedef void* HANDLE;\n#define INVALID_HANDLE_VALUE ((void*)-1)\ninline void CloseHandle(HANDLE) {}\n#endif'
  sed -i "1s|^|${win32_shim}\n|" src/psmoveservice/Filter/ExternalOrientationFilter.h
  sed -i "1s|^|${win32_shim}\n|" src/psmoveservice/Filter/PositionExternalAttachment.h
  sed -i 's/m_orientation_filter = new OrientationFilterExternal;/m_orientation_filter = nullptr;/g' src/psmoveservice/Filter/CompoundPoseFilter.cpp
  sed -i 's/m_position_filter = new PositionFilterExternalAttachment;/m_position_filter = nullptr;/g' src/psmoveservice/Filter/CompoundPoseFilter.cpp
  sed -i -E 's/(static )?const int (DeviceManagerConfig::)?CONFIG_VERSION\s*=/static constexpr int CONFIG_VERSION =/g' src/psmoveservice/Device/Manager/DeviceManager.cpp

  find src -type f \( -name "*.cpp" -o -name "*.h" \) -exec sed -i \
    -e 's|opencv2/calib3d/calib3d.hpp|opencv2/calib3d.hpp|g' \
    -e 's/\bCvScalar\b/cv::Scalar/g' \
    -e 's/\bCV_RETR_EXTERNAL\b/cv::RETR_EXTERNAL/g' \
    -e 's/\bCV_CHAIN_APPROX_SIMPLE\b/cv::CHAIN_APPROX_SIMPLE/g' \
    -e 's/\bCV_BGR2GRAY\b/cv::COLOR_BGR2GRAY/g' \
    -e 's/\bCV_GRAY2BGR\b/cv::COLOR_GRAY2BGR/g' \
    -e 's/\bCV_BGR2HSV\b/cv::COLOR_BGR2HSV/g' \
    -e 's/\bCV_RGB2HSV\b/cv::COLOR_RGB2HSV/g' \
    -e 's/\bCV_CAP_PROP_/cv::CAP_PROP_/g' {} +

  find src/psmoveservice -type f -exec sed -i 's|<opencv2/videoio/videoio_c.h>|<opencv2/videoio.hpp>|g' {} +
  sed -i '1s|^|namespace cv { enum { CAP_MODE_BGR = 0, CAP_MODE_RGB = 1, CAP_MODE_GRAY = 2, CAP_MODE_YUYV = 3 }; }\n|' src/psmoveservice/PSMoveTracker/PS3EyeTracker.cpp

  sed -i 's/bool open(int index) override;/bool open(int index);/g' src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.h
  sed -i '1s|^|#include <opencv2/opencv.hpp>\n#include <opencv2/videoio.hpp>\nusing namespace cv;\n#ifndef _WIN32\nstatic inline bool usingCLEyeDriver() { return false; }\n#endif\n|' src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.cpp
  sed -i 's/CV_BayerGB2BGR/cv::COLOR_BayerGB2BGR/g' src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.cpp
  sed -i 's/bool success = eye->getFrame(m_MatBayer.data, m_waitFrame);/eye->getFrame(m_MatBayer.data); bool success = true;/g' src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.cpp

  local cv_custom_props='namespace cv { enum { CAP_PROP_FRAMEAVAILABLE = -999, CAP_PROP_WAITFRAME = -998, CAP_PROP_MAXFAILPOLL = -997 }; }'
  sed -i "1s|^|${cv_custom_props}\n|" src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.h
  sed -i 's/cv::CAP_PROP_/CAP_PROP_/g' src/psmoveservice/PSMoveTracker/PSEye/PSEyeVideoCapture.h

  find src/psmoveconfigtool -type f -name "*.h" -exec sed -i '1s|^|#include <string>\n|' {} +
  find src/psmoveconfigtool -type f -name "*.cpp" -exec sed -i '1s|^|#include <opencv2/calib3d.hpp>\n|' {} +
  sed -i 's/AppStage_ColorCalibration:://g' src/psmoveconfigtool/AppStage_ColorCalibration.h
  sed -i 's/AppSubStage_CalibrateWithMat::k_paper_formats_names/k_paper_formats_names/g' src/psmoveconfigtool/AppSubStage_CalibrateWithMat.h
  sed -i '1s|^|#ifndef _WIN32\nstruct { void OpenConfigInExplorer() {} } configExec;\n#endif\n|' src/psmoveconfigtool/AppStage_AdvancedSettings.cpp
  sed -i -E 's/(target_link_libraries\s*\(\s*PSMoveConfigTool\b[^)]*)/\1 boost_filesystem/' src/psmoveconfigtool/CMakeLists.txt

  cat << 'EOF' >> thirdparty/imgui/imgui.h
namespace ImGui {
    inline bool ButtonChecked(const char* label, bool checked, const ImVec2& size = ImVec2(0, 0)) {
        if (checked) {
            PushStyleColor(ImGuiCol_Button, GetStyle().Colors[ImGuiCol_ButtonActive]);
        }
        bool clicked = Button(label, size);
        if (checked) {
            PopStyleColor();
        }
        return clicked;
    }
}
EOF
}

build() {
  export CC=clang
  export CXX=clang++
  export CXXFLAGS="${CXXFLAGS} -std=c++17 -Wno-register -DBOOST_BIND_GLOBAL_PLACEHOLDERS"

  cmake -B build -S "${pkgname%-git}" \
    -DCMAKE_C_COMPILER_LAUNCHER=ccache \
    -DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_CXX_STANDARD_REQUIRED=ON
  cmake --build build
}

package() {
  DESTDIR="${pkgdir}" cmake --install build

  install -Dm755 build/src/psmoveservice/PSMoveService "${pkgdir}/usr/bin/PSMoveService"
  install -Dm755 build/src/psmoveconfigtool/PSMoveConfigTool "${pkgdir}/usr/lib/psmoveservice/PSMoveConfigTool"

  install -d "${pkgdir}/usr/share/psmoveconfigtool/assets"
  cp -a "${pkgname%-git}/src/psmoveconfigtool/assets/." "${pkgdir}/usr/share/psmoveconfigtool/assets/"

  cat << 'EOF' > "${pkgdir}/usr/bin/PSMoveConfigTool"
#!/bin/sh
cd /usr/share/psmoveconfigtool
exec /usr/lib/psmoveservice/PSMoveConfigTool "$@"
EOF
  chmod 755 "${pkgdir}/usr/bin/PSMoveConfigTool"

  if [ -d "${pkgname%-git}/misc/linux/udev" ]; then
    install -Dm644 "${pkgname%-git}/misc/linux/udev/"*.rules -t "${pkgdir}/usr/lib/udev/rules.d/"
  fi
}

# mediasoup@3 iOS WebRTC Framework Build Instructions

**This can only be done on MacOS**

### Get depot tools

Follow the below guide to install depot_tools: 

 https://webrtc.org/native-code/development/prerequisite-sw/ 

---

## Get the WebRTC iOS code

```bash
mkdir webrtc-ios
cd webrtc-ios
fetch --nohooks webrtc_ios
gclient sync
cd src
git checkout -b m74 refs/remotes/branch-heads/m74
gclient sync
```

---

## Build the iOS WebRTC Universal Framework

```bash
# Without bitcode
python tools_webrtc/ios/build_ios_libs.py --extra-gn-args='is_component_build=false rtc_include_tests=false rtc_enable_protobuf=false use_rtti=true use_custom_libcxx=false'

# With bitcode
python tools_webrtc/ios/build_ios_libs.py --bitcode --extra-gn-args='is_component_build=false rtc_include_tests=false rtc_enable_protobuf=false use_rtti=true use_custom_libcxx=false'

# Build the libwebrtc static libraries (64 bit)
cd out_ios_libs
ninja -C arm64_libs/ webrtc
ninja -C x64_libs/ webrtc

# Create a FAT libwebrtc static library
lipo -create arm64_libs/obj/libwebrtc.a x64_libs/obj/libwebrtc.a -output universal/libwebrtc.a
# Check with lipo -info universal/libwebrtc.a
```

---

## Move the output to the WebRTC dependencies folder

```bash
mv [out_ios_libs]directory [XCode project]/mediasoup-client-ios/dependencies/webrtc/src/
```

---

## Build libmediasoupclient

```bash
cd [XCode project]/mediasoup-client-ios/dependencies

# Build iOS arm64
cmake . -Bbuild -DLIBWEBRTC_INCLUDE_PATH=/[XCode project]/mediasoup-client-ios/dependencies/webrtc/src -DLIBWEBRTC_BINARY_PATH=/[XCode project]/mediasoup-client-ios/webrtc/src/out_ios_libs/universal -DMEDIASOUP_LOG_TRACE=ON -DMEDIASOUP_LOG_DEV=ON -DCMAKE_CXX_FLAGS="-fvisibility=hidden" -DLIBSDPTRANSFORM_BUILD_TESTS=OFF -DIOS_SDK=iphone -DIOS_ARCHS="arm64"

make -C build

# Build x86_64 simulator
cmake . -Bbuild_86_64 -DLIBWEBRTC_INCLUDE_PATH=/[XCode project]/mediasoup-client-ios/dependencies/webrtc/src -DLIBWEBRTC_BINARY_PATH=/[XCode project]/mediasoup-client-ios/webrtc/src/out_ios_libs/universal -DMEDIASOUP_LOG_TRACE=ON -DMEDIASOUP_LOG_DEV=ON -DCMAKE_CXX_FLAGS="-fvisibility=hidden" -DLIBSDPTRANSFORM_BUILD_TESTS=OFF -DIOS_SDK=iphonesimulator -DIOS_ARCHS="x86_64"

make -C build_86_64


# Create a FAT libmediasoup/libsdptransform library
lipo -create build/libmediasoup/libmediasoup.a build_86_64/libmediasoup/libmediasoup.a -output lib/
lipo -create build/libmediasoup/libsdptransform/libsdptransform.a build_86_64/libmediasoup/libsdptransform/libsdptransform -output lib/
```

Once build include the libwebrtc.a, libmediasoup.a, libsdptransform.a in the project


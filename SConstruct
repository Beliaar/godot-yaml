# Build worker image (VM template)

clone_depth: 5

version: '{build}'

platform:
  - x86
  - x64

configuration:
  - release
  - debug

environment:
  matrix:
    #- 
      #APPVEYOR_BUILD_WORKER_IMAGE: Visual Studio 2017
      #os: windows
      #MSBUILD_FLAGS: /verbosity:minimal /maxcpucount
      #generator: "Visual Studio 15"
      #generator_x64: "Visual Studio 15 Win64"
      #PYTHON: "C:\\Python27"
      #PYTHON_VERSION: "2.7" 
      #PATH: "%PYTHON%;%PYTHON%\\bin;%PYTHON%\\Scripts;%PATH%"
    - 
      APPVEYOR_BUILD_WORKER_IMAGE: ubuntu
      os: linux
      generator_x64: "Unix Makefiles"
     

matrix:
  fast_finish: true
  exclude:
    - APPVEYOR_BUILD_WORKER_IMAGE: ubuntu
      platform: x86

# scripts that are called at very beginning, before repo cloning
init:
  - ps: echo ${env:os}
  - ps: iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/appveyor/ci/master/scripts/enable-rdp.ps1'))
  - git config --global core.autocrlf input
  - cmake --version
  # Set "build version number" to "short-commit-hash" or when tagged to "tag name" (Travis style)
  - ps: >-
      if (${env:APPVEYOR_REPO_TAG} -eq "true")
      {
        Update-AppveyorBuild -Version "${env:APPVEYOR_REPO_TAG_NAME}"
      }
      else
      {
        Update-AppveyorBuild -Version "dev-$(${env:APPVEYOR_REPO_COMMIT}.substring(0,7))"
      }
  - ps: >-
      if (${env:PLATFORM} -eq "x86")
      {
        ${env:cmake_generator}=${env:generator}
        ${env:bits}=32
      }
      elseif (${env:PLATFORM} -eq "x64")
      {
        ${env:cmake_generator}=${env:generator_x64}
        ${env:bits}=64
      }
      else
      {
        throw "Platform ${env:platform} unsupported"
      }
  - ps: >-
        if (${env:cmake_generator} -eq "Visual Studio 15")
        {
            ${env:COMPILER}="MSVC15"
        }
        elseif (${env:cmake_generator} -eq "Unix Makefiles")
        {
            ${env:COMPILER}="GCC"
        }
        else
        {
            throw "Generator ${env:cmake_generator} unsupported"
        }
install:
  - git submodule update --init --recursive
  - cmd: easy_install scons
  - sh: sudo apt-get install scons

build_script:
  - cd ..
  - git clone https://github.com/jbeder/yaml-cpp.git
  - cd yaml-cpp
  - mkdir build
  - cd build
  - ps: >- 
        cmake ..\ 
        -G ${env:cmake_generator}
        -DBUILD_SHARED_LIBS=ON
        -DCMAKE_INSTALL_PREFIX="${env:APPVEYOR_BUILD_FOLDER}/yaml-cpp" 
        -DYAML_CPP_BUILD_TESTS="OFF"
        -DYAML_CPP_BUILD_TOOLS="OFF"
        -DCMAKE_BUILD_TYPE=${env:configuration}
  #- ps: $blockRdp = $true; iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/appveyor/ci/master/scripts/enable-rdp.ps1'))
  # build
  - cmd: cmake --build . --target ALL_BUILD --config ${env:configuration} -- /logger:"C:\Program Files\AppVeyor\BuildAgent\Appveyor.MSBuildLogger.dll"
  - sh: make
  # install
  - cmd: cmake --build . --target INSTALL --config ${env:configuration} -- /logger:"C:\Program Files\AppVeyor\BuildAgent\Appveyor.MSBuildLogger.dll"
  - sh: make install
  - cd ../..
  - cd godot-yaml/godot-cpp
  - dir
  - ps: scons platform=${env:os} arch=${env:bits} headers=godot_headers generate_bindings=yes target=${env:configuration}
  - cd ..
  - ps: scons platform=${env:os} target=${env:configuration} bits=${env:bits}

after_build:

  # package dependencies artifact
  - cd demo/bin
  - ps: echo "${env:APPVEYOR_BUILD_FOLDER}/godot-yaml-${env:APPVEYOR_BUILD_VERSION}-${env:COMPILER}-${env:PLATFORM}-${env:configuration}.7z"
  - ps: 7z a -tzip -mx9 "${env:APPVEYOR_BUILD_FOLDER}/godot-yaml-${env:APPVEYOR_BUILD_VERSION}-${env:COMPILER}-${env:PLATFORM}-${env:configuration}.7z"
  - ps: appveyor PushArtifact "${env:APPVEYOR_BUILD_FOLDER}/godot-yaml-${env:APPVEYOR_BUILD_VERSION}-${env:COMPILER}-${env:PLATFORM}-${env:configuration}.7z"

artifacts:
  - path: 'godot-yaml-*.7z'
    name: GodotYaml

# deploy to Github Releases on tag push
deploy:
  provider: GitHub
  release: 'GodotYaml $(APPVEYOR_REPO_TAG_NAME)'
  tag: $(APPVEYOR_REPO_TAG_NAME)
  artifact: GodotYaml
  draft: false
  prerelease: false
  force_update: true               # overwrite files of existing release on GitHub
  on:
    branch: master                 # release from master branch only
    appveyor_repo_tag: true        # deploy on tag push only
  auth_token:                      # encrypted token from GitHub
    secure: 4dm7uTzjL+0fOu5k6huo2PbcGhxj0e7RbXHtkdjiQ1maMLunSup7bdXP4+L58wDV

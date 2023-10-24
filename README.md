# CMake 사용법

[참조 자료 1](https://www.tuwlab.com/ece/27234)

[참조 자료 2](https://www.tuwlab.com/ece/27260)

[참조 자료 3](https://www.tuwlab.com/ece/27270)

CMake를 사용하면 의존성 정보를 일일이 기술해 주지 않아도 되므로 빌드 스크립트의 관리 측면에서 매우 효율적입니다. 프로젝트를 처음 시작할 때 Build Step만 잘 구성해 놓으면, 이후에는 소스 파일(*.c)을 처음 추가할 때만 CMakeLists.txt 파일을 열어서 등록해 주면 됩니다. 이후에는 소스코드를 어떻게 수정하더라도 빌드에서 제외하지 않는 한 스크립트를 수정하지 않아도 됩니다.

CMake도 Make와 마찬가지로 의존성 검사를 해서 Incremental Build를 수행하지만, 가장 큰 차이점은 CMake는 소스파일 내부까지 들여다보고 분석해서 의존성 정보를 스스로 파악한다는 점입니다. 예를 들어, 소스파일에 헤더파일을 추가(#include)하면, 직후 빌드부터 의존성 관계 변화가 자동으로 추적되어 헤더 파일의 변화까지 추적하기 시작합니다.

또한, Makefile에서는 빌드 중간생성물인 Object파일들의 이름과 의존성 정보까지 모두 기술해 줘야 하지만, CMake에서는 그럴 필요가 전혀 없습니다. 뒤에서 살펴보겠지만, CMake의 빌드 스크립트인 CMakeLists.txt에서는 최종 빌드 결과물과 이를 빌드하기 위한 소스 파일들만 명시해 주면 그것으로 끝입니다. (여기서 최종 빌드 결과물은 실행 바이너리나 라이브러리가 됩니다.)

![빌드 예제](https://github.com/Soonbum/how_to_CMake/assets/16474083/a30b29eb-4c51-43e8-9091-d360e31c45b5)

3개의 소스파일로부터 하나의 실행파일을 만드는 예제이며, main.c에서는 foo.c와 bar.c에서 정의된 함수들을 호출하는 의존성이 존재합니다.

```
# Makefile
OBJS=main.o foo.o bar.o
TARGET=app.out
 
all: $(TARGET)
  
clean:
    rm -f *.o
    rm -f $(TARGET)
 
$(TARGET): $(OBJS)
    $(CC) -o $@ $(OBJS)
  
main.o: foo.h bar.h main.c
foo.o: foo.h foo.c
bar.o: bar.h bar.c
```

위와 같은 기능을 하는 CMake 빌드 스크립트인 CMakeLists.txt 파일은 다음과 같습니다.

```
ADD_EXECUTABLE( app.out main.c foo.c bar.c )
```

위와 같이 작성하고 Makefile을 생성하려면 다음 명령을 실행합니다.

```
cmake CMakeLists.txt
```

'cmake CMakeLists.txt' 명령은 자동 생성된 Makefile을 삭제하지 않는 한 최초 한 번만 실행해 주면 됩니다. 생성된 Makefile을 실행할 때 CMakeLists.txt 파일의 변경 여부를 검사해서 필요한 경우 Makefile을 자동으로 재생성해 줍니다.

위의 예제에서 살펴봤듯이, CMake 빌드 스크립트에서 빌드 대상 바이너리 정의 구문(ADD_EXECUTABLE)에 들어가는 내용은 출력 바이너리 이름과 소스 파일 목록이 전부입니다. 그 외의 소스 파일에 포함되는 헤더 파일들과 각 소스 파일을 컴파일한 Object파일들은 명시할 필요가 없으며, CMake 내부적으로 알아서 처리됩니다.

다음 그림에서 흐리게 표시한 부분이 CMake 내부적으로 처리되어서 빌드 스크립트에 명시하지 않아도 되는 부분입니다.

![image](https://github.com/Soonbum/how_to_CMake/assets/16474083/64510804-1cd5-4d59-9b93-4981b7badf7a)

헤더 파일(*.h)은 직접적인 빌드 대상은 아니지만 소스 파일에 포함되어서 해당 Object 파일과 내재적 의존 관계를 만듭니다. CMake는 각 Object파일을 생성하기 전에 소스 파일을 분석하여 어떤 헤더 파일들이 포함되어 있는지 파악하고, 이들 헤더 파일의 변경 여부를 검사하여 필요시 다시 컴파일을 수행합니다.

Object 파일(*.o)은 신경쓰지 않아도 되지만, CMake 내부적으로는 빌드를 수행할 때 자동 생성되는 CMakeFiles 디렉토리 안에 생성됩니다. 빌드 후 지저분하게 소스 파일과 Object 파일이 섞여 있지 않으므로 프로젝트 디렉토리를 깔끔하게 유지할 수 있습니다.

CMake를 실행하면 다음 파일과 디렉토리가 자동 생성되는데 깃으로 버전 관리를 할 경우, .gitignore 파일에 다음 내용을 추가하면 됩니다.

```
/CMakeCache.txt
/cmake_install.cmake
/CMakeFiles/
/Makefile
```

make 명령으로 CMake로 생성한 Makefile을 실행하면 가장 먼저 CMakeLists.txt파일이 변경됬는지 여부를 검사하고, 변경된 경우 Makefile을 다시 생성하여 실행합니다.

다음으로 Makefile에 정의된 각 Target별로 빌드를 수행하는데, 이 때 내부 Build Step에 따라 cmake명령으로 각 Target을 빌드하는 데 필요한 Sub-Makefile을 생성합니다. 이 때 생성되는 Sub-Makefile들도 역시 CMakeFiles 디렉토리 내부에 저장됩니다. 자동 생성되는 Sub-Makefile들도 역시 의존성 검사를 통해 이전에 만들어 뒀던 것을 재활용하거나, 다시 생성합니다.

실제로 CMake로 자동 생성된 Makefile을 뜯어보면, Target 빌드 Recipe에 다음과 같이 Sub-Makefile을 make 명령으로 호출하는 구문 하나만 달랑 써져 있음을 알 수 있습니다.

```
main.c.o:
$(MAKE) -f CMakeFiles/app.out.dir/build.make CMakeFiles/app.out.dir/main.c.o
```

## CMakeLists.txt 작성하는 방법

여기에서 다루지 않은 구문들은 [CMake 2.8.12 문서](https://cmake.org/cmake/help/v2.8.12/cmake.html)를 참조하십시오.

### 변수 정의

* 단일 변수 정의: `SET(<변수명> <값>)`
  - <값>에 공백이 포함되어 있을 경우 ""로 감싸주면 됩니다.

* 목록 변수 정의: `SET (<목록변수명> <항목> <항목> <항목> ...)`
  - <항목>들은 공백문자로 구분합니다.
  - <항목> 값에 공백이 포함되어 있을 경우 ""로 감싸주면 됩니다.

* 변수 참조: `$<변수명>`, `${<변수명>}`

## 프로젝트 전반 관련

* 필요 CMake 최소 버전 명시: `CMAKE_MINIMUM_REQUIRED (VERSION <버전>)`
  - CMake 빌드 스크립트를 실행하기 위한 최소 버전을 명시합니다.
 
* 프로젝트 이름 설정: `PROJECT (<프로젝트명>)`
  - 프로젝트 이름에 공백이 포함되어 있을 경우 ""로 감싸주면 됩니다.
  - 가급적 공백을 포함하지 않는 편이 좋습니다.

* 프로젝트 이름: `CMAKE_PROJECT_NAME`
  - PROJECT() 명령으로 설정한 프로젝트 이름이 이 변수에 저장됩니다.

* 빌드 형상(Configuration): `CMAKE_BUILD_TYPE`
  - 빌드 형상이란 빌드 목적에 따라 서로 다른 옵션을 지정해서 빌드하는 것입니다. 기본적으로 다음 4가지 빌드 형상을 지원하며, 필요한 경우 사용자 정의 빌드 형상을 정의할 수도 있습니다.
  - Debug: 디버깅 목적의 빌드
  - Release: 배포 목적의 빌드
  - RelWithDebInfo: 배포 목적의 빌드이지만, 디버깅 정보 포함
  - MinSizeRel: 최소 크기로 최적화한 배포 목적 빌드

* 콘솔에 메시지 출력: `MESSAGE ([<Type>] <메시지>)`
  - Type은 다음 중 하나이며 생략할 수 있습니다.
  - STATUS: 상태 메시지 출력 (메시지 앞에 "--"가 추가되서 출력됨)
  - WARNING: 경고 메시지를 출력하고, 계속 진행
  - AUTHOR_WARNING: 프로젝트 개발자용 경고 메시지를 출력하고, 계속 진행
  - SEND_ERROR: 오류 메시지를 출력하고 계속 진행하지만, Makefile 생성은 하지 않음
  - FATAL_ERROR: 오류 메시지를 출력하고, 작업을 즉시 중단

* Verbose Makefile 작성 여부: `CMAKE_VERBOSE_MAKEFILE`
  - True 또는 1로 지정하면 빌드 상세 과정을 모두 출력하는 Makefile을 생성합니다.
  - PROJECT() 명령을 만나면 false로 초기화되므로 반드시 이보다 뒤에 있어야 합니다.

## 빌드 대상(Target) 관련

* 빌드 대상 바이너리 추가: `ADD_EXECUTABLE (<실행파일명> <소스파일> <소스파일> ...)`
  - 빌드 최종 결과물로 생성할 실행 파일을 추가합니다.
  - <실행파일명>: 생성할 바이너리의 파일명
  - <소스파일>: 실행 파일을 생성하는 데 필요한 소스파일

* 빌드 대상 라이브러리 추가: `ADD_LIBRARY (<라이브러리이름> [STATIC|SHARED|MODULE] <소스파일> <소스파일> ...)`
  - 빌드 최종 결과물로 생성할 라이브러리를 추가합니다.
  - <라이브러리이름>: 생성할 라이브러리 이름 (lib~.a / lib~.so에서 ~에 들어갈 값)
  - [STATIC|SHARED|MODULE]: 라이브러리 종류 (생략시 STATIC)
  - <소스파일>: 라이브러리를 생성하는 데 필요한 소스파일

* 사용자 정의 Target 추가: `ADD_CUSTOM_TARGET (...)`
  - <이름>: Target 이름
  - [ALL]: make (또는 make all) 명령에서 기본 빌드 대상에 포함할지 여부
  - <출력메시지>: 명령 실행 전에 콘솔에 출력할 메시지
  - <의존대상목록>: 이 Target이 의존하는 대상 목록 (공백으로 구분)
  - <작업디렉토리>: 명령을 실행할 위치(pwd)
  - <명령>: Target을 생성하기 위한 명령(Recipe)
  - <VERBATIM>: <명령>을 Escape하지 않고 그대로 사용하려는 경우 추가 (변수, 공백, 따옴표 등)
  ```
  ADD_CUSTOM_TARGET (<이름> [ALL]
                   [COMMENT <출력메시지>]
                   [DEPENDS <의존대상목록>]
                   [WORKING_DIRECTORY <작업디렉토리>]
                   COMMAND <명령>
                   [COMMAND <명령>]
                   [VERTBATIM]
                   ...)
  ```

* Target간 의존성 정의: `ADD_DEPENDENCIES (<Target이름> <의존대상> <의존대상> ...)`
  - Top-level Target간의 의존성을 지정합니다. (Top-level Target이란 ADD_EXECUTABLE, ADD_LIBRARY, ADD_CUSTOM_TARGET 명령으로 정의한 Target들을 의미함)
  - Target을 빌드할 때 이 명령으로 정의한 의존 대상들이 'Outdated'인 경우 이들에 대한 빌드를 먼저 수행합니다.

* 설치 매크로 (make install) 정의: `INSTALL (...)`
  - make install 명령을 실행했을 때 무슨 동작을 수행할지를 지정합니다.
  - 설치 경로가 모두 같은 경우, 다음과 같이 축약된 형태로도 사용할 수 있습니다. `INSTALL (TARGETS <Target목록> DESTINATION <설치경로>)`
  - 이 명령은 상당히 많은 옵션을 제외하고 설명하였습니다. 자세한 내용은 [공식 매뉴얼](https://cmake.org/cmake/help/v2.8.12/cmake.html#command:install)을 참조하십시오.
```
INSTALL (TARGETS <Target목록>
        RUNTIME DESTINATION <바이너리_설치경로>
        LIBRARY DESTINATION <라이브러리_설치경로>
        ARCHIVE DESTINATION <아카이브_설치경로>)
```

* 설치 디렉토리: `CMAKE_INSTALL_PREFIX`
  - 설치 매크로(make install)에서 실행 바이너리와 라이브러리 등의 최종 생성물을 복사할 설치 디렉토리를 지정합니다.
  - INSTALL() 명령에서 상대 경로를 사용한 경우, 이 변수에 지정한 디렉토리가 Base 디렉토리가 됩니다.
  - 이 변수를 별도로 지정하지 않으면 기본값은 /usr/local 입니다.

## 전역 빌드 설정 관련

모든 Target에 공통으로 적용되는 빌드 옵션을 지정하는 변수와 명령들입니다. 유의할 점은 이 전역 빌드 설정 명령들은 선언 이후에 정의되는 Target들에게만 적용되기 때문에 CMakeLists.txt의 상단에 위치해야 합니다.

* C 컴파일러: `CMAKE_C_COMPILER`
  - 컴파일 및 링크 과정에서 사용할 컴파일러의 경로를 지정합니다.

* 컴파일 옵션 추가: `ADD_COMPILE_OPTIONS (<옵션> <옵션> ...)`
  - 소스파일을 컴파일하여 오브젝트 파일을 생성할 때 컴파일러에 전달할 옵션(플래그)을 추가합니다.

* 빌드 형상별 컴파일 옵션: `${CMAKE_C_FLAGS_<빌드형상>}`
  - 특정 빌드 형상에서만 사용할 컴파일 옵션(플래그)를 지정합니다.
  - 주의: 옵션이 여러개인 경우 목록으로 정의하지 말고, 위의 예시와 같이 "" 로 둘러서 하나의 문자열로 정의해야 합니다. 그렇지 않으면 각 옵션 항목들이 공백 문자가 아닌 세미콜론으로 구분되어 컴파일 명령에 입력되므로 오류가 발생합니다.

* 전처리기 매크로 추가 (-D): `ADD_DEFINITIONS (-D<매크로> -D<매크로> -D<매크로>=값 ...)`
  - 전처리기에 전달할 매크로를 정의합니다.
  - 컴파일러 옵션 중 -D에 해당합니다.

* 헤더 디렉토리 추가 (-I): `INCLUDE_DIRECTORIES (<디렉토리> <디렉토리> ...)`
  - 각 소스 파일에서 #include 구문으로 포함시킨 헤더 파일을 찾을 디렉토리 목록을 추가합니다.
  - 컴파일러 옵션 중 -I에 해당합니다.

* 라이브러리 디렉토리 지정 (-L): `LINK_DIRECTORIES (<디렉토리> <디렉토리> ...)`
  - 링크 과정에서 필요한 라이브러리 파일들을 찾을 디렉토리 목록을 지정합니다.
  - 컴파일러 옵션 중 -L에 해당합니다.

* 링크 옵션 추가: `LINK_LIBRARIES (<라이브러리> <라이브러리> ...)`
  - 링크시 포함할 라이브러리 목록을 지정합니다.
  - 라이브러리 파일명의 Prefix/Postfix는 제외하고 라이브러리 이름만 입력합니다. (예. libxxx.a에서 xxx에 해당하는 부분만 입력)
  - 컴파일러 옵션 중 -l에 해당합니다.
  - 이 명령으로 링크 옵션도 함께 지정할 수 있습니다. <라이브러리> 값이 하이픈('-')으로 시작하는 경우 링크 명령에 그대로 포함되고, 그렇지 않은 경우 앞에 '-l'이 자동으로 추가됩니다.

* 빌드 형상별 링크 옵션: `CMAKE_EXE_LINKER_FLAGS_<빌드형상>`
  - 특정 빌드 형상에서만 사용할 링크 옵션(플래그)를 지정합니다.
  - 옵션이 여러 개인 경우 ""로 감싸주면 됩니다.

* 실행 바이너리 출력 디렉토리: `RUNTIME_OUTPUT_DIRECTORY`
  - 빌드 완료한 실행 바이너리를 저장할 디렉토리를 지정합니다.

* 라이브러리 출력 디렉토리: `LIBRARY_OUTPUT_DIRECTORY`
  - 빌드 완료한 라이브러리를 저장할 디렉토리를 지정합니다.

* 아카이브 출력 디렉토리
  - 빌드 완료한 아카이브(Static 라이브러리)를 저장할 디렉토리를 지정합니다.

## 특정 대상(Target) 한정 빌드 설정 관련

* Target 컴파일 옵션 추가

* Target 전처리기 매크로 정의 (-D)

* Target 헤더 디렉토리 추가 (-I)

* Target 링크 옵션 및 라이브러리 지정 (-l)

## 빌드 절차(Step) 관련

* 템플릿 파일로부터 파일 자동 생성

* 사용자 정의 출력 파일 추가

* 빌드 과정 명령 추가

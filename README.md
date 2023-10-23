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

### SET(): 변수 정의

...

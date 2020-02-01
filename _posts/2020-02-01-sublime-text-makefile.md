---
layout: post
title: Sublime Text 与 Makefile 设置
tags:
- software
- cpp
- linux
categories:
- code
comments: true
mathjax: false
date: 2020-02-01 18:35:35 +0800
---
Sublime Text 与 Makefile 设置

## Sublime 设置
Sublime项目下面若有Makefile文件则会自动build调用，但是默认的build仅支持make和clean，没有run的设置。为了添加更多的功能，可以通过先安装`PackageResourceViewer`这个包，然后通过`Ctrl+shift+p`调用这个包的`open resource`选择更改Makefile设置,然后选择`Make.sublime-build`进行编辑，编辑内容参考如下：

```json
{
	"shell_cmd": "make -j2",
	"file_regex": "^(..[^:\n]*):([0-9]+):?([0-9]+)?:? (.*)$",
	"working_dir": "${folder:${project_path:${file_path}}}",
	"selector": "source.makefile",
	"syntax": "Packages/Makefile/Make Output.sublime-syntax",
	"keyfiles": ["Makefile", "makefile"],
	"env":{
		"LD_LIBRARY_PATH":"/usr/local/lib:$LD_LIBRARY_PATH",
	},

	"variants":
	[
		{
			"name": "Run",
			"shell_cmd": "make run"
		},
		{
			"name": "Make & Run",
			"shell_cmd": "make -j2 && echo ------------------------------ && make run"
		},
		{
			"name": "Clean & Make & Run",
			"shell_cmd": "make clean && make -j2 && echo ------------------------------ && make run"
		},
		{
			"name": "Clean",
			"shell_cmd": "make clean"
		}
	]
}

```
如上主要添加了LD_LIBRARY_PATH的环境变量，几个通过`variants`添加的使编译运行合在一起的功能。

## Makefile 模板 
c++项目对应的参考模板如下:
```bash
CC         =   g++
CFLAGS     =   -c -Wall -std=c++17
OBJECTS    =   $(SOURCES:.cpp=.o)
INCLUDES   =   -I/usr/local/include/opencv4
LIBDIRS    =   -L/usr/local/lib
LIBS       =   -lopencv_core -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc
SOURCES    =   main.cpp dog.cpp
EXECUTABLE =   dog
ARGS       =   snoop 3 ./dog.jpg

all: $(SOURCES) $(EXECUTABLE)
    
$(EXECUTABLE): $(OBJECTS) 
	$(CC)  $(OBJECTS) $(LIBDIRS) $(LIBS) -o $@

.cpp.o:
	$(CC) $(CFLAGS) $(INCLUDES) $< -o $@

run:
	./$(EXECUTABLE) ${ARGS}

clean:
	rm *.o ${EXECUTABLE}
```

## 其他说明
+ 以上的介绍主要是基于Linux, Windows下稍有不同，可参考其默认设置更改。
+ 通过ctrl+shift+B选择编辑模式。
+ 若要想在c++文件编辑模式下自动调用makefile，则需要将该项目的文件夹排在左侧文件夹列表的第一位。

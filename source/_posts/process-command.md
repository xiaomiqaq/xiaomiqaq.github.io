---
title: 命令行程序如何处理参数
date: 2023-10-18 14:27:06
tags: 
  - linux
  - C
  - C++
 
---

# 命令行程序如何处理参数

当我们想要写一个命令行程序时，经常面临这个场景

1、想要添加可选参数，部分可选参数后面要跟一个相连的命令，有些可选参数则不需要。

2、可选参数可以随意组合，顺序不限

3、如果没有指定一个路径，则将当前路径作为路径

现在我有一个命令行程序，支持可选参数-s、-S、-f。-s 后面要跟一个数字，-f后面要跟一个字符串加一个数字。



我的代码如下，仅供参考：

```c
typedef struct Options
{
	int listAttributes; //-S
	int filterSize;		//-s
	int filterDepth;	//-f
	char *filterName;	//-f
	char *cmd;
	char *directory;
} Options;
int main(int argc, char **argv)
{
	char path[1000];
	// Process Optional parameters
	Options options = {0};
	options.filterSize = -1;
	options.filterDepth = -1;
	options.filterName = 0;
	for (int i = 1; i < argc; i++)
	{

		if (strcmp(argv[i], "-S") == 0)
		{
			options.listAttributes = 1;
		}
		else if (strcmp(argv[i], "-s") == 0)
		{
			i++;
			options.filterSize = atoi(argv[i]);
			if (options.filterSize <= 0)
				return 0;
		}
		else if (strcmp(argv[i], "-f") == 0)
		{
			i++;
			options.filterName = argv[i];
			i++;
			options.filterDepth = atoi(argv[i]);
		}
		else
		{
			options.directory = argv[i];
		}
	}
	// If there is no directory path in the command, get the current path
	char current_directory[1024];
	if (options.directory == 0)
	{
		if (getcwd(current_directory, sizeof(current_directory)) == NULL)
		{
			return 0;
		}
		options.directory = current_directory;
	}
}
```




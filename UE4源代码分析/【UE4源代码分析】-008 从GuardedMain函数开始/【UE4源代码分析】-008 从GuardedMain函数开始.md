# 【UE4源代码分析】-008 从GuardedMain函数开始游戏循环
&emsp;&emsp;在上一篇《【UE4源代码分析】-005 Editor的起点-Main函数》(https://blog.csdn.net/freehawkzk/article/details/80593687)中，我们知道UE4程序启动运行时是从WinMain函数开始，之后调用`GuardedMain`函数进行处理，待`GuardedMain`退出之后，执行appExit之后程序退出。由此可见，整个游戏的有效处理实际上是发生在`GuardedMain`函数中的。今天，我们来看看在这个函数中间发生了什么。

[TOC]
## 1、处理流程
&emsp;&emsp;`GuardedMain`的代码流程是很清晰的，加上Epic的注释，很容易理解整体框架。
&emsp;&emsp;首先，由于UE4对windows的头文件进行了大量精简，在进入GuardedMain时还有一些运行环境初始化的工作没做，因此，先调用`FCoreDelegates::GetPreMainInitDelegate().Broadcast();`完成初始化工作。
&emsp;&emsp;然后，申明了一个内部类的局部变量，这里主要利用内部类局部变量析构的时候会自动执行析构函数，从而确保`EngineExit();`始终会得到执行。
&emsp;&emsp;接下来，主要工作是执行`int32 ErrorLevel = EnginePreInit( CmdLine );`,也就是进行引擎的预初始化工作。
&emsp;&emsp;之后，根据当前运行的程序是不是Editor，决定调用` EditorInit(GEngineLoop);`或`ErrorLevel = EngineInit();`,当本次运行的不是Editor的时候会执行EngineInit。经过代码跟踪，在EditorInit内部会调用EngineInit，也就是说Editor除了有游戏运行的相关资源要初始化之外还有很多其他的工作需要完成。
&emsp;&emsp;再接下来程序就进入游戏循环了。
```C++
	while( !GIsRequestingExit )
	{
		EngineTick();
	}
```
&emsp;&emsp;当程序从游戏循环中退出的时候，如果是编辑器，需要执行EditorExit。再之后就是return退出函数了。
&emsp;&emsp;等等，按照对称原则，前面执行了EngineInit，后面应该有一个对应的Exit才对啊，难道不是Editor就不用Exit了么？原来，`EngineExit`在前面讲到的局部变量析构的时候会被自动调用，很对称。

```C++
int32 GuardedMain( const TCHAR* CmdLine, HINSTANCE hInInstance, HINSTANCE hPrevInstance, int32 nCmdShow )
{
	// Super early init code. DO NOT MOVE THIS ANYWHERE ELSE!
	FCoreDelegates::GetPreMainInitDelegate().Broadcast();

	// make sure GEngineLoop::Exit() is always called.
	struct EngineLoopCleanupGuard 
	{ 
		~EngineLoopCleanupGuard()
		{
			EngineExit();
		}
	} CleanupGuard;

	// Set up minidump filename. We cannot do this directly inside main as we use an FString that requires 
	// destruction and main uses SEH.
	// These names will be updated as soon as the Filemanager is set up so we can write to the log file.
	// That will also use the user folder for installed builds so we don't write into program files or whatever.
#if PLATFORM_WINDOWS
	FCString::Strcpy(MiniDumpFilenameW, *FString::Printf(TEXT("unreal-v%i-%s.dmp"), FEngineVersion::Current().GetChangelist(), *FDateTime::Now().ToString()));

	const TCHAR* OrgCmdLine = CmdLine;

	CmdLine = FCommandLine::RemoveExeName(OrgCmdLine);

	const TCHAR* CmdOccurence = FCString::Stristr(OrgCmdLine, TEXT("-cmd"));
	GIsConsoleExecutable = CmdOccurence != NULL && CmdOccurence < CmdLine;
	
#endif

	int32 ErrorLevel = EnginePreInit( CmdLine );

	// exit if PreInit failed.
	if ( ErrorLevel != 0 || GIsRequestingExit )
	{
		return ErrorLevel;
	}

	{
		FScopedSlowTask SlowTask(100, NSLOCTEXT("EngineInit", "EngineInit_Loading", "Loading..."));

		// EnginePreInit leaves 20% unused in its slow task.
		// Here we consume 80% immediately so that the percentage value on the splash screen doesn't change from one slow task to the next.
		// (Note, we can't include the call to EnginePreInit in this ScopedSlowTask, because the engine isn't fully initialized at that point)
		SlowTask.EnterProgressFrame(80);

		SlowTask.EnterProgressFrame(20);

#if WITH_EDITOR
		if (GIsEditor)
		{
			ErrorLevel = EditorInit(GEngineLoop);
		}
		else
#endif
		{
			ErrorLevel = EngineInit();
		}
	}

	double EngineInitializationTime = FPlatformTime::Seconds() - GStartTime;
	UE_LOG(LogLoad, Log, TEXT("(Engine Initialization) Total time: %.2f seconds"), EngineInitializationTime);

#if WITH_EDITOR
	UE_LOG(LogLoad, Log, TEXT("(Engine Initialization) Total Blueprint compile time: %.2f seconds"), BlueprintCompileAndLoadTimerData.GetTime());
#endif

	ACCUM_LOADTIME(TEXT("EngineInitialization"), EngineInitializationTime);

	while( !GIsRequestingExit )
	{
		EngineTick();
	}

#if WITH_EDITOR
	if( GIsEditor )
	{
		EditorExit();
	}
#endif
	return ErrorLevel;
}
```
## 2、示意图
```flow
st=>start: 开始
e=>end: 结束
RuntimeInit=>operation: 初始化运行环境
CreateInternalStruct=>operation: 创建内部结构的局部变量
EnginePreInit=>operation: 引擎预初始化
IsEditor=>condition: 当前运行的是否是编辑器
EditorInit=>operation: 编辑器初始化
EngineInit=>operation: 引擎初始化
EngineTick=>condition: 游戏循环是否退出
IsEditor2=>condition: 当前运行的是否是编辑器
EditorExit=>operation: 退出编辑器
Destruction=>operation: 局部变量析构
EngineExit=>operation: 引擎退出

st->RuntimeInit->CreateInternalStruct->EnginePreInit->IsEditor
IsEditor(yes)->EditorInit->EngineTick
IsEditor(no)->EngineInit->EngineTick
EngineTick(yes)->IsEditor2
IsEditor2(yes)->EditorExit->Destruction
IsEditor2(no)->Destruction
Destruction->EngineExit->e
```
#Avalonia #dotnet

按照Avalonia手册用`dotnet new avalonia.app -o MyApp`创建项目时报错：
```bash
The template "Avalonia .NET App" was created successfully.

Processing post-creation actions...
Restoring /home/dbgli/cs_projects/MyApp/MyApp.csproj:
  Determining projects to restore...
/usr/lib/dotnet/sdk/8.0.118/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.TargetFrameworkInference.targets(166,5): error NETSDK1045: The current .NET SDK does not support targeting .NET 9.0.  Either target .NET 8.0 or lower, or use a version of the .NET SDK that supports .NET 9.0. Download the .NET SDK from https://aka.ms/dotnet/download [/home/dbgli/cs_projects/MyApp/MyApp.csproj]
Restore failed.
Post action failed.
Manual instructions: Run 'dotnet restore'
```

当前SDK版本不支持.NET 9.0，肯定啊，我装的是8.0。打开MyApp.csproj，把`<TargetFramework>net9.0</TargetFramework>`这一行中的9.0改为8.0，然后记得先进到项目文件夹里再`dotnet restore`，不然报错：
```bash
MSBUILD : error MSB1003: Specify a project or solution file. The current working directory does not contain a project or solution file.
```

最后`dotnet run`就可以运行程序了。用Rider的话就可以在配置界面选好版本，估计命令行应该也有参数可以配置。
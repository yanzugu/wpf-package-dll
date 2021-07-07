# wpf-package-dll
將程式打包成單一 .exe 檔案

- ## 將下列程式碼新增至 .csproj
若找不到 .csproj 檔，請至 visual studio -> 檔案 -> 開啟 -> 網路 選擇目標資料夾。
```
<Target Name="AfterResolveReferences">
  <ItemGroup>
    <EmbeddedResource Include="@(ReferenceCopyLocalPaths)" Condition="'%(ReferenceCopyLocalPaths.Extension)' == '.dll'">
      <LogicalName>%(ReferenceCopyLocalPaths.DestinationSubDirectory)%(ReferenceCopyLocalPaths.Filename)%(ReferenceCopyLocalPaths.Extension)</LogicalName>
    </EmbeddedResource>
  </ItemGroup>
</Target>
```

- ## 新增 Program.cs 並貼上下列程式碼
並至 專案 -> 屬性 -> 應用程式 -> 啟始物件 選擇 Program
```
[STAThread]
public static void Main()
{
    AppDomain.CurrentDomain.AssemblyResolve += OnResolveAssembly;
    App.Main();
}
```

- ## 在 Program.cs 新增下列方法
```
private static Assembly OnResolveAssembly(object sender, ResolveEventArgs args)
{
    Assembly executingAssembly = Assembly.GetExecutingAssembly();
    AssemblyName assemblyName = new AssemblyName(args.Name);

    var path = assemblyName.Name + ".dll";
    if (assemblyName.CultureInfo.Equals(CultureInfo.InvariantCulture) == false) path = String.Format(@"{0}\{1}", assemblyName.CultureInfo, path);

    using (Stream stream = executingAssembly.GetManifestResourceStream(path))
    {
        if (stream == null) return null;

        var assemblyRawBytes = new byte[stream.Length];
        stream.Read(assemblyRawBytes, 0, assemblyRawBytes.Length);
        return Assembly.Load(assemblyRawBytes);
    }
}
```

- [參考來源 - 方法](https://stackoverflow.com/questions/1025843/merging-dlls-into-a-single-exe-with-wpf)
- [參考來源 - Program.cs](https://www.cnblogs.com/chenxizhang/archive/2010/03/25/1694611.html)
- [參考來源 - csproj](https://blog.csdn.net/brook0344/article/details/6334353)

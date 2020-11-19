---
layout: post
title: "IL2CPP引起的Protobuf代码丢失问题"
date: 2020-11-19
categories: Unity3d IL2CPP Protobuf
tags: 
---

公司的Unity3d项目使用了google的protobuf库, 最近出了个在手机上才出现的一个bug, 出bug的代码如下.

```csharp
public static partial class MSGCLIENTSERVERPLATFORMINFOReflection {

  #region Descriptor
  /// <summary>File descriptor for ClientMsgHead/Login/MSG_CLIENT_SERVER_PLATFORM_INFO.proto</summary>
  public static pbr::FileDescriptor Descriptor {
    get { return descriptor; }
  }
  private static pbr::FileDescriptor descriptor;

  static MSGCLIENTSERVERPLATFORMINFOReflection() {
    byte[] descriptorData = global::System.Convert.FromBase64String(
        string.Concat(
          "CjlDbGllbnRNc2dIZWFkL0xvZ2luL01TR19DTElFTlRfU0VSVkVSX1BMQVRG",
          "T1JNX0lORk8ucHJvdG8aH1Byb3RvRXh0ZW5kL015QXJyYXlMZW5ndGgucHJv",
          "dG8ipwEKH01TR19DTElFTlRfU0VSVkVSX1BMQVRGT1JNX0lORk8SIAoYdTY0",
          "R0lEVGVjaFN0b3JhZ2VTZXJ2aWNlGAEgASgEEjAKDnN6UGxhdGZvcm1OYW1l",
          "GAIgASgMQhiKtRgUUExBVEZPUk1fTkFNRV9MRU5HVEgSMAoOc3pQbGF0Zm9y",
          "bUFyZWEYAyABKAxCGIq1GBRQTEFURk9STV9BUkVBX0xFTkdUSEID+AEBYgZw",
          "cm90bzM="));
    descriptor = pbr::FileDescriptor.FromGeneratedCode(descriptorData,
        new pbr::FileDescriptor[] { global::MyArrayLengthReflection.Descriptor, },
        new pbr::GeneratedClrTypeInfo(null, null, new pbr::GeneratedClrTypeInfo[] {
          new pbr::GeneratedClrTypeInfo(typeof(global::MSG_CLIENT_SERVER_PLATFORM_INFO), global::MSG_CLIENT_SERVER_PLATFORM_INFO.Parser, new[]{ "U64GIDTechStorageService", "SzPlatformName", "SzPlatformArea" }, null, null, null, null)
        }));
  }
  #endregion
}
```

---

```csharp
public static partial class MyArrayLengthReflection {

  #region Descriptor
  /// <summary>File descriptor for ProtoExtend/MyArrayLength.proto</summary>
  public static pbr::FileDescriptor Descriptor {
    get { return descriptor; }
  }
  private static pbr::FileDescriptor descriptor;

  static MyArrayLengthReflection() {
    byte[] descriptorData = global::System.Convert.FromBase64String(
        string.Concat(
          "Ch9Qcm90b0V4dGVuZC9NeUFycmF5TGVuZ3RoLnByb3RvGiBnb29nbGUvcHJv",
          "dG9idWYvZGVzY3JpcHRvci5wcm90bzo2Cg1NeUFycmF5TGVuZ3RoEh0uZ29v",
          "Z2xlLnByb3RvYnVmLkZpZWxkT3B0aW9ucxjRhgMgASgJQgP4AQFiBnByb3Rv",
          "Mw=="));
    descriptor = pbr::FileDescriptor.FromGeneratedCode(descriptorData,
        new pbr::FileDescriptor[] { global::Google.Protobuf.Reflection.DescriptorReflection.Descriptor, },
        new pbr::GeneratedClrTypeInfo(null, new pb::Extension[] { MyArrayLengthExtensions.MyArrayLength }, null));
  }
  #endregion

}
```

问题: MSGCLIENTSERVERPLATFORMINFOReflection使用了MyArrayLengthReflection自定义扩展, Unity3d使用mono编译是不会有问题的, 主要问题是Unity3d使用IL2CPP进行编译时, 会进行代码优化, 上面的代码会导致MyArrayLengthReflection被裁剪.

原因: 原因还是MyArrayLengthReflection未直接被使用, 导致IL2CPP认为MyArrayLengthReflection不会被使用.
然后protobuf使用反射去序列化MyArrayLengthReflection, 导致异常.

解决方法: 在Assert目录下增加link.xml, 明确告诉IL2CPP不要对protobuf库进行代码裁剪

```xml
<linker>
    <assembly fullname="Google.Protobuf" preserve="all"/>
</linker>
```

资料: [https://docs.unity3d.com/Manual/ManagedCodeStripping.html](https://docs.unity3d.com/Manual/ManagedCodeStripping.html)
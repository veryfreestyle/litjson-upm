# 快速开始

## JSON 与对象的相互映射

在 .Net 程序中处理 JSON 格式数据，最自然的方式是用 JSON 文本填充某个类的新实例，这个类可以是自定义的（结构与输入 JSON 匹配），也可以是更通用的字典类。

反过来，如果要将对象中的数据生成新的 JSON 字符串，则可以进行类似“导出”的操作。

为此，*LitJSON* 提供了 `JsonMapper` 类，包含两个主要方法用于 JSON 与对象的互转：`JsonMapper.ToObject` 和 `JsonMapper.ToJson`。

### 简单的 `JsonMapper` 示例

如下例所示，`ToObject` 方法有一个泛型变体 `JsonMapper.ToObject<T>`，用于指定返回对象的类型。

```cs
using LitJson;
using System;

public class Person
{
    // C# 3.0 自动实现属性
    public string   Name     { get; set; }
    public int      Age      { get; set; }
    public DateTime Birthday { get; set; }
}

public class JsonSample
{
    public static void Main()
    {
        PersonToJson();
        JsonToPerson();
    }

    public static void PersonToJson()
    {
        Person bill = new Person();

        bill.Name = "William Shakespeare";
        bill.Age  = 51;
        bill.Birthday = new DateTime(1564, 4, 26);

        string json_bill = JsonMapper.ToJson(bill);

        Console.WriteLine(json_bill);
    }

    public static void JsonToPerson()
    {
        string json = @"
            {
                \"Name\"     : \"Thomas More\",
                \"Age\"      : 57,
                \"Birthday\" : \"02/07/1478 00:00:00\"
            }";

        Person thomas = JsonMapper.ToObject<Person>(json);

        Console.WriteLine("Thomas' age: {0}", thomas.Age);
    }
}
```

示例输出：

```console
{"Name":"William Shakespeare","Age":51,"Birthday":"04/26/1564 00:00:00"}
Thomas' age: 57
```

### 使用非泛型的 `JsonMapper.ToObject`

当没有或不需要与数据结构匹配的自定义类时，可以使用非泛型的 `ToObject`，它返回一个 `JsonData` 实例。`JsonData` 是通用类型，可存储 JSON 支持的所有数据类型，包括列表和字典。

```cs
using LitJson;
using System;

public class JsonSample
{
    public static void Main()
    {
        string json = @"
          {
            \"album\" : {
              \"name\"   : \"The Dark Side of the Moon\",
              \"artist\" : \"Pink Floyd\",
              \"year\"   : 1973,
              \"tracks\" : [
                \"Speak To Me\",
                \"Breathe\",
                \"On The Run\"
              ]
            }
          }
        ";

        LoadAlbumData(json);
    }

    public static void LoadAlbumData(string json_text)
    {
        Console.WriteLine("Reading data from the following JSON string: {0}",
                          json_text);

        JsonData data = JsonMapper.ToObject(json_text);

        // 字典可像哈希表一样访问
        Console.WriteLine("Album's name: {0}", data["album"]["name"]);

        // 存储在 JsonData 实例中的标量元素可以强制转换为其本身类型
        string artist = (string) data["album"]["artist"];
        int    year   = (int) data["album"]["year"];

        Console.WriteLine("Recorded by {0} in {1}", artist, year);

        // 数组也可像普通列表一样访问
        Console.WriteLine("First track: {0}", data["album"]["tracks"][0]);
    }
}
```

示例输出：

```console
Reading data from the following JSON string:
          {
            "album" : {
              "name"   : "The Dark Side of the Moon",
              "artist" : "Pink Floyd",
              "year"   : 1973,
              "tracks" : [
                "Speak To Me",
                "Breathe",
                "On The Run"
              ]
            }
          }

Album's name: The Dark Side of the Moon
Recorded by Pink Floyd in 1973
First track: Speak To Me
```

## 读取器与写入器

另一种处理 JSON 数据的接口是通过流式读取和写入的类：`JsonReader` 和 `JsonWriter`。

这两种类型实际上是该库的基础，`JsonMapper` 就是基于它们构建的，因此可以将它们视为 LitJSON 的底层接口。

### 使用 `JsonReader`

```cs
using LitJson;
using System;

public class DataReader
{
    public static void Main()
    {
        string sample = @"{
            \"name\"  : \"Bill\",
            \"age\"   : 32,
            \"awake\" : true,
            \"n\"     : 1994.0226,
            \"note\"  : [ \"life\", \"is\", \"but\", \"a\", \"dream\" ]
          }";

        PrintJson(sample);
    }

    public static void PrintJson(string json)
    {
        JsonReader reader = new JsonReader(json);

        Console.WriteLine ("{0,14} {1,10} {2,16}", "Token", "Value", "Type");
        Console.WriteLine (new String ('-', 42));

        // Read() 方法在没有可读内容时返回 false
        while (reader.Read()) {
            string type = reader.Value != null ?
                reader.Value.GetType().ToString() : "";

            Console.WriteLine("{0,14} {1,10} {2,16}",
                              reader.Token, reader.Value, type);
        }
    }
}
```

该示例输出：

```console
         Token      Value             Type
------------------------------------------
   ObjectStart                            
  PropertyName       name    System.String
        String       Bill    System.String
  PropertyName        age    System.String
           Int         32     System.Int32
  PropertyName      awake    System.String
       Boolean       True   System.Boolean
  PropertyName          n    System.String
        Double  1994.0226    System.Double
  PropertyName       note    System.String
    ArrayStart                            
        String       life    System.String
        String         is    System.String
        String        but    System.String
        String          a    System.String
        String      dream    System.String
      ArrayEnd                            
     ObjectEnd                            
```

### 使用 `JsonWriter`

`JsonWriter` 类非常简单。通常如果要将任意对象转为 JSON 字符串，直接用 `JsonMapper.ToJson` 即可。

```cs
using LitJson;
using System;
using System.Text;

public class DataWriter
{
    public static void Main()
    {
        StringBuilder sb = new StringBuilder();
        JsonWriter writer = new JsonWriter(sb);

        writer.WriteArrayStart();
        writer.Write(1);
        writer.Write(2);
        writer.Write(3);

        writer.WriteObjectStart();
        writer.WritePropertyName("color");
        writer.Write("blue");
        writer.WriteObjectEnd();

        writer.WriteArrayEnd();

        Console.WriteLine(sb.ToString());
    }
}
```

示例输出：

```console
[1,2,3,{"color":"blue"}]
```

## 配置库的行为

JSON 是非常简洁的数据交换格式，仅此而已。因此，在程序中处理 JSON 数据时，可能需要针对一些超出 JSON 规范的小细节做出决策。

例如，读取包含单引号字符串或 JavaScript 风格注释的 JSON 字符串，这些都不是 JSON 标准的一部分，但有些开发者常用。你可以根据需要选择宽容或严格。又如，想将 .Net 对象转为带缩进的美化 JSON 字符串。

要声明你想要的行为，可以修改 `JsonReader` 和 `JsonWriter` 对象的部分属性。

### 配置 `JsonReader`

```cs
using LitJson;
using System;

public class JsonReaderConfigExample
{
    public static void Main()
    {
        string json;

        json = " /* these are some numbers */ [ 2, 3, 5, 7, 11 ] ";
        TestReadingArray(json);

        json = " [ \"hello\", 'world' ] ";
        TestReadingArray(json);
    }

    static void TestReadingArray(string json_array)
    {
        JsonReader defaultReader, customReader;

        defaultReader = new JsonReader(json_array);
        customReader  = new JsonReader(json_array);

        customReader.AllowComments            = false;
        customReader.AllowSingleQuotedStrings = false;

        ReadArray(defaultReader);
        ReadArray(customReader);
    }

    static void ReadArray(JsonReader reader)
    {
        Console.WriteLine("Reading an array");

        try {
            JsonData data = JsonMapper.ToObject(reader);

            foreach (JsonData elem in data)
                Console.Write("  {0}", elem);

            Console.WriteLine("  [end]");
        }
        catch (Exception e) {
            Console.WriteLine("  Exception caught: {0}", e.Message);
        }
    }
}
```

输出：

```console
Reading an array
  2  3  5  7  11  [end]
Reading an array
  Exception caught: Invalid character '/' in input string
Reading an array
  hello  world  [end]
Reading an array
  Exception caught: Invalid character ''' in input string
```

### 配置 `JsonWriter`

```cs
using LitJson;
using System;

public enum AnimalType
{
    Dog,
    Cat,
    Parrot
}

public class Animal
{
    public string     Name { get; set; }
    public AnimalType Type { get; set; }
    public int        Age  { get; set; }
    public string[]   Toys { get; set; }
}

public class JsonWriterConfigExample
{
    public static void Main()
    {
        var dog = new Animal {
            Name = "Noam Chompsky",
            Type = AnimalType.Dog,
            Age  = 3,
            Toys = new string[] { "rubber bone", "tennis ball" }
        };

        var cat = new Animal {
            Name = "Colonel Meow",
            Type = AnimalType.Cat,
            Age  = 5,
            Toys = new string[] { "cardboard box" }
        };

        TestWritingAnimal(dog);
        TestWritingAnimal(cat, 2);
    }

    static void TestWritingAnimal(Animal pet, int indentLevel = 0)
    {
        Console.WriteLine("\nConverting {0}'s data into JSON..", pet.Name);
        JsonWriter writer1 = new JsonWriter(Console.Out);
        JsonWriter writer2 = new JsonWriter(Console.Out);

        writer2.PrettyPrint = true;
        if (indentLevel != 0)
            writer2.IndentValue = indentLevel;

        Console.WriteLine("Default JSON string:");
        JsonMapper.ToJson(pet, writer1);

        Console.Write("\nPretty-printed:");
        JsonMapper.ToJson(pet, writer2);
        Console.WriteLine("");
    }
}
```

输出：

```console

Converting Noam Chompsky's data into JSON..
Default JSON string:
{"Name":"Noam Chompsky","Type":0,"Age":3,"Toys":["rubber bone","tennis ball"]}
Pretty-printed:
{
    "Name" : "Noam Chompsky",
    "Type" : 0,
    "Age"  : 3,
    "Toys" : [
        "rubber bone",
        "tennis ball"
    ]
}

Converting Colonel Meow's data into JSON..
Default JSON string:
{"Name":"Colonel Meow","Type":1,"Age":5,"Toys":["cardboard box"]}
Pretty-printed:
{
  "Name" : "Colonel Meow",
  "Type" : 1,
  "Age"  : 5,
  "Toys" : [
    "cardboard box"
  ]
}
```


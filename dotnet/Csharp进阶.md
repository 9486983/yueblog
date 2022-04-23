## 析构方法

- ### 作用: 释放对象

  - ##### 谁在使用呢?

  - ##### GC在调用, 用于回收非托管资源

    - ###### Windows句柄，数据库连接，TImer，FileStream等

  - ##### `~`波浪号

  - #### 需要实现IDispose接口

```c#
~DisposeTest(){
    Dispose(false)
}
```

```C#
//方式一
SqlConnection con = new Sqlconnection();
try{
    con.Opne();
}
catch(Exception){
    
}
finally{
    con.Close();//会调用析构方法, 只不过已经被封装
}

//方式二
using(SqlConnection con = new SqlConnection()){
    //使用
}
//编译后=>
SqlConnection con = new Sqlconnection();
try{
    con.Opne();
}
finally{
    con.Dispose();
}
```

##### 判断是否为非托管资源的小技巧: 查看一个类是否实现IDispose接口

- ### Close()和Dispose()的区别

  - ##### Close是关闭对象, 没有完全释放

  - ##### Dispose是完全释放, 再次使用需要重新创建对象

## 虚方法

- ### Virtual

  - ##### 作用: 允许子类/派生类，进行重写，实现不一样的功能

  - ##### 特点: 方便维护

```c#
//当前创建了加法类
public class AdditionCalculateHelper{
    public virtual Calculate(int num1, int num2){
        return num1 + num2;
    }
}

//通过继承加法类, 重写Calculate方法，实现乘法
public class MultiplicationCalculateHelper : AdditionCalculateHelper {
    public override Calculate(int num1, int num2){
        return num1 * num2;
    }
}
```

```C#
public class Program{
    public void Main(string[] args){
        AddtionCalculateHelp add = new AddtionCalculateHelp();
        Console.Write(add.Calculate(2, 3));//5
        MultiplicationCalculateHelper mult = new MultiplicationCalculateHelper();
        Console.Write(mult.Calculate(2, 3));//6
    }
}
```



## 抽象方法

- ### 定义: 

  - ##### 抽象方法一定要写在抽象类中

  - ##### 抽象类不能实例化(new)，只能用派生类实现

  - ##### 不带方法体

- ### 使用场合

  - ##### 强制性一定要实现

```C#
public abstract class AbstractCalculate {
    //规范好让子类实现
    public abstract int Calculate(int num1, int num2);
}

//继承时一定要实现
public abstract class AbstractAddCalculate: AbstractCalculate {
    public override int Calculate(int num1, int num2){
        return num1 + num2;
    }
}
```

- ### 和接口`Interface`区别使用场合

  - #### 区别: 

    - ##### 抽象类单继承，接口可以多继承

    - ##### 抽象类中可以写普通方法，虚方法。接口只能写规范，不能实现

  - #### 使用场合: 

    - ##### 抽象类常用于比较固定的，抽象范围大一些的事物

      - ###### 动物=>

        - ###### 男人

        - ###### 女人

    - ##### 接口适用于经常修改，只是一个定义规范的地方



## 扩展方法

- 在不修改原方法的基础对方法进行扩展
- 只允许在`静态类`中定义`静态方法`
- 在方法的第一个参数前加上`this`即表示为它扩展

```c#
public static class ObjectExtend{
    
    public static string ToJson(this object obj){
        try {
            return JsonConvert.SerializeObject(obj);
        }
        catch {
            return null;
        }
    }
    
    /// <summary>
    /// 模拟SQL中的ISNULL: 校验_this是否为null, 如果是则返回_that
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="_this"></param>
    /// <param name="_that"></param>
    /// <returns></returns>
    public static T IsNull<T>(this T _this, T _that)
    {
        if (_this == null && _that == null)
            throw new NullReferenceException();
        if (_this != null)
            return _this;
        else
            return _that;
    }
}

class Program{
    static Main(string[] args){
    	var obj = new { Title = "扩展方法", DateTime = DateTime.Now };
    	string json = obj.ToJson();
        string json1 = json.IsNull("{}");
	}
}

```

## Task的CancellationTokenSource

```C#
class Program
{
    static void Main(string[] args)
    {
        CancellationTokenSource tokenSource = new CancellationTokenSource();
        //设置30秒后停止
        tokenSource.CancelAfter(30 * 1000);
        //停止时的操作
        tokenSource.Token.Register(() => { Console.WriteLine("------任务结束------"); });
        Excute(tokenSource.Token);
        //也可以手动调用Cancel结束任务
        //tokenSource.Cancel();	
        
        Console.ReadKey();
    }
   	
    public static void Excute(CancellationToken token)
    {
        Task.Run(() =>
        {
           while (!token.IsCancellationRequested)
           {
               Thread.Sleep(100);
               Console.WriteLine(DateTime.Now.ToString("G"));
           }
        }, token);

    }
}
```

## Sunday算法

##### 字符串模式匹配，获得匹配字符串的下标

```C#
public static int Sunday(string text, string pattern)
{
    int i, j, k;
    i = j = 0;
    int tl, pl;
    int pe;
    int rev = -1;

    if ((null == text || null == pattern) || (tl = text.Length) < (pl = pattern.Length))
        return -1;

    while (i < tl && j < pl)
    {
        //如果匹配, 则两个索引++, 直到j的长度=pattern的长度
        if (text[i] == pattern[j])
        {
            //匹配正确，就继续
            ++i;
            ++j;
            continue;
        }

        //匹配失败
        pe = i + pl;
        if (pe >= tl) return -1;

        for (k = pl - 1; k >= 0 && text[pe] != pattern[k]; --k) { }

        i += (pl - k);  //(pl - k) 表示i需要移动的步长
        rev = i;   //记录当前索引
        j = 0;  //重新开始
    }

    return i < tl ? rev : -1;
}
```


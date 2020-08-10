Java程序开发过程中,需要从键盘获取输入值是常有的事，但Java它偏偏就没有像c语言给我们提供的scanf()，C++给我们提供的cin()获取键盘输入值那样简易而方便的现成函数！Java没有提供这样的函数也不代表遇到这种情况我们就束手无策，我们可以用以下三种方法来读取键盘输入

方法一：从控制台接收一个字符，然后将其打印出来

`public static void main(String [] args) throws IOException{` 

​     `System.out.print("Enter a Char:");` 

​     `char i = (char) System.in.read();` 

​     `System.out.println("your char is :"+i);` 

`}` 

虽然此方式实现了从键盘获取输入的字符，但是System.out.read()只能针对一个字符的获取，同时，获取进来的变量的类型只能是char，当我们输入一个数字，希望得到的也是一个整型变量的时候，我们还得修改其中的变量类型，这样就显得比较麻烦。

方法二：从控制台接收一个字符串，然后将其打印出来。 在这个题目中，我们需要用到BufferedReader类和InputStreamReader类

import java.io.*;

public static void main(String [] args) throws IOException{ 

​      BufferedReader br = new BufferedReader(new InputStreamReader(System.in)); 

​      String str = null; 

​      System.out.println("Enter your value:"); 

​      str = br.readLine(); 

​      System.out.println("your value is :"+str); 

}

这样我们就能获取我们输入的字符串。

方法三：这种方法我认为是最简单，最强大的，就是用Scanner类

import java.util.Scanner;

public static void main(String [] args) { 

​     Scanner sc = new Scanner(System.in); 

​     System.out.println("请输入你的姓名："); 

​     String name = sc.nextLine(); 

​     System.out.println("请输入你的年龄："); 

​     int age = sc.nextInt(); 

​     System.out.println("请输入你的工资："); 

​     float salary = sc.nextFloat(); 

​     System.out.println("你的信息如下："); 

​     System.out.println("姓名："+name+"\n"+"年龄："+age+"\n"+"工资："+salary); 

}

这段代码已经表明，Scanner类不管是对于字符串还是整型数据或者float类型的变量，只需做一点小小的改变，就能够实现功能！无疑他是最强大的！

但是，在使用第三种输入方法的时候有一个需要注意的地方，就是nextLine()函数，在io包中有一个和他功能一样的函数我next()函数，他们的功能一样，但是在实现上有什么差别呢，请看下面代码：

public static void main(String [] args) { 

​     Scanner sc = new Scanner(System.in); 

​     System.out.println("请输入你的年龄："); 

​     int age = sc.nextInt(); 

​     System.out.println("请输入你的姓名："); 

​     String name = sc.nextLine(); 

​     System.out.println("请输入你的工资："); 

​     float salary = sc.nextFloat(); 

​     System.out.println("你的信息如下："); 

​     System.out.println("姓名："+name+"\n"+"年龄："+age+"\n"+"工资："+salary); 

}

这段代码和上边第三种实现输入方法给出的例子代 码区别在于，这段代码先执行nextInit()再执行nextLine()，而第三种方法的例子是先执行nextLine()，再执行 nextInit()，当你在运行着两段代码的时候你会发现第三种方法的例子可以实现正常的输入，而这段代码却在输入年龄，敲击enter键后，跳过了输 入姓名，直接到了输入工资这里，（可以自己运行代码看看）这是为什么呢？其实，在执行nextInit()函数之后，敲击了enter回车键，回车符会被 nextLine()函数吸收，实际上是执行了nextLine()函数吸收了输入的回车符（并不是没有执行nextLine函数），前面讲到和 nextLine()功能一样的函数next(),他们的区别就在于:next()函数不会接收回车符和tab，或者空格键等，所以在使用 nextLine()函数的时候，要注意敲击的回车符有个被其吸收，导致程序出现BUG！！！

最后小小的总结一下next()和nextLine()的区别：

在java中，next()方法是不接收空格的，在接收到有效数据前，所有的空格或者tab键等输入被忽略，若有有效数据，则遇到这些键退出。nextLine()可以接收空格或者tab键，其输入应该以enter键结束。
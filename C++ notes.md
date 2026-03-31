# C++ Notes

## 26/03/31

### cin.putbacK()

    char ch;        
    cin.get(ch);    //cin.putback()只能在已经读取过的位置放回字符，不能向空输入流中任意插入字符。
    string s;
    cin >> s;       //读取123
    cin.putback('0');
    cin >> s;       //读取0
    cout << s << '\n';
    cin >> s;       //读取456
    cout << s;

输入：
a 123 456
输出：
0
456

这是因为cin.putback()会把字符插入缓冲区的开头，而不是末尾。
读取完123后，缓冲区为" 456"。
cin.putback('0')插入后，缓冲区为"0 456"。

在修PPP第6章的计算器的bug时发现了这个特性。
    void Token_stream::ignore(char c)
    {
        if (full && c == buffer.kind)
        {
            full = false;
            return;
        }
        full = false;

        char ch;
        while (cin.get(ch))
            if (ch == c)
                return;
    }
有一种情况使得进入这个函数时缓冲区为空，使得程序在这里需要用户输入。
就想着用cin.putback(c)在缓冲区末尾加上结束符，解决需要输入的问题，cin.putback()只能插入到开头。

## 26/03/30

### static静态局部变量和函数返回引用类型

    const Date& today()
    {
            static const Date today = get_date_from_clock();             // initialize today the first time we get here
            return today;
    }

static声明的局部变量在第一次初始化后，生命周期和全局变量相同。
在第一次使用today()以后，today = get_date_from_clock()，都只是修改了局部变量today的值，而没有创建新的变量。

## 26/03/29

### cin输出浮点数的有效位数

    double x = 3.14'1523123'2313'123'111;
    cout << x;

输出：3.14152
因为cin默认输出浮点数的6位有效数字

    double y = 12345678.9;
    cout << y;

控制台会输出：1.23457e+07

使用setprecision()输出更多有效数字

    double x = 3.14'1523123'2313'123'111;       //(')是位分隔符，帮助阅读
    cout << setprecision(15) << x;

输出：3.14152312323131

## 26/03/25

### 引用类型和const

如果只是读取数据，可以在引用类型前加上const，防止参数被意外修改。

    void print(const vector<double>& v)                   // pass-by-const-reference
    {
            cout << "{ ";
            for (int i = 0; i<v.size(); ++i) {
                    cout << v[i];
                    if (i!=v.size()−1)
                            cout << ", ";
            }
            cout << " }\n";
    }

### 函数声明和定义

函数声明可以不用带参数

    int my_find(vector<string>, string, int);                    // not naming arguments

    void e7_4()
    {
        vector<string> vs;
        string s;
        cout << my_find(vs, s, 3) << '\n';
    }

    int my_find(vector<string> vs, string s, int hint)
    // search for s in vs starting at hint
    {
        if (hint < 0 || vs.size() <= hint)
            hint = 0;
        for (int i = hint; i < vs.size(); ++i)        // search starting from hint
            if (vs[i] == s)
                return i;
        for (int i = 0; i < hint; ++i)                    // if we didn’t find s, so search before hint
            if (vs[i] == s)
                return i;
        return -1;
    }

如果不再需要用第三个参数，可以不命名。

    int my_find(vector<string> vs, string s, int)
    {
        for (int i = 0; i<vs.size(); ++i)
        if (vs[i]==s) 
            return i;
        return −1;
    }

## 26/03/24

### cin.get()

cin.get()可以读取输入流中的空格和换行符

    while (cin)
    {
        char ch = cin.get();    //也可以char ch; cin.get(ch);
        if (ch == '\n')
            cout << "find a \\n.";
        if (ch == ' ')
            cout << "find a whitespace.";
        if (isspace(ch))
            cout << " That's a space." << '\n';
    }

输入：
123 456 abc dsds
输出：
find a whitespace. That's a space.
find a whitespace. That's a space.
find a whitespace. That's a space.
find a \n. That's a space.

## 26/03/22

### 标准库函数isaplha()

        isalpha('）');

![img](img/2026-03-22-17-30-51.png)
函数参数的值超出了有效范围[-1，255]。
 '）' 是宽字符，Unicode码点是65289。

## 26/03/19

### 函数参数引用传递

    void deal_with_plus_sign(string &s)
    {
        if (s[0] == '+')
            s.erase(0, 1);
    }

    bool is_integer(string s)
    {
        deal_with_plus_sign(s);
        for (char x : s)
        {
            if (x < '0' || x > '9')
                return false;
        }
        return true;
    }

    int string_to_positive_integer(string s)
    {
        if (is_integer(s))
        {
            int num = 0;
            int digit = 1;
            int size = s.size();
            int add = -1;
            for (int i = size - 1; i > -1; --i)
            {
                add = (s[i] - '0') * digit;
                //边界检查
                if (num > numeric_limits<int>::max() - add)
                    return -1;
                num += add;
                digit *= 10;
            }
            return num;
        }
        else
            return -1;
    }

deal_with_plus_sign(string &s)是新加的用于处理正号的函数。
只在这个函数里使用引用类型，程序仍不能正常运行。
因为它只修改了is_integer() 从 string_to_positive_integer()获取的字符串s的副本，
没有改变string_to_positive_integer()使用的字符串s。

需要在is_integer()内也使用引用类型。is_integer(string &s)

## 26/03/17

### ixx文件的函数声明没有加参数

![img](img/2026-03-17-09-49-04.png)
![img](img/2026-03-17-09-49-24.png)

连接错误，编译器找不到无参数的test1()函数实现。

## 26/03/16

### int类型 与 阶乘

        int fac = 1;
        for (int i = 1; i < 20; ++i)
        {
            for (int j = i; j > 0; --j)
            {
                fac *= j;
            }
            cout << "fac(" << i << ") == " << fac << '\n';
            fac = 1;
        }

输出：
fac(1) == 1
fac(2) == 2
fac(3) == 6
fac(4) == 24
fac(5) == 120
fac(6) == 720
fac(7) == 5040
fac(8) == 40320
fac(9) == 362880
fac(10) == 3628800
fac(11) == 39916800
fac(12) == 479001600
fac(13) == 1932053504
fac(14) == 1278945280
fac(15) == 2004310016
fac(16) == 2004189184
fac(17) == -288522240
fac(18) == -898433024
fac(19) == 109641728

因为
12! == 479001600;
13! == 6227020800 > 2147483647 (图中显示1932053504是因为只保留了32位的值，超出部分被截断)
所以int类型最多只能存12!。

检查溢出的方法：

加法：if(num > numeric_limits\<int>::max() - addendd)

乘法：if(num > numeric_limits\<int>::max() / multiplier)

如果是计算排列和组合，可以用for循环。

### cin输入CTRL+Z

    string s;
    while (true)
    {
        std::cin >> s;
        
        //省略中间代码...

        s.clear();
    }

string 能存储所有字符，所以cin很难进入错误状态。
但如果单独输入CTRL+Z，会使cin进入错误状态。
可以在上面的循环末尾加上cin.clear();重置cin的状态。

另外，CTRL+Z 只有在单独一行时才会被识别为 EOF。如果它出现在其他字符之后，则会被视为普通字符。

    string s;
    int count = 0;
    while (count < 10)
    {
        cin >> s;
        ++count;
    }
    cout << "end successfully";

输入：
CTRL+Z 0909s
输出：
end successfully

而输入：
0909s CTRL+Z
控制台显示：
![img](img/2026-03-16-10-15-59.png)
仍需要输入，说明cin没有进入错误状态，这又说明CTRL+Z没有被识别为EOF。

## 26/03/15

### <<和&&的优先级

    int num1 = 0b1010;      //10
    int num2 = 0b1001;      //9
    cout << num1 && num2;

输出了10，因为“<<”的优先级比“&&”高，所以输出的实际上是0b1010的十进制值，然后计算(cout)&&(0b1001)得到true。

查看(cout)&&(0b1001)的结果：

    cout << (cout << num1 && num2);

输出：
101

与运算的运算符是“&”，而不是“&&”。

    cout << (num1 & num2);
    cout << bitset<4>(num1 & num2);

输出：
8
1000

直接比较优先级：
    int x = 10;
    int y = 10;
    cout << "x&y == " << x & y << '\n'
        << "x|y == " << x | y << '\n'
        << "x^y == " << x ^ y << '\n'
        << "~x == " << ~x << '\n'
        << "!x == " << !x << '\n';

![img](img/2026-03-31-09-39-27.png)

编译器提示表达式具有未区分范围的枚举类型，也是因为运算符“<<”优先级比位运算符(&, |, ^, ~, !)高

所以，直接使用cout输出位运算的表达式时，应该给表达式加上括号cout << (x & y)。

### 异常处理

    try
    {
        int num;
        cin >> num;
        switch (num)
        {
        case 0:
            throw runtime_error("It's '0'.");
            cout << " Why not?" << '\n';
            break;
        }
    }
    catch (exception& e)
    {
        cerr << "Exception: " << e.what() << '\n';
    }

输入：
0
输出：
0
Exception: It's '0'.        //没有输出" Why not?"

在case 0:分支的error()函数抛出异常后，程序的控制权就会立即转移给异常处理机制，当前作用域内剩余的正常执行流程都会被中断。所以cout << " Why not?" << '\n';没有被执行。

### 输入数据超出int范围使cin进入错误状态

    void input_pairs(vector<Name_value> Nvs)
    {
        bool duplicate_name = false;
        string name = " ";
        int grade = -1;
        while (true)
        {
            duplicate_name = false;
            cin >> name >> grade;

            //结束输入
            if (name == "NoName" && grade == 0)
                break;
            
            //检查输入名字是否已存在
            for (Name_value x : Nvs)
            {
                if (name == x.name)
                {
                    duplicate_name = true;
                    break;
                }
            }
            if (duplicate_name)
            {
                cout << "It's a duplicate name, you have entered it before." << '\n';
            }
            else
            {
                Name_value nv(name, grade);
                Nvs.push_back(nv);
            }
        }
    }

输入（5，55555555555555555555555），程序会进入死循环，持续输出“It's a duplicate name, you have entered it before.”
![img](img/2026-03-15-09-45-53.png)

因为55555555555555555555555超出了int的最大范围2147483647，使cin进入错误状态。
在第二次及以后的循环，name和grade的值始终为第一次输入的值。

可以输入字符，然后转换为数字，因为输入字符基本不会使cin出错；
或者每次循环结束时重置cin的状态。

## 26/03/11

### cin.putback()

        char i = '/';
        cin >> i;
        cin.putback(i);
        int c = 0;
        cin >> c;
        cout << "c == " << c;

输入565623，控制台打印565623，觉得奇怪，因为char只能存-128到127。

实际流程为：输入565623，i只读取了一个字符‘5’，而没有读取全部的565623，读取后缓冲区还剩下65623。
cin.putback(i)把‘5’放回缓冲区后，cin又把565623全部读入c。

注释掉cin.putback(i);后，再次输入565623。
可以看到控制台打印的是65623

## 26/03/09

### 异常输出多字符字面量

    void exercise12()
    {
        vector<int> target;
        int seconds = time(0);
        default_random_engine engine(seconds);
        uniform_int_distribution<int> dist(0, 9);
        for (int i = 0; i < 4; ++i)
            target.push_back(dist(engine));
        
        cout << "target.size() == " << target.size() << '\n';
        cout << "The answer is: ";
        for (int i = 0; i < target.size(); ++i)
            cout << target[i];
        cout << ' !' << '\n';
    }

这个生成随机数函数会输出8位数，但后四位始终输出8225。
起初以为是越界访问了，但target.size()大小正确。
调试发现，cout << ' !' << '\n';，这一行输出了4个数，而' !'为“空格+！”，是多字符字面量。
多字符字面量将字符组合成一个一个整数，输出它的数值表示，而不是字符。

    cout << ' !' << '\n';

输出：8225

### cin的状态

    int input = -1;
    int count = 0;
    string s = "^_^";
    bool incorrect = true;
    while (incorrect)
    {
        cin >> input;
        cin >> s;
        if(s != "^_^")
            cout << "Invalid input, you should enter a 4-digit integer." << '\n';
        else
        {
            if ((input < 1000) || (input > 9999))
                cout << "Invalid input. A 4-digit number is required." << '\n';
            cout << "Input == " << input << '\n';
        }
        s = "^_^";
        ++count;
    }

第一次输入“\*”，会使cin进入错误状态，会跳过第二次输入，导致缓冲区的“\*”不会被消费，程序进入死循环。

    int x;
    string s;

    cin >> x;
    //cin.clear();
    cin >> s;

    cout << "x == " << x << '\n'
        << "s == " << s << '\n';

输入：*
输出：
x == 0
s ==

使用cin.clear()重新设置cin为默认的正确状态。

## 26/03/08

### cout的状态

    if (cout)
        cout << "Success!\n";
    else cout << "Fail\n";

if检查cout的状态，cout的默认状态为真，所以会输出“Success!”。

    if(cout << "Success1!\n" << "Success2!\n" << "Success3!\n");

输出：
Success1!
Success2!
Success3!

输出3次，设置3次cout的状态。

### cin的特殊处理

    cout << "Please enter some integers(press '|' to stop): " << '\n';
    vector<int> v;
    char c = ' ';
    int n;
    while (c != '|')
    {
        if(cin >> n)
            v.push_back(n);
        else
        {
            cin.clear();
            if (c != '|')
                cout << "That's not an integer! If you want to stop input, enter '|'." << '\n';
        }
    }

输入‘+’，else分支会执行一次然后进入下一个循环，而输入‘*’或‘=’，程序会进入死循环。
这里先不关注输入‘*’或‘=’，把目光放在输入‘+’上。
cin 对 ‘+’ 和 ‘-’ 进行特殊处理。‘+’被视为正号；‘-’被视为负号。

    int num;
    cin >> num;
    cout << num;

输入：+7
输出：7

输入：-7
输出：-7

另外，如果输入错误，会把变量赋值为0，同时把cin进入错误状态。

    int num;
    if (cin >> num)
    {
        cout << num << '\n';
    }
    cout << num;

输入：+
输出：0

输入：+0
输出：
0
0

另外的另外，

    void exercise8()
    {
        cout << "Please enter some integers(press '|' to stop): " << '\n';
        vector<int> v;
        char c = ' ';
        int n;

        while (c != '|')
        {
            if(cin >> n)
                v.push_back(n);
            else
            {
                cin.clear();
                cin >> c;
                if (c != '|')
                    cout << "That's not an integer! If you want to stop input, enter '|'." << '\n';
            }
        }
    }

输入：

![img](img/2026-03-08-16-07-45.png)
![img](img/2026-03-08-16-08-02.png)
![img](img/2026-03-08-16-19-06.png)

这说明，虽然‘+’和其他字符虽然同为字符，cin在碰到时它们时会把变量值设置为0，且进入错误状态，但cin会消费掉缓冲区的‘+’，但不会消费掉缓冲区的其他字符。

## 26/03/06

main.cpp

    import drill;

    int main()
    {
        try
        {
            throw runtime_error(">?");
            return 0;
        }
        catch (exception& e)
        {
            cerr << "error: " << e.what() << '\n';
            return 1;
        }
        catch (...)
        {
            cerr << "Oops: unknown exception!" << '\n';
            return 2;
        }
    }

4.drill.ixx

    export module drill;

    import std;

    using namespace std;

    export void drill_1();

运行main.cpp，会提示标准库函数未定义，因为using namespace std;的作用域仅限于当前文件->4.drill.ixx。
而main.cpp里没有这个定义。

4.drill.cpp

    module drill;
    void drill_1()
    {
        try
        {
            throw runtime_error(">?");
        }
        catch (exception& e)
        {
            cerr << "error: " << e.what() << '\n';
        }
        catch (...)
        {
            cerr << "Oops: unknown exception!" << '\n';
        }
    }

修改后的main.cpp

import drill;

    int main()
    {
        drill_1();
        return 0;
    }

使用main.cpp调用drill_1()函数，程序正常执行，没有未定义错误。
这是因为main函数只是启动执行drill_1()函数和获取它的结果，该函数没有在main.cpp里执行。
而drill_1()是在4.drill.ixx和4.drill.cpp共同组成的drill模块中执行的，模块导入了std标准库且声明了使用std::命名空间。

## 26/02/16

### cin被隐式转换为bool类型

    vector<double> temps;
    for (double temp; cin >> temp;)
        temps.push_back(temp);

    double sum = 0;
    for (double x : temps)      //  ==     for (int i = 0; i < temps.size(); i++)   double x = temps[i];
        sum += x;
    cout << "Average temperatrue: " << sum / temps.size() << '\n';
    return 0;

Basically, cin>>temp is true if a value was read correctly and false otherwise,
so that for-statement will read all the doubles we give it and stop when we give it anything else.
For example, if you typed    1.2 3.4 5.6 7.8 9.0 |
then temps would get the five elements 1.2, 3.4, 5.6, 7.8, 9.0 (in that order, for example, temps[0]==1.2).
We used the character '|' to terminate the input – anything that isn’t a double can be used.

### string是字符容器，而不是一个简单的整体

    cout << "Please enter a string" << endl;
    string s;
    cin >> s;
    for (char c : s)
        cout << c << '\t' << int(c) << endl;
    
输入：string
输出：
s       115
t       116
r       114
i       105
n       110
g       103

for (char c : s) cout << c << '\t' << int(c) << endl;
能遍历输出字符串s的每个字符，因为std::string提供了迭代器接口。。。

### 副本与使用引用

    //替换敏感词为“BLEEP”
    vector<string> words;
    cout << "Please enter a few words." << endl;

    for (string word; cin >> word;)
        words.push_back(word);

    for (string word : words)
        if (word == "Broccoli" || word == "broccoli")
            word = "BLEEP";

    for (string word : words)
        cout << word << endl;

西兰花不会被成功替换，因为第二个for循环的word是数组元素的副本，修改时不会修改原数组。

![img](img/2026-03-08-16-32-53.png)
输入：a broccoli a banana a bear some Broccoli brobroccoli
再输入：CTRL+Z
输出：
a
broccoli
a
banana
a
bear
some
Broccoli
brobroccoli

应该使用引用类型，for (string &word : words)
修改后：
使用同样的输入
输出：
a
BLEEP
a
banana
a
bear
some
BLEEP
brobroccoli

broccoli成功被替换为BLEEP

## 26/02/14

    cout << "Please enter yen('y'), kroner('k'), or pounds('p') and I'll convert it into dollars." << endl;
    double value;
    char unit = ' ';
    cin >> value >> unit;
    switch (unit)
    {
    case 'y':
        cout << value << unit << " is " << value * 0.0065 << " dollars";
        break;
    case 'k':
        cout << value << unit << " is " << value * 0.1589 << " dollars";
        break;
    case 'p':
        cout << value << unit << " is " << value * 1.3647 << " dollars";
        break;
    default:
        cout << "Sorry I don't know a unit called " << unit;
    }
    cout << endl;

输入1yuan,会输出“1y is 0.0065 dollars”，而非 “Sorry I don't know a unit called yuan”。
因为cin在从缓冲区读取时，只会读取yuan的第一个字符y，使得程序执行'y'分支。

## 26/02/13

### 1.cin输入时类型不匹配导致输入失败

    cout << "Please enter two integer values." << endl;
    int val1, val2;
    cin >> val1 >> val2;
    if (val1 < val2)
        cout << val1 << " is smaller than " << val2;
    else if (val1 > val2)![alt text](image-7.png)
        cout << val1 << " is larger than " << val2;
    else
        cout << val1 << " equals " << val2;
    cout << endl;
    int sum = val1 + val2;
    int difference = val1 - val2;
    int product = val1 * val2;
    int ratio = val1 / val2;

    cout << val1 << " + " << val2 << " == " << sum << endl
        << val1 << " - " << val2 << " == " << difference << endl
        << val1 << " * " << val2 << " == " << product << endl
        << val1 << " / " << val2 << " == " << ratio << endl;

输入3.1，按下回车，不等输入第二个值，程序直接显示输入结果。

![alt text](img/image-3.png)

cin>>val1时：cin从缓冲区读取3后读取到小数点，停止读取，3存入val1。
cin>>val2时：缓冲区还剩下.1,cin读取到小数点，读取失败，val2仍是未初始化的状态。

## 26/01/31

### cin的特性

1.如果一直没有遇到非空白字符，则会持续等待真正的输入（忽略这次输入）

2.如果检查到非空白字符，则在下一次遇到空白字符时，会使用空白字符以前的字符串，将空白字符以后的字符串存入缓冲区。

    string previous;
    string current;
    cin >> previous >> current;
    cout << "previous == " << previous << ", current == " << current << " \n";

输入：
    1.Charles Dickens
    2.Mr. Charles Dickens

输出：
    1.
    previous == Charles, current == Dickens
    2.
    previous == Mr., current == Charles

        string previous;
        string current;
        while (cin >> current)
        {
            if (previous == current)
                cout << current: <<
                cout << "repeated word: " << current << "\n";
            previous = current;
        }

输入：
    1.The cat cat jumped
    2.She she laughed "he he he!" because what he did did not look very very good good
输出：
    1.repeated word: cat
    2.
    repeated word: did
    repeated word: very
    repeated word: good

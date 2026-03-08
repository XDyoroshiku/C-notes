# C++ Notes

## 26/03/08

### cout的状态

    if (cout)
        cout << "Success!\n";
    else cout << "Fail\n";

if检查cout的状态，cout的默认状态为真，所以会输出“Success!”。

![img](img/2026-03-08-09-40-35.png)

    if(cout << "Success1!\n" << "Success2!\n" << "Success3!\n")

会输出3次，设置3次cout的状态。

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
这个程序进入死循环是正常的。
cin 对 ‘+’ 和 ‘-’ 进行特殊处理。‘+’被视为正号；‘-’被视为负号。

    int main()
    {
        int num;
        cin >> num;
        cout << num;
    }

![img](img/2026-03-08-15-02-39.png)
![img](img/2026-03-08-15-03-01.png)

另外，如果输入错误，会把变量赋值为0，同时把cin进入错误状态。

    int main()
    {
        int num;
        if (cin >> num)
        {
            cout << num << '\n';
        }
        cout << num;
    }

![img](img/2026-03-08-15-43-58.png)
![img](img/2026-03-08-15-44-27.png)
![img](img/2026-03-08-15-45-24.png)
![img](img/2026-03-08-15-50-54.png)
![img](img/2026-03-08-15-51-17.png)

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

    int main()
    {
        vector<double> temps;
        for (double temp; cin >> temp;)
            temps.push_back(temp);

        double sum = 0;
        for (double x : temps)      //  ==     for (int i = 0; i < temps.size(); i++)   double x = temps[i];
            sum += x;
        cout << "Average temperatrue: " << sum / temps.size() << '\n';
        return 0;
    }

Basically, cin>>temp is true if a value was read correctly and false otherwise,
so that for-statement will read all the doubles we give it and stop when we give it anything else.
For example, if you typed    1.2 3.4 5.6 7.8 9.0 |
then temps would get the five elements 1.2, 3.4, 5.6, 7.8, 9.0 (in that order, for example, temps[0]==1.2).
We used the character '|' to terminate the input – anything that isn’t a double can be used.

### string是字符容器，而不是一个简单的整体

    import std;

    using namespace std;

    void exercise2()
    {
        cout << "Please enter a string" << endl;
        string s;
        cin >> s;
        for (char c : s)
            cout << c << '\t' << int(c) << endl;
    }
for (char c : s) cout << c << '\t' << int(c) << endl;
能遍历输出字符串s的每个字符，因为std::string提供了迭代器接口。。。
![img](img/2026-02-16-21-31-21.png)

### 副本与使用引用

    import std;

    using namespace std;

    //替换敏感词为“BLEEP”
    int main()
    {
        vector<string> words;
        cout << "Please enter a few words." << endl;

        for (string word; cin >> word;)
            words.push_back(word);

        for (string word : words)
            if (word == "Broccoli" || word == "broccoli")
                word = "BLEEP";

        for (string word : words)
            cout << word << endl;

        return 0;
    }

西兰花不会被成功替换，因为第二个for循环的word是数组元素的副本，修改时不会修改原数组。

更正：使用引用，for (string &word : words)
![img](img/2026-03-08-16-32-53.png)

## 26/02/14

    void e3412()
    {
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
    }

输入1yuan,会输出“1y is 0.0065 dollars”，而非 “Sorry I don't know a unit called yuan”。
![img](img/2026-03-08-16-31-00.png)
因为cin在从缓冲区读取时，只会读取yuan的第一个字符y，使得程序执行'y'分支。

## 26/02/13

### 1.cin输入时类型不匹配导致输入失败

    void exercise4()
    {
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
    }

输入3.1，按下回车，不等输入第二个值，程序直接显示输入结果。
![alt text](img/image-3.png)

cin>>val1时：cin从缓冲区读取3后读取到小数点，停止读取，3存入val1。
cin>>val2时：缓冲区还剩下.1,cin读取到小数点，读取失败，val2仍是未初始化的状态。

## 26/01/31

### cin的特性

1.如果一直没有遇到非空白字符，则会持续等待真正的输入（忽略这次输入）

2.如果检查到非空白字符，则在下一次遇到空白字符时，会使用空白字符以前的字符串，将空白字符以后的字符串存入缓冲区。

    #include <iostream>

    using namespace std;

    int main()
    {
        string previous;
        string current;
        cin >> previous >> current;
        cout << "previous == " << previous << ", current == " << current << " \n";
        return 0;
    }

输入：
    1.Charles Dickens
    2.Mr. Charles Dickens

输出：
    1.
    ![alt text](img/image.png)
    2.
    ![alt text](img/image-1.png)

    #include <iostream>

    using namespace std;

    int main()
    {
        string previous;
        string current;
        while (cin >> current)
        {
            if (previous == current)
                cout << current: <<
                cout << "repeated word: " << current << "\n";
            previous = current;
        }
        return 0;
    }

输入：
    1.The cat cat jumped
    2.She she laughed "he he he!" because what he did did not look very very good good
输出：
    ![alt text](img/image-2.png)

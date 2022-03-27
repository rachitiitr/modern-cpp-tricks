## 介绍  <!-- omit in toc -->
以下列出了在面试和竞赛编程中经常使用的C++技巧。
如果你喜欢 Rachit 的工作，你可以在以下平台关注作者 - 
- Discord - https://bit.ly/discord-rachit
- Programming YouTube Channel - https://bit.ly/rachityoutube

## 目录 <!-- omit in toc -->
- [别再嵌套 `min(x, min(y, ...))`](#别再嵌套-minx-miny-)
- [像 JavaScript 析构一样在C++中使用结构化绑定](#像-JavaScript-析构一样在C++中使用结构化绑定)
- [ 强大的 Log 和 Debug](#强大的-Log-和-Debug)
  - [debug 宏如何工作?](#debug-宏如何工作?)
  - [这个宏存在的问题 - 不可扩展](#这个宏存在的问题-\--不可扩展)
  - [用强大的宏来解决问题](#用强大的宏来解决问题)
- [用于多变量和容器的泛型读写器](#用于多变量和容器的泛型读写器)
- [C++中的装饰器和多参数](#C++中的装饰器和多参数)
  - [YouTube 直播 Demo](#YouTube-直播-Demo)
  - [一行打印尽可能多的变量](#一行打印尽可能多的变量)
  - [C++ 中强大的装饰器函数](#C++-中强大的装饰器函数)
  - [通过嵌套的方式利用装饰器函数](#通过嵌套的方式利用装饰器函数)


## 别再嵌套 `min(x, min(y, ...))`
使用初始化列表搭配 `std::min`、`std::max` 让生活变得更美好
```cpp
small = min(x, min(y, min(z, k))); // 旧方法
small = min({x, y, z, k}); // 更轻松的办法
```

## 像 JavaScript 析构一样在C++中使用结构化绑定
```cpp
pair<int, int> cur = {1, 2};
auto [x, y] = cur;
// x等于1，y等于2
// 不需要使用cur.first和cur.second获取值

array<int, 3> arr = {1, 0, -1};
auto [a, b, c] = arr;
// a等于1, b等于0, c等于-1
```


----------------


## 强大的 Log 和 Debug

### debug 宏如何工作?
直奔主题，我经常使用 `debug` 宏，它能够将变量名和变量转化成字符串。

```cpp
#define deb(x) cout << #x << " " << x 
int ten = 10;
deb(ten); // 打印 "ten = 10"
```

在 debug 的时候非常有用。

### 这个宏存在的问题 - 不可扩展
当你有多个变量要记录时，最终你会有更多的 `deb2` 和 `deb3` 宏。

```cpp
#define deb(x) cout << #x << " " << x 
#define deb2(x) cout << #x << " " << x << " "  << #y << " " << y 
#define deb3(x, y, z) cout << #x << " " << x << " "  << #y << " " << y << " "  << #z << " " << z 
```

这就造成了不可扩展。

### 用强大的宏来解决问题
这个办法就是用变量宏和折叠表达式

```cpp
#define deb(...) logger(#__VA_ARGS__, __VA_ARGS__)
template<typename ...Args>
void logger(string vars, Args&&... values) {
    cout << vars << " = ";
    string delim = "";
    (..., (cout << delim << values, delim = ", "));
}

int xx = 3, yy = 10, xxyy = 103;
deb(xx); // 打印 "xx = 3"
deb(xx, yy, xxyy); // 打印 "xx, yy, xxyy = 3, 10, 103"
```


----------------


## 用于多变量和容器的泛型读写器
```cpp
template <typename... T>
void read(T &...args) {
    ((cin >> args), ...);
}

template <typename... T>
void write(string delimiter, T &&...args) {
    ((cout << args << delimiter), ...);
}

template <typename T>
void readContainer(T &t) {
    for (auto &e : t) {
        read(e);
    }
}

template <typename T>
void writeContainer(string delimiter, T &t) {
    for (const auto &e : t) {
        write(delimiter, e);
    }
    write("\n");
}
```
### 使用方法
```cpp
// 问题: 读取三个空间独立的整型数据，并在不同的行打印
	int x, y, z;
	read(x, y, z);
	write("\n", x, y, z);
	
// 用不同的数据类型也能运行 :)
	int n;
	string s;
	read(n, s);
	write(" ", s, "has length", n, "\n");
	
// 问题: 读取数组N个整型，打印到输出控制台。
	int N;
	read(N);
	vector<int> arr(N);
	readContainer(arr);
	writeContainer(" ", arr); // output: arr[0] arr[1] arr[2] ... arr[N - 1]
	writeContainer("\n", arr);
	/**
 * output:
 * arr[0]
 * arr[1]
 * arr[2]
 * ...
 * ...
 * ...
 * arr[N - 1]
 */
```


----------------


## C++中的装饰器和多参数

### YouTube 直播 Demo
- 在YouTube上查看在线demo [here on Rachit's channel](https://www.youtube.com/watch?v=ZhWyNjPGXD4).


### 一行打印尽可能多的变量
```cpp
template<typename ...T>
void printer(T&&... args) {
    ((cout << args << " "), ...);
}

int age = 25;
string name = "Rachit";
printer("I am", name, ',', age, "years old"); 
// ^ 这条语句打印输出如下
// I am Rachit, 25 years old
```

### C++ 中强大的装饰器函数
```cpp
template<typename F>
auto debug_func(const F& func) {
    return [func](auto &&...args) { // forward reference
        cout << "input = ";
        printer(args...);
        auto res = func(forward<decltype(args)>(args)...);
        cout << "res = " << res << endl;
        return res;
    };
}

debug_func(pow)(2, 3);
// ^ this automatically prints
// input = 2 3 res = 8
```

### 通过嵌套的方式利用装饰器函数
Lets define another decorator `beautify` as follows.
```cpp
template<typename F>
auto beautify(const F& func) {
    return [func](auto &&...args) { // 前向引用
        cout << "========" << endl;
        func(forward<decltype(args)>(args)...);
        cout << "========" << endl;
    };
}

beautify(debug_func(pow(2, 3)));
// ^ 现在的输出
// ========
// input = 2 3 res = 8
// ========
```
像这样嵌套的写泛型装饰器，你可以完成很多东西。
思考一下写一个 `log_time` 的装饰器，计算一个函数运行的时间。
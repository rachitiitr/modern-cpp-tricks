## Introduction

This is the list of modern CPP tricks often used in Coding Interviews and Competitive Programming.  
If you like Rachit's work, you can follow at -

- Discord - https://bit.ly/discord-rachit
- Programming YouTube Channel - https://bit.ly/rachityoutube

## Contents:

- [Powerful Logging and Debugging](#powerful-logging-and-debugging)
  - [How debug macros work?](#how-debug-macros-work)
  - [The Problem with this macro - its not scalable](#the-problem-with-this-macro---its-not-scalable)
  - [Solution using a powerful macro](#solution-using-a-powerful-macro)

## Powerful Logging and Debugging

### How debug macros work?

Straight to the point, I have often used the `debug` macro which stringifies the variable names and their values.

```cpp
#define deb(x) cout << #x << " " << x
int ten = 10;
deb(ten); // prints "ten = 10"
```

This is often useful in debugging.

### The Problem with this macro - its not scalable

However, when you have multiple variables to log, you end up with more `deb2` and `deb3` macros.

```cpp
#define deb(x) cout << #x << " " << x
#define deb2(x) cout << #x << " " << x << " "  << #y << " " << y
#define deb3(x, y, z) cout << #x << " " << x << " "  << #y << " " << y << " "  << #z << " " << z
```

This is not scalable.

### Solution using a powerful macro

Here is the solution using variadic macros and fold expressions,

```cpp
#define deb(...) logger(#__VA_ARGS__, __VA_ARGS__)
template<typename ...Args>
void logger(string vars, Args&&... values) {
    cout << vars << " = ";
    string delim = "";
    (..., (cout << delim << values, delim = ", "));
}

int xx = 3, yy = 10, xxyy = 103;
deb(xx); // prints "xx = 3"
deb(xx, yy, xxyy); // prints "xx, yy, xxyy = 3, 10, 103"
```

Printing vectors or any STL containers is pain again.

### Solution using an even more powerful macro ([Source](https://codeforces.com/blog/entry/79024))

Note that, this is a bit bulky and cryptic. Don't EVER touch this code.

```cpp
int recur_depth = 0;
#ifdef DEBUG
#define dbg(x)                                                                                                                         \
    {                                                                                                                                  \
        ++recur_depth;                                                                                                                 \
        auto x_ = x;                                                                                                                   \
        --recur_depth;                                                                                                                 \
        cerr << string(recur_depth, '\t') << "\033[91m" << __func__ << ":" << __LINE__ << "\t" << #x << " = " << x_ << "\033[39m" << endl; \
    }
#else
#define dbg(x)
#endif
template <typename Ostream, typename Cont>
typename enable_if<is_same<Ostream, ostream>::value, Ostream &>::type operator<<(Ostream &os, const Cont &v)
{
    os << "[";
    for (auto &x : v)
    {
        os << x << ", ";
    }
    return os << "]";
}
template <typename Ostream, typename... Ts>
Ostream &operator<<(Ostream &os, const pair<Ts...> &p)
{
    return os << "{" << p.first << ", " << p.second << "}";
}
```

If the argument is `int`, This prints the value. If it's an `iterator`, It prints those values neatly, seperated by commas. Oh, Did I tell you it prints the line number of the call too?

Also, You don't have to remove these statements before submission!

If you want to enable this macro, Pass `DEBUG` as compilation flag. See this below in action.

Instead of

```
g++ main.cpp
```

You do,

```
g++ -DDEBUG main.cpp
```

If this flag isn't included, All the debug function calls are silently ignored.
Refer [source](https://codeforces.com/blog/entry/79024) for more details.

# Error handling

* [Return codes](#return-codes)
* [Assertions](#assertions)

> Defensive programming doesn’t mean being defensive about your programming—“It does so work!” The idea is based on defensive driving. In defensive driving, you adopt the mind-set that you’re never sure what the other drivers are going to do. That way, you make sure that if they do something dangerous you won’t be hurt. You take responsibility for protecting yourself even when it might be the other driver’s fault. In defensive programming, the main idea is that if a routine is passed bad data, it won’t be hurt, even if the bad data is another routine’s fault. More generally, it’s the recognition that programs will have problems and modifications, and that a smart programmer will develop code accordingly.

(c) Steven C. McConnell "Code Complete" Chapter 8
  
Experience shows that if you just think about error handling, then the errors themselves become less.

## Return codes

If an error can occur in a method, or if we can get an error from another used method, then the method must return an error.  
We do not plan to use exceptions for historical reasons.  
In the simplest cases, this may be a return of a boolean value.
But it’s better to try to return the error code, this approach has many advantages:

* While writing methods, we think about what errors may occur to return the correct code.
* We can show the user the correct message, and suggest what the user can do to fix the problem.
* The user can tell us the error code, and then it will be easier for us to understand what is wrong (for example, in a ticket on the tracker)
* During debugging and in the log, we will see what is wrong, thereby quickly detect and fix the problem.

To return error codes, you must use the type `Ret`.  
`Ret` is a type containing an error code, possibly text and methods for convenient handling of errors.  

Example:

```cpp
mu::Ret checkFormat(const std::string& str)
{
    if (str.empty()) {
        return make_ret(Err::BadFormat);
    }
    ...
    return make_ret(Ret::Code::Ok);
}

```cpp
mu::Ret parse(const std::string& str)
{
    Ret ret = checkFormat(str);
    if (!ret) {
        return ret;
    }
    ...
    return make_ret(Ret::Code::Ok);
}
```

The enumerator Ret::Code contains common return codes and a range of codes for modules:

```cpp
class Ret
{
    ...
    enum class Code {
        Undefined       = -1,
        Ok              = 0,
        Error           = 1,

        GlobalFirst     = 10,
        GlobalLast      = 99,

        NotationFirst   = 100,
        NotationLast    = 499
    };
...
};
```

Each module has its own error pool. At the root of the module should be a file with the name `moduleerrors.h`, which should contain the error code of the module and `make_ret` function to create a return instance.
Example:

_notationerrors.h_

```cpp
namespace mu {
namespace domain {
namespace notation {
// 100 - 499
enum class Err {
    Undefined       = int(Ret::Code::Undefined),
    NoError         = int(Ret::Code::Ok),
    UnknownError    = int(Ret::Code::NotationFirst),

    // file
    FileUnknownError    = 110,
    FileNotFound        = 111,
    FileOpenError       = 112,
    FileBadFormat       = 113,
    FileUnknownType     = 114,
    FileNoRootfile      = 115,
    FileTooOld          = 116,
    FileTooNew          = 117,
    FileOld300Format    = 118,
    FileCorrupted       = 119,
    FileCriticalCorrupted = 120,
};

inline mu::Ret make_ret(Err e)
{
    return Ret(static_cast<int>(e));
}

}
}
}
```

## Assertions

Assertions check for software exception errors, that is, they check what should never happen. There is no need to use assertions that can occur during normal operation, for example, the correctness of the input data, because the input data can be any, and this is not a developer’s error. A typical use of assertions is for example checking for a non-null pointer, where it should be non-null.
Assets are usually placed at the beginning of the method and check the input arguments or the current state of the class.

__Important__: We adhere to the survival strategy, that is, an error in any place should not lead to the crash of the entire program, the program should not be like a balloon, in which any puncture leads to the balloon deflated. Therefore, it is not enough just to check the assert that will work in the debug build, but you also need to handle the error in the release build.

For convenient use of assert and error handling, you can use macros (defined in `log.h`):

* IF_ASSERT_FAILED(cond)
* IF_ASSERT_FAILED_X(cond, msg)

They contain `assert`, output to the log and `if` to handle errors in the release.

Example:

```cpp
bool doSomeThing(Thing* ptr)
{
    return ptr->do();
}

```

It is expected that the passed pointer should not be null. But what if for some unexpected reason it is null? it will crash and the program will close.  
To show that we are expecting a non-null pointer and detect an error, we can add an assertion.  

```cpp
bool doSomeThing(Thing* ptr)
{
    assert(ptr);
    return ptr->do();
}

```

Assert will only work in debug, and the program will crash so that the developer does not ignore the problem.  
But what will happen in the release? It will also crash and the program will close. For the user, this behaviour is a problem, user can not do anything about it. Moreover, we may lose user data, for example, unsaved changes in the notation.
Therefore, we must handle this situation for the release.

```cpp
bool doSomeThing(Thing* ptr)
{
    assert(ptr);
    if (!ptr) {
        LOGE() << "assert failed, ptr in null";
        return false;
    }
    return ptr->do();
}

```

The `IF_ASSERT_FAILED` macro does just that, it will check assert, output an error to the log and open `if` expressions

```cpp
void doSomeThing(Thing* ptr)
{
    IF_ASSERT_FAILED(ptr) {
        return false;
    }
    return ptr->do();
}

```

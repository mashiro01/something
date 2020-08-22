# c/c++ 中判断两数相加是否溢出

```cpp
#include <iostream>

using namespace std;

int isAddOverflow(int a, int b) {
    int c = 0;

    __asm {
        mov eax, a
        add eax, b
        jo overflowed
        xor eax, eax
        jmp no_overflowed

    overflowed:
        mov eax, 1
        mov c, eax

    no_overflowed:
    }


    return c;
}

int main() {
    cout << isAddOverflow(0x7fffffff, 1);

    return 0;
}
```

用汇编判读OF标志位是否变化即可得到

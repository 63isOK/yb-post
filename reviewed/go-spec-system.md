# 系统注意事项

## unsafe包

一个内置包，提供了一个些低级别编程工具，包括违反类型系统的操作。

使用unsafe的包，需要手动审核类型安全，也可能会缺少可移植性。

    package unsafe

    type ArbitraryType int  
        // shorthand for an arbitrary Go type; it is not a real type
    type Pointer *ArbitraryType

    func Alignof(variable ArbitraryType) uintptr
    func Offsetof(selector ArbitraryType) uintptr
    func Sizeof(variable ArbitraryType) uintptr

Pointer是一个指针，但\*Pointer不一定是ok的

Pointer和uintptr是可以相互转化的。

    var f float64
    bits = *(*uint64)(unsafe.Pointer(&f))

    type ptr unsafe.Pointer
    bits = *(*uint64)(ptr(&f))

    var p ptr = nil

Alignof/Sizeof 是取对齐边界和size

Offsetof是用于支持指针的算术运算的

## size和字节对其

    type                                 size in bytes

    byte, uint8, int8                     1
    uint16, int16                         2
    uint32, int32, float32                4
    uint64, int64, float64, complex64     8
    complex128                           16

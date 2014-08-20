# Swift Codeblocks Syntax Highlight

For [The Swift Programming Language 中文版](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/).

Everthing is Ok now, thanks [Samy Pessé](http://www.samypesse.fr/).

[Coding](https://coding.net) has swift syntax highlight in [Swift Codeblocks Syntax Highlight](http://swift.coding.io/), and [source code](https://coding.net/u/tj2/p/swift/git).

[Swift Codeblocks Syntax Highlight](http://muxuezi.gitbooks.io/swift-codeblocks/) on Gitbook is Ok.

~~There's a swfit syntax bug highlight on [Gitbook](https://www.gitbook.io) now, even though [Samy Pessé](http://www.samypesse.fr/) had upgraded the highlight version of [gitbook/package.json](https://github.com/GitbookIO/gitbook) to [8.1.0](https://highlightjs.org/), which support swfit syntax.~~

test:

```swift
extension OCBool: Equatable{
}

//支持等值判断运算符
func ==( left: OCBool, right: OCBool )->Bool{
    switch (left, right){
    case (.ocTrue, .ocTrue):
            return true
    default:
        return false
    }
}
//支持位与运算符
func &( left:OCBool, right: OCBool)->OCBool{

    if left{
        return right
    }
    else{
        return false
    }
}
//支持位或运算符
func |( left:OCBool, right: OCBool)->OCBool{

    if left{
        return true
    }
    else{
        return right
    }
}

//支持位异或运算符
func ^( left:OCBool, right: OCBool)->OCBool{
    return OCBool( left != right )
}
//支持求反运算符
@prefix func !( a:OCBool )-> OCBool{
    return a ^ true
}
//支持组合求与运算符
func &= (inout left:OCBool, right:OCBool ){
    left = left & right
}


var isHasMoney:OCBool = true
var isHasWife:OCBool = true
var isHasHealty:OCBool = true
var isHasLover:OCBool = true

isHasMoney != isHasHealty
isHasHealty == isHasMoney
isHasWife ^ isHasLover
isHasWife = !isHasLover

if (isHasMoney | isHasHealty) & isHasHealty{
    println( "坐看云起时")
}else
{
    println("千金散尽还复来")
}
```





[![Build Status](https://www.gitbook.io/button/status/book/muxuezi/swift-codeblocks)](https://www.gitbook.io/book/muxuezi/swift-codeblocks/activity)

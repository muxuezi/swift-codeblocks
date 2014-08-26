# Swift Codeblocks Syntax Highlight

Everthing is Ok now, thanks [Samy Pessé](http://www.samypesse.fr/).

Lastest version at [The Swift Programming Language 中文版](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/).

~~由于[The Swift Programming Language 中文版](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/)不能swift语言高亮.~~

Enjoy：

- [Swift Codeblocks Syntax Highlight](http://swift.coding.io/) on [Coding][0].

- [Swift Codeblocks Syntax Highlight](http://muxuezi.gitbooks.io/swift-codeblocks/) on [Gitbhub][4].

~~There's a swfit syntax bug highlight on[Gitbook][2]now, even though [Samy Pessé](http://www.samypesse.fr/) had upgraded the highlight version of [gitbook/package.json](https://github.com/GitbookIO/gitbook) to [8.1.0](https://highlightjs.org/), which support swfit syntax.~~

- [Coding][1]和[Gitbook][2]都通过[highlightjs][3]支持swift语法高亮;
- [Coding][0]和[Github][4]可以在代码管理页面直接语法高亮;
- [Gitbook][2]还不行，如[首页](https://www.gitbook.io/book/muxuezi/swift-codeblocks)不支持swift语法高亮。

如果不论PDF、MOBI、EPUB格式输出，则
```
Coding = Github + Gitbook
```

一般html解决办法见[highlightjs][3]，需要xcode.css、highlight.pack.js，并在head增加:

```html
<link rel="stylesheet" href="../assert/css/xcode.css">
<script src="../assert/js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```
[0]:https://coding.net/u/tj2/p/swift/git
[1]:https://coding.net/ "Coding"
[2]:https://www.gitbook.io "Gitbook"
[3]:https://highlightjs.org/usage/ "highlightjs"
[4]:https://github.com/muxuezi/swift-codeblocks/


<link rel="stylesheet" href="../assets/css/xcode.css">
<script src="../assets/js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

Test:

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

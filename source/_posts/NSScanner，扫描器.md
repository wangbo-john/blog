title: NSScanner，扫描器
entitle: 'NSScanner'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528013009
date: 2018-06-03 16:03:29
tags:
    - 苹果
    - objective-c
keywords: iOS, NSScanner
description: 来自2015.5.29的笔记：NSScanner，扫描器。
photos:
---

```
- (void)testScanNumberFromString
{
    NSString *str = @"98234hk323hello234你好";
    NSMutableString *numberString = [[NSMutableString alloc] init];
    
    NSScanner *scanner = [NSScanner scannerWithString:str];
    NSString *tempString;
    
    while (![scanner isAtEnd]) {
        [scanner scanUpToCharactersFromSet:[NSCharacterSet decimalDigitCharacterSet] intoString:nil];
        
        //收集数字
        [scanner scanCharactersFromSet:[NSCharacterSet decimalDigitCharacterSet] intoString:&tempString];;
        [numberString appendString:tempString];
        tempString = @"";
    }
    NSLog(@"@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ number is: %@",numberString);
}
```

使用

```
[scanner scanUpToCharactersFromSet:[NSCharacterSet newlineCharacterSet] intoString:&indexString]; //扫描一行
[scanner scanUpToString:@" scanover " intoString:&theString];  //从游标开始扫描，直到给定字符串为止。期间扫描的字符串存到theString
[scanner scanString:@"sanMe" intoString:NULL]; //直接扫描指定字符串

```


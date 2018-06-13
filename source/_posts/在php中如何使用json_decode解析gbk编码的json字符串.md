---
title: 在php中如何使用json_decode解析gbk编码的json字符串
date: 2016-06-27 16:00:00
categories: PHP
tags: PHP
---
今天遇到用json_decode解析gbk编码的字符串问题。参考了这篇文章.
大家都知道，json都是utf8编码的。json_encode后的字符串都是会变成”\u4fe1\u6d77\u9f99”格式。
如下面的代码：

输出结果为：”\u4fe1\u6d77\u9f99”

如果你有一个符合json格式的gbk编码的字符串，如何使用json_decode进行解析呢？

答案其实很简单，呵呵。就是把字符串转为utf8编码既可。

你可以在gbk编码的文件中执行如下脚本，看效果。

输出结果如下：
NULL string(9) “信海龙”

为什么只要转为utf8编码就可以呢？分析下源码。
在ext/json/json.c文件有json_decode的实现。代码如下：

```
PHP_JSON_API void php_json_decode(zval *return_value, char *str, int str_len, zend_bool assoc, long depth TSRMLS_DC) /* {{{ */
{
    int utf16_len;
    zval *z;
    unsigned short *utf16;
    JSON_parser jp;
 
    utf16 = (unsigned short *) safe_emalloc((str_len+1), sizeof(unsigned short), 1);
 
    utf16_len = utf8_to_utf16(utf16, str, str_len);
    if (utf16_len <= 0) {
        if (utf16) {
            efree(utf16);
        }
        JSON_G(error_code) = PHP_JSON_ERROR_UTF8;
        RETURN_NULL();
    }
 
    if (depth <= 0) {
        php_error_docref(NULL TSRMLS_CC, E_WARNING, "Depth must be greater than zero");
        efree(utf16);
        RETURN_NULL();
    }
 
    ALLOC_INIT_ZVAL(z);
    jp = new_JSON_parser(depth);
    if (parse_JSON(jp, z, utf16, utf16_len, assoc TSRMLS_CC)) {
        *return_value = *z;
    }
    else
    {
        double d;
        int type;
        long p;
 
        RETVAL_NULL();
        if (str_len == 4) {
            if (!strcasecmp(str, "null")) {
                /* We need to explicitly clear the error because its an actual NULL and not an error */
                jp->error_code = PHP_JSON_ERROR_NONE;
                RETVAL_NULL();
            } else if (!strcasecmp(str, "true")) {
                RETVAL_BOOL(1);
            }
        } else if (str_len == 5 && !strcasecmp(str, "false")) {
            RETVAL_BOOL(0);
        }
 
        if ((type = is_numeric_string(str, str_len, &p, &d, 0)) != 0) {
            if (type == IS_LONG) {
                RETVAL_LONG(p);
            } else if (type == IS_DOUBLE) {
                RETVAL_DOUBLE(d);
            }
        }
 
        if (Z_TYPE_P(return_value) != IS_NULL) {
            jp->error_code = PHP_JSON_ERROR_NONE;
        }
 
        zval_dtor(z);
    }
    FREE_ZVAL(z);
    efree(utf16);
    JSON_G(error_code) = jp->error_code;
    free_JSON_parser(jp);
}
```

注意方法中这行代码：
utf16_len = utf8_to_utf16(utf16, str, str_len);
也就是把utf8编码的串转换为utf16编码串，然后在调用parse_JSON方法解析。找个方法在JSON_parser.c中定义。

看注释，这个方法是需要接收utf16编码的。
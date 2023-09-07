# 断言ASSERTION
## EXPECT_ 与 ASSERT_
- EXPECT_
产生非致命错误，允许当前函数继续执行
- ASSERT_
产生致命错误，抛弃当前函数(注意后面释放部分不会执行)
## 显式成功和失败
- SUCCEED()
- FAIL()
产生致命错误
- ADD_FAILURE()
产生非致命错误
- ADD_FAILURE_AT(*`file_path`*, *`line_number`*)
产生非致命错误，给出具体文件和行数

## 广义断言
- EXPECT_THAT(*`value`*, *`matcher`*)
- ASSERT_THAT(*`value`*, *`matcher`*)
判断*`value`*符不符合*`matcher`*，具体啥查阅相关文档

## 布尔条件
- EXPECT_TRUE(*`condition`*)
- ASSERT_TRUE(*`condition`*)

- EXPECT_FALSE(*`condition`*)
- ASSERT_FALSE(*`condition`*)

## 二进制比较
### Equal
- EXPECT_EQ(*`val1`*, *`val2`*)
- ASSERT_EQ(*`val1`*, *`val2`*)

### Not equal
- EXPECT_NE(*`val1`*, *`val2`*)
- ASSERT_NE(*`val1`*, *`val2`*)

### LessThan
- EXPECT_LT(*`val1`*, *`val2`*)
- ASSERT_LT(*`val1`*, *`val2`*)

### LessThanEqual
- EXPECT_LE(*`val1`*, *`val2`*)
- ASSERT_LE(*`val1`*, *`val2`*)

### GreaterThan
- EXPECT_GT(*`val1`*, *`val2`*)
- ASSERT_GT(*`val1`*, *`val2`*)

### GreaterThanEqual
- EXPECT_GE(*`val1`*, *`val2`*)
- ASSERT_GE(*`val1`*, *`val2`*)

## 字符串比较
用于C字符串，`string`用EXPECT_EQ或ASSERT_EQ

- EXPECT_STREQ(*`str`*, *`str2`*)
- ASSERT_STREQ(*`str1`*, *`str2`*)

- EXPECT_STRNE(*`str`*, *`str2`*)
- ASSERT_STRNE(*`str1`*, *`str2`*)
### Case Equal
- EXPECT_STRCASEEQ(*`str`*, *`str2`*)
- ASSERT_STRCASEEQ(*`str1`*, *`str2`*)

- EXPECT_STRCASENE(*`str`*, *`str2`*)
- ASSERT_STRCASENE(*`str1`*, *`str2`*)

## 浮点数类型比较
- EXPECT_FLOAT_EQ(*`val1`*, *`val2`*)
- ASSERT_FLOAT_EQ(*`val1`*, *`val2`*)

- EXPECT_DOUBLE_EQ(*`val1`*, *`val2`*)
- ASSERT_DOUBLE_EQ(*`val1`*, *`val2`*)

### 相差范围内
- EXPECT_NEAR(*`val1`*, *`val2`*, *`abs_error`*)
- ASSERT_NEAR(*`val1`*, *`val2`*, *`abs_error`*)

# 测试夹具TEST FIXTURE
一个Test里面有Arrange, Act, Assert。测试夹具就是将同样的Arrange先初始化好，供Test_F用

继承`::testing::Test`
重写`SetUp()`和`TearDown()`
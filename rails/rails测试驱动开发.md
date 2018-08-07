[toc]
## 1. 引言 
继承自 Minitest::Test（ActiveSupport::TestCase 的超类）的类中定义的方法，只要名称以 test_ 开头（区分大小写），就是一个“测试”。此外，Rails 定义了 test 方法，它接受一个测试名称和一个块。下面两种测试效果相同
```
test "the truth" do
  assert true
end

def test_the_truth
  assert true
end
```
## 2. 断言
```
assert true
```
断言求值对象（或表达式），然后与预期结果比较。例如，断言可以检查：

- 两个值是否相等
- 对象是否为 nil
- 一行代码是否抛出异常
- 用户的密码长度是否超过 5 个字符

一个测试中可以有一个或多个断言，对断言的数量没有限制。只有全部断言都成功，测试才能通过。

## 3. 运行测试
```
rails test                                  //统一测试
rails test  testName                        //测试特定的测试文件
 bin/rails test testName.rb -n methodName   //测试特定文件的特定方法
 bin/rails test testName.rb:n               //运行某一行中的测试
```
## 4. 固件
```
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development
 
steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```
固件不是为了创建测试中用到的每一个对象，需要公用的默认数据时才应该使用。默认情况下，Rails 会自动加载 test/fixtures 目录中的所有固件。加载的过程分为三步：

- 从数据表中删除所有和固件对应的数据；
- 把固件载入数据表；
- 把固件中的数据转储成方法，以便直接访问

固件是 Active Record 实例。在测试用例中可以直接访问这个对象，因为固件中的数据会转储成测试用例作用域中的方法。
```
# 返回 david 固件对应的 User 对象
users(:david)
 
# 返回 david 的 id 属性
users(:david).id
 
# 还可以调用 User 类的方法
david = users(:david)
david.call(david.partner)
```
## 5. 模型测试
模型测试用于测试应用中的各个模型
```
 bin/rails generate test_unit:model article title:string body:text
```
## 6. 系统测试
系统测试用于测试用户与应用的交互，可以在真正的浏览器中运行，也可以在无界面浏览器中运行。系统测试建立在 Capybara 之上。
```
bin/rails generate system_test users
```
系统测试的配置都在application_system_test_case.rb 文件中，默认情况下，系统测试使用 Selenium 驱动在 Chrome 浏览器中运行，界面尺寸为 1400x1400。

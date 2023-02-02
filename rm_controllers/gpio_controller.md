## 1. init

1. 获取参数文件中的gpios，也就是gpio名字
2. 初始化一个实时发布者
3. 一个for循环，遍历每一个gpio
   - 每一个gpio定义一个state_handle，放到一个vector中
   - 通过state_handle获得gpio的name和value，并放到gpio_state_pub_->msg_
   - 通过state_handle获得gpio类型（输入或输出），放到gpio_state_pub_->msg_
   - 如果gpio是输出，会给这个gpio定义一个command_handle，用来发命令，并把这个句柄放到另一个vector中
4. 一个订阅者，接收指令

## 2. update

1. ```c++
   gpio_state_pub_->trylock()
   //要从实时循环发布数据，需要运行trylock()获取数据锁，这个函数是为了获取对msg的唯一访问权限，成功会返回true，也就是获得了对msg的访问权限
   ```

2. 遍历每一个gpio，把每一个gpio的value写到msg_.gpio_state[]

3. 写时间戳

4. 解锁并发布消息

## 3. setGpioCmd

1. 这是/command这个话题订阅者的回调函数
2. 遍历每一个gpio_command_handle，在msg中如果找到了对应的gpio name的话，就setcommand
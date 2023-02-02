## 1. SocketCAN::open(...)

1. boost::function<void(const can_frame& frame)> handler
   - boost::function类型的对象是一个函数指针，可以指向跟函数签名相同类型的函数，这里定义的函数的签名是“void(const can_frame& frame)”，也就是返回值为空，参数为const can_frame& frame的函数
   - 这个函数指针可以被多次调用

```c++
{
  reception_handler = std::move(handler);  //右值转换
  /* Request a socket，by calling function "socket(int af, int type, int protocol)"；
     三个参数意义为：IP地址类型、套接字类型、协议类型；
     这里创建的socket为：IP类型为控制器局域网、原始套接字、原始can协议
  */
  sock_fd_ = socket(PF_CAN, SOCK_RAW, CAN_RAW);
  if (sock_fd_ == -1)  //创建socket失败会返回-1
  {
    ROS_ERROR("Error: Unable to create a CAN socket");
    return false;
  }
  char name[16] = {};  // avoid stringop-truncation，16个字符的数组，避免数组过小导致字符串截断
  strncpy(name, interface.c_str(), interface.size()); //将指定大小的字符串复制到数组中；name：存放字符的数组、interface.c_str()：指向的字符串、interface.size()：从字符串复制的大小
  strncpy(interface_request_.ifr_name, name, IFNAMSIZ);
  // 通过ioctl函数获取网络接口的地址，获取失败会返回-1
  if (ioctl(sock_fd_, SIOCGIFINDEX, &interface_request_) == -1)
  {
    ROS_ERROR("Unable to select CAN interface %s: I/O control error", name);
    // 调用close函数，使不可用的socket失效
    close();
    return false;
  }
  // Bind the socket to the network interface
  address_.can_family = AF_CAN;//要连接的网络接口的IP地址类型
  address_.can_ifindex = interface_request_.ifr_ifindex; //网络接口的ip地址
  int rc = bind(sock_fd_, reinterpret_cast<struct sockaddr*>(&address_), sizeof(address_));  //将socket连接到网络接口
  if (rc == -1)
  {
    ROS_ERROR("Failed to bind socket to %s network interface", name);
    close();
    return false;
  }
  // 启动一个单独的、事件驱动的线程来接收can帧
  return startReceiverThread(thread_priority);
}
```



## 2. SocketCAN::startReceiverThread(int thread_priority)

```c++
{
  // Frame reception is accomplished in a separate, event-driven thread.
  // See also: https://www.thegeekstuff.com/2012/04/create-threads-in-linux/
  terminate_receiver_thread_ = false;  //中止接收线程的标志
    
  /*通过pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg)函数创建线程
   
    thread ：传出参数，线程创建成功后，子线程的线程ID被写到该变量中
	attr ：设置线程的属性，一般使用默认值（nullptr）
	start_routine ：函数指针，该函数为子线程执行的逻辑代码
	arg ：给第三个参数使用，传参
	
	创建线程成功，函数返回0，失败则返回一个错误号
  */
  int rc = pthread_create(&receiver_thread_id_, nullptr, &socketcan_receiver_thread, this);  
  if (rc != 0)  //说明创建线程失败
  {
    ROS_ERROR("Unable to start receiver thread");
    return false;
  }
  ROS_INFO("Successfully started receiver thread with ID %lu", receiver_thread_id_);
  sched_param sched{ .sched_priority = thread_priority };  //设置线程的参数
  pthread_setschedparam(receiver_thread_id_, SCHED_FIFO, &sched);  //用上面设置的参数，设置线程参
  return true;
}
```



## 3. SocketCan::socketcan_receiver_thread(void* argv)

```c++
{
  /*
   * The first and only argument to this function
   * is the pointer to the object, which started the thread.
   */
  auto* sock = (SocketCAN*)argv;
  // Holds the set of descriptors, that 'select' shall monitor
  fd_set descriptors;
  // Highest file descriptor in set
  int maxfd = sock->sock_fd_;
  // How long 'select' shall wait before returning with timeout
  struct timeval timeout
  {
  };
  // Buffer to store incoming frame
  can_frame rx_frame{};
  // Run until termination signal received
  sock->receiver_thread_running_ = true;
  while (!sock->terminate_receiver_thread_)
  {
    timeout.tv_sec = 1.;  // Should be set each loop
    // Clear descriptor set
    FD_ZERO(&descriptors);
    // Add socket descriptor
    FD_SET(sock->sock_fd_, &descriptors);
    // Wait until timeout or activity on any descriptor
    if (select(maxfd + 1, &descriptors, nullptr, nullptr, &timeout))
    {
      size_t len = read(sock->sock_fd_, &rx_frame, CAN_MTU);
      if (len < 0)
        continue;
      if (sock->reception_handler != nullptr)
        sock->reception_handler(rx_frame);
    }
  }
  sock->receiver_thread_running_ = false;
  return nullptr;
}
```



## 4. SocketCAN::write(can_frame* frame)

```c++
{
  if (!isOpen())  //socket没有打开
  {
    ROS_ERROR_THROTTLE(5., "Unable to write: Socket %s not open", interface_request_.ifr_name);
    return;
  }
  if (::write(sock_fd_, frame, sizeof(can_frame)) == -1)  //通过write(sock_fd_, frame, sizeof(can_frame))向can bus发送can帧
    ROS_DEBUG_THROTTLE(5., "Unable to write: The %s tx buffer may be full", interface_request_.ifr_name);
}
```
*读取超电数据流程**

- Referee::read()中调用super_capacitor_.read(rx_buffer_)
  - 遍历rx_buffer中每个字节，并调用dtpReceivedCallBack(k_i)
    - 调用SuperCapacitor::receiveCallBack



**some knowledge**

- [serial类](http://docs.ros.org/en/indigo/api/serial/html/classserial_1_1Serial.html)
  - avaliable()：返回buffer中的字节数
  - read()：从串口读取指定数量的字节到指定的缓冲区，返回的读取到的字节数




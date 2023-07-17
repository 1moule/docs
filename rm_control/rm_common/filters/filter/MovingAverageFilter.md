## 1. header file

```c++
template <typename T>
class MovingAverageFilter : public Filter<T>
{
public:
  explicit MovingAverageFilter(int num_data);
  ~MovingAverageFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  T* buffer_;
  int num_data_;
  int idx_;
  T sum_;
};
```



## 2. source file

```c++
template <typename T>
MovingAverageFilter<T>::MovingAverageFilter(int num_data) : num_data_(num_data), idx_(0), sum_(0.0)
{
  buffer_ = new T[num_data_];  //new一段空间给buffer
  memset((void*)buffer_, 0.0, sizeof(T) * num_data_);  //buffer填0
}

template <typename T>
void MovingAverageFilter<T>::input(T input_value)  //输入的值放到队位 & 删除队首
{
  sum_ -= buffer_[idx_];  //当idx_ < num_data时，这一项减的都是0，大于等于时才是减掉队首（虽然下标没变，但是按照一个动态的列表来看，减掉buffer[0]，buffer[1]就是队首，后面看法类似）
  sum_ += input_value;
  buffer_[idx_] = input_value;
  ++idx_;
  idx_ %= num_data_; //当idx_ < num_data时，返回就是他本身；idx_ >= num_data，模运算返回的结果可以保证他在数组范围内
}

template <typename T>
T MovingAverageFilter<T>::output()  //返回队列均值
{
  return sum_ / num_data_;
}

template <typename T>
void MovingAverageFilter<T>::clear()
{
  sum_ = 0.0;
  memset((void*)buffer_, 0.0, sizeof(T) * num_data_);
}

template <typename T>
MovingAverageFilter<T>::~MovingAverageFilter()
{
  delete[] buffer_;
}

template class MovingAverageFilter<double>;
template class MovingAverageFilter<float>;
```
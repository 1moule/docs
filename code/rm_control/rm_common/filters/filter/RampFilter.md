**before reading**

-  斜坡滤波我们目前只在chassis base中用到
- acc_为加速度，也是vt图的的斜率
- 

## 1. header file

```c++
template <typename T>
class RampFilter : public Filter<T>
{
public:
  RampFilter(T acc, T dt);
  ~RampFilter() = default;
  void input(T input_value);
  void clear();
  void clear(T last_value);
  void setAcc(T acc);  // without clear.
  T output();

private:
  T last_value_;
  T acc_;
  T dt_;
};
```



## 2. source file

```c++
template <typename T>
RampFilter<T>::RampFilter(T acc, T dt)
{
  //构造函数中设置初始的斜坡滤波器斜率以及时间微元dt
  acc_ = acc;
  dt_ = dt;
  RampFilter::clear();
}

template <typename T>
void RampFilter<T>::input(T input_value)
{
  last_value_ += minAbs(input_value - last_value_, acc_ * dt_);
}

template <typename T>
void RampFilter<T>::clear()
{
  last_value_ = 0.;
}

template <typename T>
void RampFilter<T>::clear(T last_value)
{
  last_value_ = last_value;
}

template <typename T>
void RampFilter<T>::setAcc(T acc)//设置斜率
{
  acc_ = acc;
}

template <typename T>
T RampFilter<T>::output()
{
  return last_value_;
}

template class RampFilter<float>;
template class RampFilter<double>;
```
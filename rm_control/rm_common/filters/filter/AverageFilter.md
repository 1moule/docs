## 1. header file

```c++
template <typename T>
class AverageFilter : public Filter<T>
{
public:
  AverageFilter(T dt, T t_const, T limit);
  ~AverageFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  T est_value_;
  T dt_;
  T t_const_;
  T limit_;
};
```



## 2. source file

```c++
template <typename T>
AverageFilter<T>::AverageFilter(T dt, T t_const, T limit) : dt_(dt), t_const_(t_const), limit_(limit)
{
  est_value_ = 0.;
}

template <typename T>
AverageFilter<T>::~AverageFilter()
{
  est_value_ = 0;
}

template <typename T>
void AverageFilter<T>::clear()
{
  est_value_ = 0.;
}

template <typename T>
void AverageFilter<T>::input(T input)
{
  T update_value = input - est_value_;
  if (fabs(update_value) > limit_)
  {
    update_value = 0.;
  }
  est_value_ += (dt_ / (dt_ + t_const_)) * update_value;
}

template <typename T>
T AverageFilter<T>::output()
{
  return est_value_;
}

template class AverageFilter<float>;
template class AverageFilter<double>;
```
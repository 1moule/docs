```c++
//滤波器的父类，下面的滤波器都是继承这个类
template <typename T>
class Filter
{
public:
  Filter() = default;
  virtual ~Filter() = default;
  virtual void input(T input_value) = 0;
  virtual T output() = 0;
  virtual void clear() = 0;
};

//巴特沃斯滤波器
template <typename T>
class ButterworthFilter : public Filter<T>
{
public:
  ButterworthFilter(int num_sample, T dt, T cutoff_frequency);
  ~ButterworthFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  T* mpBuffer_;
  int mCurIdx_;
  int mNumSample_;
  T mDt_;
  T mCutoffFreq_;
  T mValue_;
};

//数字低通滤波器
template <typename T>
class DigitalLpFilter : public Filter<T>
{
public:
  DigitalLpFilter(T w_c, T t_s);
  ~DigitalLpFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  T Lpf_in_prev_[2];
  T Lpf_out_prev_[2];
  T Lpf_in1_, Lpf_in2_, Lpf_in3_, Lpf_out1_, Lpf_out2_;
  T lpf_out_;
};

//均值滤波器
//给定一个固定长度的队列，每次将输入的值放到队尾，并将队首的值扔出，然后对队列中的值求算术平均即可获得滤波结果
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

//微分低通滤波器
template <typename T>
class DerivLpFilter : public Filter<T>
{
public:
  DerivLpFilter(T w_c, T t_s);
  ~DerivLpFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  T Lpf_in_prev_[2];
  T Lpf_out_prev_[2];
  T Lpf_in1_, Lpf_in2_, Lpf_in3_, Lpf_out1_, Lpf_out2_;
  T lpf_out_;
};

//
template <typename T>
class FF01Filter : public Filter<T>
{
public:
  FF01Filter(float t_s, float w_c);
  virtual ~FF01Filter();
  virtual void input(T input_value);
  virtual T output();
  virtual void clear();

private:
  T Lpf_in_prev_[2];
  T Lpf_out_prev_[2];
  T Lpf_in1_, Lpf_in2_, Lpf_in3_, Lpf_out1_, Lpf_out2_;
  T lpf_out_;
};

template <typename T>
class FF02Filter : public Filter<T>
{
public:
  FF02Filter(float t_s, float w_c);
  ~FF02Filter();
  void input(T input_value);
  T output();
  void clear();

private:
  T Lpf_in_prev_[2];
  T Lpf_out_prev_[2];
  T Lpf_in1_, Lpf_in2_, Lpf_in3_, Lpf_out1_, Lpf_out2_;
  T lpf_out_;
};

//均值滤波器
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

//斜坡滤波器
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

//一欧元滤波器
template <typename T>
class OneEuroFilter : public Filter<T>
{
public:
  OneEuroFilter(double _freq, T _mincutoff, T _beta, T _dcutoff);
  ~OneEuroFilter();
  void input(T input_value);
  T output();
  void clear();

private:
  double freq;
  bool firsttime;
  T mincutoff, beta, dcutoff;
  T x_prev, dhatxprev, hatxprev;
  T filtered_val;
};
```
# Isaac Gym Env

1. 创建python3.8虚拟环境（我用virtualenv）

   ```
   mkvitualenv env-name python=python3.8
   ```

2. 安装pytorch1.10的cuda-11.3版本

   ```bash
   pip install torch==1.10.1+cu113 torchvision==0.11.2+cu113 torchaudio==0.10.1 -f https://download.pytorch.org/whl/cu113/torch_stable.html
   ```

3. 安装Isaac Gym




# 一些问题

1. ```bash
   AttributeError: module 'numpy' has no attribute 'float'.`np.float` was a deprecated alias for the builtin `float`.
   ```

   解决：将numpy降级，可能导致matplotlib版本不兼容，matplotlib也要降级

   ```
   pip uninstall numpy
   pip install numpy==1.19.5
   pip install matplotlib==3.6.3  
   ```

2. ```
   ModuleNotFoundError: No module named 'tensorboard'
   ```

   解决：pip安装

   ```
   pip install tensorboard
   ```

3. ```
   AttributeError: module 'distutils' has no attribute 'version'
   ```

   解决：setuptools降级

   ```
   pip uninstall setuptools
   pip install setuptools==59.5.0
   ```

   


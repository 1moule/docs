# 内存对齐

1. 在使用open函数时，加入标签O_DIRECT需要进行内存对齐
2. method：（使用posix_memalign函数）

```c++
      if (posix_memalign((void**)&buffer, 4096, 40960) != 0)
      {
        printf("Errori in posix_memalign\n");
      }

//函数原型
int posix_memalign (void **memptr, size_t alignment, size_t size);
/*
memptr是分配好的内存空间的首地址
alignment是

成功后，返回的内存地址保存在memptr中，返回值为0
```


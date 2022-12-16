## 1. 配置git

1. git config --[global](https://so.csdn.net/so/search?q=global&spm=1001.2101.3001.7020) user.name "xx"    
2. git config --global user.email "xx@gmail.com" 

## 2. 生成ssh密钥

1. ```
   ssh-keygen -t rsa
   ```

2. ```typescript
   cat ~/.ssh/id_rsa.pub 复制所有内容
   ```

3. 打开github，打开SSH Keys，粘贴刚才复制的内容

   
# 对 git 提交进行签名

## 步骤一：生成 GPG Key

https://help.github.com/articles/generating-a-new-gpg-key/

## 步骤二：添加到 github 账户中

## 步骤三：设置 git 提交时进行签名

```shell
git config --global user.signingkey [Key ID]
git config --global commit.gpgsign true
git config --global gpg.program gpg
```

## 坑：

问题：
完成上述的操作后，进行签名提交报错

```
error: gpg failed to sign the data fatal: failed to write commit object
```

解决办法：

```shell
export GPG_TTY=$(tty)
```

为了方便可以添加到~/.profile


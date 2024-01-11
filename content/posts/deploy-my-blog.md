---
layout: post
comments: true
title: '自动化部署我的博客'
excerpt: （更像赛博公民地）使用赛博工具
categories:
  - 部署
  - 填坑
date: '2024-01-11T18:20:00+08:00'
---

之前一直对github的sign commit以及action有所耳闻，打算借这次部署博客的机会把两者都安排一下。

## Sign Commit

签名（Sign）代表着身份认证，那么我们需要PGP Key来作为我们的身份证明。

### 生成PGP Key

这部分参考了现成的科普博客[^pgp1][^pgp2][^pgp3][^pgp4]，此处只记录一些常用指令：
**记得生成签名用的子公密钥对，并收好主公密钥对，后者只用来签发子公密钥对。**

```bash
gpg --full-gen-key --expert  # 根据自己的需要生成gpg主密钥及加密/签名用的子密钥
gpg --edit-key  # 你生成的主密钥姓名或key id，退出时记得save
gpg --gen-revoke -ao revoke.pgp # 以防你丢失密钥控制权，记得生成撤销证书
gpg --export-secret-keys  # key id后加上“!”只导出主密钥
gpg --export-secret-subkeys  # 备份子密钥（平时使用）

# ~/.gnupg/gpg.conf  # 可以让你在`gpg -K`和`gpg -k`时少打几个字符
keyid-format long
with-fingerprint
```

### 签名

跟着[Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)和[Linux Foundation的GPG说明](https://docs.releng.linuxfoundation.org/en/latest/gpg.html)的指示很容易就完成了。值得注意的是说明中的`gpg2`在现在大抵已等价于`gpg`了（至少Archlinux如此），具体可`ls -lah | grep gpg`自行检验。同样地，这里只记录一些关键指令：

```bash
git config --global user.signingkey <GPG-KEY-ID>！  # 只使用子密钥
git config --global commit.gpgsign true
git config --global push.gpgsign true

gpg -a --export <GPG-KEY-ID>!  # 只导出子公钥

[ -f ~/.bashrc ] && echo -e '\nexport GPG_TTY=$(tty)' >> ~/.bashrc  # 确保远程登录你带GUI的主机时不会因无法弹出输入密码而报错
```

值得一提的是，如果你和我一样使用子公密钥对进行签名，上述两处加备注的行最后记得添加`！`，表示只使用子公密钥对信息（不影响你的指纹信息）。

## Actions

> GitHub Actions 是一种持续集成和持续交付 (CI/CD) 平台，可用于自动执行生成、测试和部署管道。 您可以创建工作流程来构建和测试存储库的每个拉取请求，或将合并的拉取请求部署到生产环境。[^gha]

这部分[Github官方的说明](https://docs.github.com/en/actions)给的很详细了，照着quick start走一遍就差不多知道该如何写自己的action配置了。

在[我的自动化部署配置](https://github.com/Kuro96/kuro96.github.io/blob/master/.github/workflows/hugo-deploy.yml)中，我使用了[easingthemes/ssh-deploy](https://github.com/easingthemes/ssh-deploy)（自建服务器）而不是[actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)（GitHub Pages）来部署我的博客，前者需要对linux（也可以是其他）服务器有一定的了解，【一种建议是】会使用[nginx](https://www.nginx.com/)和[acme.sh](https://github.com/acmesh-official/acme.sh)实现https网页的搭建；后者可以参考[GitHub的官方说明](https://pages.github.com/)。

## 关于这个博客的其他细节

本博客基于[Hugo](https://gohugo.io/)和[PaperMod](https://adityatelange.github.io/hugo-PaperMod/)搭建。部署过程大量参考了[xen0n](https://github.com/xen0n/xen0n.github.io)的配置。
其中有少量文章使用到了静态图片，我自己部署了[lsky-pro](https://github.com/lsky-org/lsky-pro)作为博客专用图床。

此外，由于使用了大厂的云服务器，暂时不考虑CDN。（又不是不能用

如果能帮到你构建自己的博客，那就再好不过了。

[^pgp1]: [2021年，用更现代的方法使用PGP（上）](https://ulyc.github.io/2021/01/13/2021%E5%B9%B4-%E7%94%A8%E6%9B%B4%E7%8E%B0%E4%BB%A3%E7%9A%84%E6%96%B9%E6%B3%95%E4%BD%BF%E7%94%A8PGP-%E4%B8%8A/)
[^pgp2]: [2021年，用更现代的方法使用PGP（中）](https://ulyc.github.io/2021/01/18/2021%E5%B9%B4-%E7%94%A8%E6%9B%B4%E7%8E%B0%E4%BB%A3%E7%9A%84%E6%96%B9%E6%B3%95%E4%BD%BF%E7%94%A8PGP-%E4%B8%AD/)
[^pgp3]: [2021年，用更现代的方法使用PGP（下）](https://ulyc.github.io/2021/01/26/2021%E5%B9%B4-%E7%94%A8%E6%9B%B4%E7%8E%B0%E4%BB%A3%E7%9A%84%E6%96%B9%E6%B3%95%E4%BD%BF%E7%94%A8PGP-%E4%B8%8B/)
[^pgp4]: [GPG 最佳实践](https://www.yangqi.show/posts/gpg-best-practices)
[^gha]: [了解 GitHub Actions](https://docs.github.com/zh/actions/learn-github-actions/understanding-github-actions)

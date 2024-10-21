---
title: '清理git中的大文件objects/pack'
publishedAt: '2024-06-27'
summary: '最近某个项目构建的时候，jenkins一直报错，问了运维才知道是项目文件太大了导致打包失败，所以花一下午排查下问题并记录'
image: '/blog-img/first-load-js/cover.webp'
rssImage: '/blog-img/first-load-js/cover.jpeg'
category: 'Tech'
---

<aside>
最近某个项目构建的时候，jenkins一直报错，问了运维才知道是项目文件太大了导致打包失败，所以花一下午排查下问题并记录。
</aside>

## 一、问题源：

经过排序是.git下pack文件过大了，足足2.5个G，也难怪会不让过。

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled.png)

### 所以…pack文件是啥?

> 在Git中，`pack`文件是Git用来存储和传输对象的一种方式。当Git仓库中的对象（如提交、树、文件等）数量增加时，Git会将这些对象打包成一个或多个`pack`文件，以减少存储空间和提高网络传输效率。

看了下，项目的提交记录和维护人员确实人多，好吧，那就从pack文件入手；

## 二、解决方案：

### 1、找到当前项目前五个最大的文件

```jsx
git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -5 | awk '{print$1}')"
```

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled%201.png)

这里看到是“release/report.json”文件占用最大，记录一下，下面用得着；

### 2、将大文件从git记录中移除

```jsx
git filter-branch --force --index-filter 'git rm -rf --cached --ignore-unmatch 大文件名' --prune-empty --tag-name-filter cat -- --all
```

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled%202.png)

### 3、彻底删除并清理空间

```jsx
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
```

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled%203.png)

### 4、查看优化效果

查看效果我们可以用`du -h -d 1 .git`来看当前git大小；

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled%204.png)

可以看到，经过处理已经优化到725MB了，最后同步到云端

### 5、同步到云端并删除本地仓库中已经不存在的远程跟踪分支

```jsx
git push --force
git remote prune origin
```

### 6、再次尝试构建

![Untitled](%E6%B8%85%E7%90%86git%E4%B8%AD%E7%9A%84%E5%A4%A7%E6%96%87%E4%BB%B6objects%20pack%2020898cdef2e24a99b6a7bc8679c6448d/Untitled%205.png)

可以看到，再次构建已经成功，下班！！！

# 总结归纳

没啥总结的，希望对大家有帮助，早点WLB~

# 📎 参考文章

[](https://www.jianshu.com/p/fe3023bdc825)

[【经验分享】git项目.git/objects/pack很大，clone很久，object文件清理\_.git pack文件过大-CSDN博客](https://blog.csdn.net/happy921212/article/details/134948152)

[git进阶 | 03 -如何彻底删除git中的大文件\_git删除大文件-CSDN博客](https://blog.csdn.net/Mculover666/article/details/125646383)

<aside>
有关GIT相关问题，欢迎您在底部评论区留言，一起交流~

</aside>

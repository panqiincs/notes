# How to rebuilt your blog

## Clone blog code

Clone the code from Github:

```
git clone git@github.com:panqiincs/hexo_blog.git hexo_blog
```

## Install hexo

If you are in a new computer, you must install hexo. Change directory to `hexo\_blog`, and run:

```
cd hexo_blog
npm install hexo-cli -g
npm install hexo --save
```

If it is to slow, use proxy.

Type `hexo -v`, you will see a lot of version information, success.

## Initialize hexo

Still in the `hexo\_blog` directory. You do not need to initialize by `hexo init`, just enter the path:

```
npm install
```

## Enjoy

```
hexo g
hexo s
```

## 

See the blog: http://localhost:4000


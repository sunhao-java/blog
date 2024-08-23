# [Blog](https://www.crazy-coder.cn/)
## Introduce

This is my blog site source code, where the source code is in the master branch and the generated static pages are in the gh-pages branch.

## Technology
1. Node.js (Base)
2. Hexo (Build blog site)
3. Markdown (Write blog)
4. See more in `package.json`

## How to develop
> You'll need the `nodejs` development environment, as well as a proper installation of `npm` or `cnpm` with the latest version. For `cnpm`, please refer to [CNPM](https://github.com/cnpm/cnpm)

1. Clone this repo to your PC.
```
git clone git@github.com:sunhao-java/blog.git
```

2. Install hexo and hexo-cli globally.(npm/cnpm)
```
npm install hexo hexo-cli -g
```

3. Install dependencies.
```
// root of blog repo
npm install
```

4. Instructions for use
```
1. hexo serve
2. hexo generate
3. For more, please visit hexo's official website at https://hexo.io/zh-cn/
```

## How to build with Github Actions

1. build

    see more at `.github/workflows/deploy.yml`

2. Configuration in gihub

    ![](https://imgs.lodsve.com:9000/images/2024/08/23/b044ba3f0e100b0f32a0bbb5c50c85e1.png)

3. Using custom domain names
    - Create a file named `CNAME` in the `blog/source` directory
    - The content is the domain name you will use
        ```
        www.crazy-coder.cn
        ```
    - Configure domain name resolution on your domain's provider's website
        ```
        Record type: CNAME
        Host Records: www
        Recorded Value: sunhao-java.github.io
        ```
    - sunhao-java.github.io
        - sunhao-java: your github username
        - github.io: fixed value
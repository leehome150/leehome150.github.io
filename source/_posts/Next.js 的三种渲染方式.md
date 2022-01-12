---
title: Next.js 的三种渲染方式
abbrlink: b2ae764
date: 2021-12-23 15:06:56
tags: Next.js
categories: Next.js
comments: true
---

> 客户端渲染(BSR)
> 静态页面生成(SSG)
> 服务端渲染(SSR)

- #### 客户端渲染(BSR)

顾名思义，客户端渲染只是在浏览器执行的渲染，我们一半用 JS、React、Vue 创建 HTML,举个例子：我们渲染一个列表:
我们先通过 usePost 这个自定义 Hook 通过 axios 返回一个的文件列表：

```
const usePosts = () => {
  const [posts, set_posts] = useState<Acticle[]>();
  useEffect(() => {
    axios.get("/api/v1/posts").then((res) => {
      set_posts(res.data);
    });
  }, []);
  return { posts };
};
```

我们然后把他渲染在页面上：

```
const posts = () => {
  const { posts } = usePosts();

  return (
    <div>
      <h1>文章列表</h1>
      {posts?.map((item) => {
        return <div key={item.id}>{item.title}</div>;
      })}
    </div>
  );
};
```

由上面我们可以知道客户端渲染通过<strong> AJAX</strong> 得到成功响应，然后渲染动态内容（静态内容与服务器渲染，比如：文章列表这几个字）。这也是我们现如今比较常用的渲染方式，因为大部分公司都是前后端分离。

他的缺点也很明显：

- 白屏
  但是我们在 set_posts(res.data)之前打个 debugger,发现页面看不到文章了。
- SEO 不友好
  我们打开源代码会发现看不到渲染的数据，只看到引用的 JS,因为搜索引擎不会执行 JS,只能看到 HTML。

这也有了下面两种渲染方式的用武之地：

- #### 静态文件生成(SSG)

背景：如果返回的文件与用户无关，都是一样的，那为什么不一次生成，然后发给每个人，N 次客户端渲染就变成了一次静态页面生成，这个就叫做动态内容静态化。

Next.js 提供 getStaticProps 这个 API,就可以直接通过 Props 传入：

```
const posts: NextPage<Props> = (props) => {
  const { posts } = props;
  return (
    <div className="content">
      <h1>文章列表</h1>
      <ul>
        {posts?.map((item) => {
          return (
            <Link href={`/posts/${item.title}`} key={item.title}>
              <a>
                <li key={item.title}>
                  {item.id}.{item.title}
                </li>
              </a>
            </Link>
          );
        })}
      </ul>
    </div>
  );
};

export default posts;

export const getStaticProps = async () => {
  const posts = await getPosts();
  return {
    props: { posts: JSON.parse(JSON.stringify(posts)) },
  };
};

```

这样的好处：我们不用 AJAX 也能拿到内容，不会白屏，而且我们查看源代码也能看到数据内容，也有利于 SEO。

- #### 服务端渲染（SSR）
  如果动态内容与用户先关联，较难提前静态化，这个时候我们可以使用服务端渲染。
  Next.js 提供 getStaticProps 这个 API,就可以直接通过 Props 传入,比如我们获取用户的浏览器类型：
  ```
  const Index: NextPage<Props> = (props) => {
  const { browser } = props;
  return (
    <div className={styles.container}>
      <h1>
        你的浏览器是{browser.name},版本为{browser.version}
      </h1>
      <img src={img} alt="" />
    </div>
  );
  };
  export default Index;
  export const getServerSideProps: GetServerSideProps = async (context) => {
  const ua = context.req.headers["user-agent"];
  const result = new UAParser(ua).getResult();
  return {
    props: {
      browser: result.browser,
    },
  };
  };
  ```
  服务端渲染的好处也是不会白屏，有利于 SEO 优化，相比于 SSG 能实现服务端渲染跟用户相关的动态内容。

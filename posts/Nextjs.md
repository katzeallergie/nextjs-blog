---
title: 'Next.jsのページレンダリング'
date: '2022-09-29'
---

# 1. Pre-rendering
SPAではページをロードする時、まず空の html を読み込んで JS ファイルも読み込んでその JS が画面をレンダリングする。

この問題を解決するため、 Next.js は基本的にすべてのページを Pre-rendering する。これはクライエント側の JS がレンダリングする代わりに、各ページに対して html を予め作っておくことを意味する。そうするとパーフォーマンスでも SEO でもより良い結果が出せる。

こうやって生成された html は必要最小限の JS コードに繋げられる。ページがブラウザにロードされるときにその JS コードも実行され、ページは完全にインタラクティブになる（このプロセスを Hydration と呼びます）。

# 2. SSG
## 2-1. SSGとは
SSG は Static Site Generation の略で、言葉通り**静的サイト**を生成する Pre-rendering 。この方式では **html がビルド時に生成され、各リクエストに再利用される**。最初にページがロードされる時にすぐ html を作る。

## 2-2. getStaticProps
もしロードするページが外部データに全然依存していないならデータをフェッチする必要もないのでただ単に静的な html を生成すればよい。

しかし、**外部データに依存する時**もある。公式サイトの例をそのまま借りると、ブログページで記事リストを読み込むときなどが該当する。こういう時はgetStaticProps関数を使う。下のように同じファイルで`getStaticProps`を使うと外部データをpropsとして渡すことができる。

```jsx
function Blog({ posts }) {
   return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  )
}

// この関数はビルド時に実行される
export async function getStaticProps() {
  // posts を取得するため外部 API endpoint を読み込む
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // { props: { posts } }を返すことで、
  // Blog コンポーネントはビルド時に`posts`を prop として受け取る
  return {
    props: {
      posts,
    },
  }
}
export default Blog
```

# 3. SSR
## 3-1. SSRとは
SSRは Server-side Rendering の略で、この方式では**htmlが各リクエストの度に生成される**。
リクエストが起こる度に html を生成するので、SSGより遅いが、Pre-renderingされたページを常に最新状態に維持することができるというメリットがある。

## 3-2. getServerSideProps
SSRでは`getServerSideProps`という関数を使う。
使い方は`getStaticProps`とほぼ同じ。違いはこちらはビルド時ではなく、全てのリクエストの度に実行される点。
```jsx
function Page({ data }) {
  // Render data...
}

// すべてのリクエストの度に実行される
export async function getServerSideProps() {
  // 外部APIからデータフェッチ
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  // props を通じて Page に data を渡す
  return { props: { data } }
}

export default Page
```
---
引用: https://zenn.dev/luvmini511/articles/1523113e0dec58

---
title: react hooks 中的循环调用
tags: [前端]
date: 2021-02-24
---
场景1：

```ts
const HomePage: React.FC<HomePageProps> = (props) => {

    console.log('组件内输出')

    const [text, setText] = useState<string>('原始文字')

    useEffect(() => {
        console.log('useEffect输出')
        setText('改变了文字')
    })
}
// 输出结果：
// 组件内输出     ---------- 初始化输出
// useEffect输出 ---------- 初始化输出
// 组件内输出     ---------- setState导致更新组件输出
// useEffect输出 ---------- setState导致组件更新输出
// 组件内输出     ---------- ?
```

---
title: umi css
date: 2020-05-25
private: true
---
# umi
## css modules 方案
umi 通用 css modules 方案讨论
https://github.com/umijs/umi/issues/1417

## 修改主题
    //.umirc.ts
    import darkTheme from '@ant-design/dark-theme';

    export default defaultConfig({ 
        theme: {
            ...darkTheme,
            // '@primary-color': '#1DA57A',
            // 'text-color': 'white',
        },
    })

## global.less
> note: 此文件内不需要加:global
src/global.less:

    .pad10{
        padding:10px;
    }
    td.ant-table-column-sort{
        background: inherit !important;
    }

## 组件class
    <Button className="pad10" onClick={addRow} type="primary">添加域</Button>

## 组件class 覆盖
一般用style=xxx 覆盖就可以了。

比如我们如果想覆盖overrid-ant-btn 子组件

    .override-ant-btn {
        :global(.ant-btn) {
            border-radius: 16px;
        }
    }

    <Parent className={styles.override-ant-btn}>
        <Btn className="ant-btn">
# 二年级数学互动自学课程

本仓库通过 GitHub Pages 发布第一、第二单元的单文件互动课程。

## 页面

- 网站首页（第一单元）：`index.html`
- 第一单元独立地址：`grade2-math-unit1.html`
- 第二单元独立地址：`grade2-math-unit2.html`

## 更新课程

将课程源文件复制到对应的发布文件后提交并推送：

```bash
cp ../grade2-math-unit1.html index.html
cp ../grade2-math-unit1.html grade2-math-unit1.html
cp ../grade2-math-unit2.html grade2-math-unit2.html
git add index.html grade2-math-unit1.html grade2-math-unit2.html
git commit -m "Update course content"
git push
```

第一单元同时复制为 `index.html`，因此访问站点根地址会直接进入第一单元。

## 本地预览

```bash
python3 -m http.server 8000
```

然后访问 `http://localhost:8000/`。

## 数据说明

课程进度保存在访问者浏览器的 `localStorage` 中，不会上传到服务器，也不会跨设备同步。

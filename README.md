# 二年级数学互动自学课程

本仓库通过 GitHub Pages 发布互动课程。

**在线课程首页：[打开二年级数学互动自学课程](https://kevinxie120.github.io/grade2-math-course/)**

## 页面

- 课程目录首页：`index.html`
- 第一单元独立地址：`grade2-math-unit1.html`
- 第二单元独立地址：`grade2-math-unit2.html`
- 第三单元独立地址：`grade2-math-unit3.html`
- 第四单元独立地址：`grade2-math-unit4.html`
- 第五单元独立地址：`grade2-math-unit5.html`
- 第六单元独立地址：`grade2-math-unit6.html`
- 第七单元独立地址：`grade2-math-unit7.html`
- 第八单元独立地址：`grade2-math-unit8.html`
- 第九单元独立地址：`grade2-math-unit9.html`
- 第十单元独立地址：`grade2-math-unit10.html`

## 朗读功能

十个单元右上角的“朗读”按钮第一次点击开始朗读，再次点击立即停止。自然读完或发生语音错误后，按钮会自动恢复为“朗读”。

## 更新课程

将课程源文件复制到对应的发布文件后提交并推送：

```bash
cp ../grade2-math-unit1.html grade2-math-unit1.html
cp ../grade2-math-unit2.html grade2-math-unit2.html
cp ../grade2-math-unit3.html grade2-math-unit3.html
cp ../grade2-math-unit4.html grade2-math-unit4.html
cp ../grade2-math-unit5.html grade2-math-unit5.html
cp ../grade2-math-unit6.html grade2-math-unit6.html
cp ../grade2-math-unit7.html grade2-math-unit7.html
cp ../grade2-math-unit8.html grade2-math-unit8.html
cp ../grade2-math-unit9.html grade2-math-unit9.html
cp ../grade2-math-unit10.html grade2-math-unit10.html
git add grade2-math-unit1.html grade2-math-unit2.html grade2-math-unit3.html grade2-math-unit4.html grade2-math-unit5.html grade2-math-unit6.html grade2-math-unit7.html grade2-math-unit8.html grade2-math-unit9.html grade2-math-unit10.html
git commit -m "Update course content"
git push
```

`index.html` 是独立课程目录。更新单元课程时只复制对应的 `grade2-math-unitN.html`，不要用第一单元覆盖首页。

## 本地预览

```bash
python3 -m http.server 8000
```

然后访问 `http://localhost:8000/`。

## 数据说明

课程进度保存在访问者浏览器的 `localStorage` 中，不会上传到服务器，也不会跨设备同步。

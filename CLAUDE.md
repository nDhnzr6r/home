# CLAUDE.md — Home 项目

## 项目概述
个人主页前端项目，Vue 3 + Vite，基于 imsyy/home 二次开发。

## 技术栈
- Vue 3 + Vite + Pinia
- Element Plus + Swiper + APlayer
- Axios + Sass + PWA

## 关键文件
- `src/views/MoreSet/index.vue` — 更新日志（`upData` 变量）
- `package.json` — 版本号（`version` 字段）
- `src/components/Weather.vue` — 天气组件
- `src/components/Background.vue` — 壁纸系统
- `src/utils/cursor.js` — 自定义鼠标特效
- `src/utils/getTime.js` — 纪念日灰度模式
- `.env` — 高德 Key 等环境变量

## 更改时
- 在 `upData` 数组**最前面**插入新版本对象：`{ version: "x.y.z", new: ["新增项"], fix: ["修复项"] }`
- 更新 `package.json` 的 `version` 版本号

## 运行
- 开发：`npm run dev`，运行在 `http://localhost:3000`
- 构建：`npm run build`

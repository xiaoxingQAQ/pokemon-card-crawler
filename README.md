# 宝可梦卡片爬虫

使用 Puppeteer v24.10.0 开发的宝可梦卡片数据爬虫，用于抓取 https://asia.pokemon-card.com 网站上的卡片信息。

## 功能特性

- ✅ 启动 Puppeteer 浏览器并打开目标页面
- ✅ 等待页面加载完成，选取 `rightColumn` 元素
- ✅ 获取所有 `card` 类的 `<li>` 元素
- ✅ 提取每个卡片的详情页链接
- ✅ 将相对链接转换为完整链接
- ✅ 逐个访问卡片详情页面并提取完整数据
- ✅ 支持能量类型图片到中文文本的映射
- ✅ 提取卡片的所有详细信息（名称、属性、招式、图鉴等）
- ✅ 自动下载卡片高清图片到本地
- ✅ 渐进式写入 JSONL 格式文件（实时保存，断点续传友好）
- ✅ 图鉴编号自动补零格式化（如：0001、0025）
- ✅ 完善的错误处理和资源清理机制

## 安装依赖

```bash
cd puppeteer
npm install
```

## 使用方法

### 基础运行
```bash
npm start
```

### 开发模式（带调试）
```bash
npm run dev
```

## 代码结构

```
puppeteer/
├── package.json        # 项目配置和依赖
├── index.js           # 主爬虫代码
└── README.md          # 说明文档
```

## 主要类和方法

### PokemonCardCrawler 类

- `init()` - 初始化浏览器和页面
- `getCardLinks()` - 获取卡片列表页面的所有卡片链接
- `visitCardDetail(cardUrl)` - 访问单个卡片详情页面
- `visitAllCardDetails(cardLinks)` - 批量访问所有卡片详情页面
- `cleanup()` - 清理浏览器资源

## 配置选项

### 浏览器配置
- `headless: false` - 显示浏览器窗口（便于调试）
- `defaultViewport` - 设置视口大小为 1280x720
- 包含防检测的启动参数

### 请求策略
- 使用 `networkidle2` 等待策略确保页面完全加载
- 内置随机延迟（1-3秒）避免被反爬虫
- 设置合理的超时时间（30秒）

## 运行流程

1. **启动阶段**：初始化 Puppeteer 浏览器
2. **页面加载**：打开目标页面并等待加载完成
3. **元素定位**：查找 `div.rightColumn` 元素
4. **链接提取**：获取所有 `li.card` 中的 `<a>` 标签 href 属性
5. **链接转换**：将相对路径转换为完整 URL
6. **结果输出**：在控制台显示所有找到的卡片链接

## 数据结构

每个卡片的数据结构如下：

```json
{
  "card_id": "hk00008497",
  "card_type": "基礎",
  "name": {
    "zh": "保母曼波",
    "en": null
  },
  "dex_info": {
    "national_no": "0594",
    "category": "看護寶可夢",
    "height": "1.2m",
    "weight": "31.6kg"
  },
  "stats": {
    "hp": 120
  },
  "attributes": {
    "weakness": "雷×2",
    "resistance": "無",
    "retreat_cost": 2
  },
  "abilities": [
    {
      "name": "衝浪",
      "type": "水",
      "damage": 30,
      "effect": null
    }
  ],
  "card_info": {
    "illustrator": "Shinji Kanda",
    "card_number": "G SVAW F 001/023",
    "rarity": "F稀有",
    "set": "朱&紫"
  },
  "flavor_text": "會用魚鰭溫柔地抱住受傷或是虛弱的寶可夢，並用特殊的黏膜加以治療。",
  "appearance": null,
  "image_path": "cardImages/hk00008497.jpg"
}
```

## 示例输出

```
=== 宝可梦卡片数据爬虫启动 ===

进行初始化设置...
正在启动 Puppeteer...
Puppeteer 启动完成
正在打开页面: https://asia.pokemon-card.com/hk/card-search/list/?expansionCodes=SVAW
页面加载完成，等待内容渲染...
找到 rightColumn 元素，开始提取卡片链接...
成功提取到 23 个卡片链接:
1. https://asia.pokemon-card.com/hk/card-search/detail/8497/
2. https://asia.pokemon-card.com/hk/card-search/detail/8498/
...

--- 开始逐个处理卡片，结果将实时写入 pokemon_cards_SVAW.jsonl ---

[1/23] ✅ 已抓取并保存: 保母曼波 (hk00008497)
[2/23] ✅ 已抓取并保存: 螢光魚 (hk00008498)
[3/23] ✅ 已抓取并保存: 霓虹魚 (hk00008499)
...

🎉 全部操作完成！数据已保存在 pokemon_cards_SVAW.jsonl，图片已保存在 cardImages 目录。

正在关闭浏览器...
浏览器已关闭

=== 爬虫运行完成 ===
```

## 能量类型映射

爬虫内置了能量类型图片到中文文本的映射表：

```javascript
const energyMap = {
    'Water.png': '水',
    'Lightning.png': '電',
    'Colorless.png': '無色',
    'Fighting.png': '鬥',
    'Psychic.png': '超',
    'Fire.png': '火',
    'Grass.png': '草',
    'Darkness.png': '惡',
    'Metal.png': '鋼',
    'Dragon.png': '龍',
    'Fairy.png': '妖',
};
```

## 自定义配置

### 修改目标网址

在 `PokemonCardCrawler` 构造函数中修改 `targetUrl`：

```javascript
constructor() {
    // ... 其他配置
    this.targetUrl = 'https://asia.pokemon-card.com/hk/card-search/list/?expansionCodes=YOUR_CODE';
}
```

### 修改输出文件名

在 `run()` 方法中调用 `saveData()` 时传入自定义文件名：

```javascript
await this.saveData('custom_filename.json');
```

### 调整爬取延迟

在 `visitAllCardDetails()` 方法中修改延迟时间：

```javascript
// 添加延迟避免被反爬虫（1-3秒随机延迟）
await this.delay(1000 + Math.random() * 2000);
```

## 注意事项

- 请遵守网站的 robots.txt 和使用条款
- 建议在请求间添加适当延迟，避免对服务器造成过大压力
- 当前版本仅抓取链接，具体数据抓取逻辑需要根据实际需求开发
- 如遇到反爬虫机制，可能需要调整请求策略

## 技术栈

- **Node.js** - 运行环境
- **Puppeteer v24.10.1** - 无头浏览器控制
- **ES Modules** - 模块系统 
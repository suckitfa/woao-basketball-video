# 我奥篮球视频数据提取器 🏀



## 📢 项目简介

我奥篮球视频数据提取器是一款专为篮球爱好者、教练和分析师设计的强大工具。它能够从我奥篮球平台轻松提取比赛回放、精彩集锦等视频数据，让您的篮球观赛和分析体验更上一层楼。

### 🌟 主要特性

- **自动提取**: 智能获取我奥篮球平台的比赛视频和回放链接
- **一键复制**: 轻松复制视频链接，方便分享或保存
- **详细信息**: 显示视频时长、创建时间和有效期等关键信息
- **用户友好**: 简洁的界面设计，操作直观易用
- **高可靠性**: 采用双重数据获取机制，确保稳定性

## 🚀 快速开始

1. 确保您的浏览器已安装 Tampermonkey 插件。
2. 点击 [这里](#) 安装脚本。
3. 打开我奥篮球的比赛详情页面。
4. 享受便捷的视频数据提取体验！

## 💡 使用场景

- **球迷**: 轻松获取比赛回放，重温精彩瞬间
- **教练**: 快速访问完整比赛视频，进行战术分析
- **内容创作者**: 方便地获取素材，制作精彩的篮球内容

## 📈 为什么选择我们的工具？

1. **节省时间**: 自动化的数据提取过程，让您专注于观看和分析
2. **增强分析**: 轻松获取全场回放，深入研究比赛细节
3. **提升体验**: 简化视频分享过程，增强社交互动
4. **保护隐私**: 本地数据处理，确保您的使用习惯不被泄露

## 📚 技术细节

- 使用 JavaScript 开发
- 兼容主流浏览器
- 采用 XHR 拦截技术
- 支持备用 API 调用

## 🤝 贡献指南

我们欢迎并感谢任何形式的贡献！如果您有任何改进意见或遇到问题，请：

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建一个 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

## 📞 联系我们

如果您有任何问题或建议，欢迎联系我们：

- 项目地址：[https://github.com/yourusername/woao-basketball-extractor](https://github.com/yourusername/woao-basketball-extractor)
- 电子邮件：your.email@example.com

---

💖 如果您喜欢这个项目，别忘了给我们一个星标！

#我奥篮球 #篮球视频 #回放 #数据提取 #篮球分析
### 全部代码

```js
// ==UserScript==
// @name         我奥-篮球完整优化版篮球赛程数据提取器
// @namespace    http://tampermonkey.net/
// @version      1.1
// @description  使用XHR劫持和备用方法从API中提取篮球赛程数据，提供更可靠的数据加载和优化的用户体验
// @match        https://m.lanqiu.woaoo.net/schedule/detail*
// @grant        GM_addStyle
// @grant        GM_setClipboard
// @grant        GM_xmlhttpRequest
// ==/UserScript==

(function() {
    'use strict';

    let panelVisible = false;
    let dataContainer = null;
    let scheduleData = null;
    let dataLoaded = false;

    GM_addStyle(`
        #toggleButton, #retryButton {
            position: fixed;
            z-index: 10000;
            padding: 8px 15px;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 14px;
            transition: background-color 0.3s;
        }
        #toggleButton {
            top: 10px;
            right: 10px;
            background-color: #4CAF50;
        }
        #toggleButton:hover {
            background-color: #45a049;
        }
        #retryButton {
            top: 50px;
            right: 10px;
            background-color: #f0ad4e;
        }
        #retryButton:hover {
            background-color: #ec971f;
        }
        #dataPanel {
            position: fixed;
            top: 90px;
            right: 10px;
            width: 300px;
            max-height: calc(100vh - 110px);
            overflow-y: auto;
            background-color: rgba(255, 255, 255, 0.95);
            border: 1px solid #ccc;
            padding: 15px;
            border-radius: 8px;
            z-index: 9999;
            font-size: 14px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            display: none;
        }
        .copyButton {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            transition: background-color 0.3s, transform 0.1s;
        }
        .copyButton:hover {
            background-color: #0056b3;
        }
        .copyButton:active {
            transform: scale(0.95);
        }
        .copyButton.copied {
            background-color: #28a745;
        }
        .dataItem {
            margin-bottom: 15px;
            padding: 12px;
            background-color: #f8f9fa;
            border-radius: 6px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }
    `);

    // 劫持 XHR
    function interceptXHR() {
        const XHR = XMLHttpRequest.prototype;
        const open = XHR.open;
        const send = XHR.send;

        XHR.open = function(method, url) {
            this._url = url;
            return open.apply(this, arguments);
        };

        XHR.send = function() {
            this.addEventListener('load', function() {
                if (this._url.includes('/appapi/scheduleHighlightVideo/getSchedulePlaybackAndHighlights')) {
                    const response = JSON.parse(this.responseText);
                    if (response.code === 200 && response.data) {
                        handleData(response.data);
                    }
                }
            });
            return send.apply(this, arguments);
        };
    }

    // 处理获取到的数据
    function handleData(data) {
        if (!dataLoaded) {
            scheduleData = data;
            createDataPanel(scheduleData);
            createToggleButton();
            dataLoaded = true;
            if (document.getElementById('retryButton')) {
                document.getElementById('retryButton').remove();
            }
        }
    }

    // 备用方法：直接获取数据
    function fetchDataDirectly() {
        const scheduleId = extractScheduleId();
        if (scheduleId) {
            const apiUrl = `https://gatewayapi.woaolanqiu.cn/appapi/scheduleHighlightVideo/getSchedulePlaybackAndHighlights?scheduleId=${scheduleId}`;
            GM_xmlhttpRequest({
                method: "GET",
                url: apiUrl,
                onload: function(response) {
                    if (response.status === 200) {
                        const data = JSON.parse(response.responseText);
                        if (data.code === 200 && data.data) {
                            handleData(data.data);
                        } else {
                            console.error('API返回错误:', data.message);
                        }
                    } else {
                        console.error('请求失败，状态码:', response.status);
                    }
                },
                onerror: function(error) {
                    console.error('请求出错:', error);
                }
            });
        } else {
            console.error('URL中未找到scheduleId');
        }
    }

    // 从URL中提取scheduleId
    function extractScheduleId() {
        const url = new URL(window.location.href);
        return url.searchParams.get('scheduleId');
    }

    // 格式化时长
    function formatDuration(seconds) {
        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);
        const remainingSeconds = Math.round(seconds % 60);
        return `${hours > 0 ? hours + '时' : ''}${minutes}分${remainingSeconds}秒`;
    }

    // 复制到剪贴板
    function copyToClipboard(name, url, button) {
        const copyText = `${name}：${url}`;
        GM_setClipboard(copyText, 'text');
        button.textContent = '已复制!';
        button.classList.add('copied');
        setTimeout(() => {
            button.textContent = '复制链接';
            button.classList.remove('copied');
        }, 2000);
    }

    // 创建并显示数据面板
    function createDataPanel(data) {
        if (dataContainer) {
            document.body.removeChild(dataContainer);
        }

        dataContainer = document.createElement('div');
        dataContainer.id = 'dataPanel';
        document.body.appendChild(dataContainer);

        let content = '<h2 style="margin-top:0; color: #333;">篮球赛程数据</h2>';

        // 处理所有回放
        if (data.playbacks && data.playbacks.length > 0) {
            content += '<h3 style="color: #555;">回放</h3>';
            data.playbacks.forEach((playback, index) => {
                content += `
                    <div class="dataItem">
                        <p><strong style="color: #333;">${playback.partName}</strong></p>
                        <p style="color: #666;">时长: ${formatDuration(playback.duration)}</p>
                        <button class="copyButton" onclick="copyToClipboard('${playback.partName}', '${playback.videoUrl}', this)">复制链接</button>
                    </div>
                `;
            });
        } else {
            content += '<p style="color: #666;">没有可用的回放</p>';
        }

        // 处理全场比赛集锦
        if (data.wholeHighlightCollection) {
            const highlight = data.wholeHighlightCollection;
            const gameNameAndScore = `${highlight.homeTeamName} ${highlight.homeTeamScore} - ${highlight.awayTeamScore} ${highlight.awayTeamName}`;
            content += `
                <h3 style="color: #555;">全场比赛集锦</h3>
                <div class="dataItem">
                    <p><strong style="color: #333;">${gameNameAndScore}</strong></p>
                    <p style="color: #666;">时长: ${formatDuration(highlight.duration)}</p>
                    <p style="color: #666;">创建时间: ${highlight.createTime}</p>
                    <p style="color: #666;">有效期还剩: ${highlight.remainingValidDays}天</p>
                    <button class="copyButton" onclick="copyToClipboard('全场比赛集锦 ${gameNameAndScore}', '${highlight.url}', this)">复制链接</button>
                </div>
            `;
        } else {
            content += '<p style="color: #666;">没有可用的全场比赛集锦</p>';
        }

        dataContainer.innerHTML = content;

        // 添加复制功能
        unsafeWindow.copyToClipboard = copyToClipboard;
    }

    // 切换面板显示/隐藏
    function togglePanel() {
        if (dataContainer) {
            panelVisible = !panelVisible;
            dataContainer.style.display = panelVisible ? 'block' : 'none';
        }
    }

    // 创建触发按钮
    function createToggleButton() {
        if (!document.getElementById('toggleButton')) {
            const button = document.createElement('button');
            button.id = 'toggleButton';
            button.textContent = '显示/隐藏数据';
            button.onclick = togglePanel;
            document.body.appendChild(button);
        }
    }

    // 添加重试按钮
    function addRetryButton() {
        if (!document.getElementById('retryButton')) {
            const retryButton = document.createElement('button');
            retryButton.id = 'retryButton';
            retryButton.textContent = '重新加载数据';
            retryButton.onclick = fetchDataDirectly;
            document.body.appendChild(retryButton);
        }
    }

    // 直接劫持XHR
    interceptXHR();
    // 主函数
    function main() {

        // 如果5秒后仍未加载数据，尝试直接获取并添加重试按钮
        setTimeout(() => {
            if (!dataLoaded) {
                fetchDataDirectly();
                addRetryButton();
            }
        }, 5000);
    }

    // 当页面加载完成时运行主函数
    window.addEventListener('load', main);
})();
```
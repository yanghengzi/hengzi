<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>内网批量PING工具 (Linux版本)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    
    <!-- 配置Tailwind自定义主题 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#3B82F6',
                        secondary: '#10B981',
                        neutral: '#1F2937',
                        accent: '#F59E0B',
                        success: '#22C55E',
                        warning: '#FBBF24',
                        danger: '#EF4444',
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    
    <!-- 自定义工具类 -->
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .text-shadow {
                text-shadow: 0 2px 4px rgba(0,0,0,0.1);
            }
            .transition-all-300 {
                transition: all 300ms ease-in-out;
            }
            .bg-gradient-blue {
                background: linear-gradient(135deg, #3B82F6 0%, #2563EB 100%);
            }
            .card-hover {
                @apply hover:shadow-lg hover:-translate-y-1 transition-all duration-300;
            }
        }
    </style>
    
    <style>
        body {
            background-color: #f9fafb;
            background-image: 
                radial-gradient(#e5e7eb 1px, transparent 1px),
                radial-gradient(#e5e7eb 1px, transparent 1px);
            background-size: 50px 50px;
            background-position: 0 0, 25px 25px;
        }
        
        /* 动画效果 */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .animate-fadeIn {
            animation: fadeIn 0.5s ease-out forwards;
        }
        
        /* 滚动条样式 */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 4px;
        }
        
        ::-webkit-scrollbar-thumb {
            background: #c5c5c5;
            border-radius: 4px;
        }
        
        ::-webkit-scrollbar-thumb:hover {
            background: #a8a8a8;
        }
    </style>
</head>
<body class="min-h-screen font-sans text-neutral">
    <!-- 页面头部 -->
    <header class="bg-white shadow-md">
        <div class="container mx-auto px-4 py-4">
            <div class="flex items-center justify-between">
                <h1 class="text-[clamp(1.5rem,3vw,2.5rem)] font-bold text-primary text-shadow">
                    <i class="fa fa-network-wired mr-3"></i>内网批量PING工具 (Linux版本)
                </h1>
                <div class="text-sm text-gray-500">
                    <i class="fa fa-info-circle mr-1"></i>适用于Linux环境的快速网络检测
                </div>
            </div>
        </div>
    </header>

    <!-- 主要内容 -->
    <main class="container mx-auto px-4 py-8">
        <div class="max-w-4xl mx-auto">
            <!-- 工具介绍卡片 -->
            <div class="bg-white rounded-xl shadow-md p-6 mb-8 animate-fadeIn">
                <div class="flex items-start">
                    <div class="bg-primary/10 p-3 rounded-lg mr-4">
                        <i class="fa fa-lightbulb-o text-primary text-xl"></i>
                    </div>
                    <div>
                        <h2 class="text-xl font-semibold mb-2">使用说明</h2>
                        <p class="text-gray-600 mb-3">
                            本工具帮助您在Linux系统上批量执行PING命令，检测内网设备连通性。由于浏览器安全限制，工具会生成可复制的命令，您需要在终端中执行。
                        </p>
                        <div class="bg-yellow-50 border-l-4 border-yellow-400 p-3 rounded">
                            <div class="flex">
                                <div class="flex-shrink-0">
                                    <i class="fa fa-exclamation-triangle text-yellow-500"></i>
                                </div>
                                <div class="ml-3">
                                    <p class="text-sm text-yellow-700">
                                        <strong>提示：</strong> 在Linux系统中，您可以使用快捷键 <kbd class="px-2 py-1 bg-gray-200 rounded text-xs">Ctrl + Alt + T</kbd> 打开终端，然后粘贴生成的命令执行。如果下载脚本文件，请记得使用 <code class="bg-gray-200 px-1 rounded text-xs">chmod +x batch_ping.sh</code> 赋予执行权限。
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 输入区域 -->
            <div class="bg-white rounded-xl shadow-md p-6 mb-8 animate-fadeIn" style="animation-delay: 0.1s">
                <h2 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa fa-pencil-square-o text-primary mr-2"></i>输入IP地址列表
                </h2>
                <div class="mb-4">
                    <label for="ipList" class="block text-sm font-medium text-gray-700 mb-2">
                        IP地址（每行一个）：
                    </label>
                    <textarea 
                        id="ipList" 
                        rows="8" 
                        class="w-full px-4 py-3 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-primary focus:border-primary transition-all-300"
                        placeholder="例如：
192.168.1.1
192.168.1.2
192.168.1.10-192.168.1.20
10.0.0.1-10.0.0.254"
                    ></textarea>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                    <div>
                        <label for="pingCount" class="block text-sm font-medium text-gray-700 mb-2">
                            PING次数：
                        </label>
                        <input 
                            type="number" 
                            id="pingCount" 
                            min="1" 
                            max="10" 
                            value="4" 
                            class="w-full px-4 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-primary focus:border-primary transition-all-300"
                        >
                    </div>
                    <div>
                        <label for="timeout" class="block text-sm font-medium text-gray-700 mb-2">
                            超时时间(秒)：
                        </label>
                        <input 
                            type="number" 
                            id="timeout" 
                            min="1" 
                            max="10" 
                            value="1" 
                            class="w-full px-4 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-primary focus:border-primary transition-all-300"
                        >
                    </div>
                    <div>
                        <label for="packetSize" class="block text-sm font-medium text-gray-700 mb-2">
                            数据包大小(字节)：
                        </label>
                        <input 
                            type="number" 
                            id="packetSize" 
                            min="32" 
                            max="65500" 
                            value="56" 
                            class="w-full px-4 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-primary focus:border-primary transition-all-300"
                        >
                    </div>
                </div>
                <div class="flex flex-wrap gap-3">
                    <button id="generateBtn" class="bg-gradient-blue text-white px-6 py-3 rounded-lg shadow-md hover:shadow-lg transition-all-300 flex items-center card-hover">
                        <i class="fa fa-magic mr-2"></i>生成PING命令
                    </button>
                    <button id="clearBtn" class="bg-gray-200 text-gray-700 px-6 py-3 rounded-lg shadow-sm hover:shadow-md transition-all-300 flex items-center card-hover">
                        <i class="fa fa-eraser mr-2"></i>清空输入
                    </button>
                    <button id="sampleBtn" class="bg-secondary/20 text-secondary px-6 py-3 rounded-lg shadow-sm hover:shadow-md transition-all-300 flex items-center card-hover">
                        <i class="fa fa-file-text-o mr-2"></i>加载示例
                    </button>
                </div>
            </div>

            <!-- 结果区域 -->
            <div class="bg-white rounded-xl shadow-md p-6 animate-fadeIn" style="animation-delay: 0.2s">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-semibold flex items-center">
                        <i class="fa fa-code text-accent mr-2"></i>PING命令结果 (Linux)
                    </h2>
                    <div class="flex gap-2">
                        <button id="copyBtn" class="bg-gray-100 hover:bg-gray-200 text-gray-700 px-4 py-2 rounded-lg transition-all-300 flex items-center text-sm card-hover" disabled>
                            <i class="fa fa-clipboard mr-1"></i>复制命令
                        </button>
                        <button id="downloadBtn" class="bg-gray-100 hover:bg-gray-200 text-gray-700 px-4 py-2 rounded-lg transition-all-300 flex items-center text-sm card-hover" disabled>
                            <i class="fa fa-download mr-1"></i>下载脚本文件
                        </button>
                    </div>
                </div>
                <div class="relative">
                    <pre id="resultArea" class="bg-gray-50 p-4 rounded-lg border border-gray-200 overflow-x-auto max-h-[300px] text-sm font-mono text-gray-800 whitespace-pre-wrap">
<!-- 结果将在这里显示 -->
请输入IP地址并点击"生成PING命令"按钮</pre>
                    <div id="copySuccess" class="absolute top-3 right-3 bg-success text-white px-3 py-1 rounded-full text-xs opacity-0 transition-opacity duration-300">
                        <i class="fa fa-check mr-1"></i>已复制
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="bg-white shadow-inner mt-12">
        <div class="container mx-auto px-4 py-6">
            <div class="text-center text-gray-500 text-sm">
                <p>内网批量PING工具 (Linux版本) &copy; 2023 | 设计用于Linux内网环境网络检测</p>
            </div>
        </div>
    </footer>

    <!-- JavaScript -->
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // 获取DOM元素
            const ipListInput = document.getElementById('ipList');
            const pingCountInput = document.getElementById('pingCount');
            const timeoutInput = document.getElementById('timeout');
            const packetSizeInput = document.getElementById('packetSize');
            const generateBtn = document.getElementById('generateBtn');
            const clearBtn = document.getElementById('clearBtn');
            const sampleBtn = document.getElementById('sampleBtn');
            const resultArea = document.getElementById('resultArea');
            const copyBtn = document.getElementById('copyBtn');
            const downloadBtn = document.getElementById('downloadBtn');
            const copySuccess = document.getElementById('copySuccess');

            // 解析IP范围，例如将 192.168.1.10-192.168.1.20 解析为多个IP
            function expandIpRange(ipRange) {
                const ipList = [];
                
                // 检查是否为IP范围
                if (ipRange.includes('-')) {
                    const [startIp, endIp] = ipRange.split('-');
                    const startParts = startIp.split('.').map(Number);
                    const endParts = endIp.split('.').map(Number);
                    
                    // 简单验证IP格式
                    if (startParts.length === 4 && endParts.length === 4 && 
                        startParts.every(n => !isNaN(n) && n >= 0 && n <= 255) && 
                        endParts.every(n => !isNaN(n) && n >= 0 && n <= 255)) {
                        
                        // 找出第一个不同的部分的索引
                        let diffIndex = 0;
                        for (; diffIndex < 4; diffIndex++) {
                            if (startParts[diffIndex] !== endParts[diffIndex]) break;
                        }
                        
                        // 生成IP范围
                        if (diffIndex === 3) { // 只有最后一段不同
                            for (let i = startParts[3]; i <= endParts[3]; i++) {
                                ipList.push(`${startParts[0]}.${startParts[1]}.${startParts[2]}.${i}`);
                            }
                        } else {
                            // 格式不正确，作为单个IP处理
                            ipList.push(ipRange);
                        }
                    } else {
                        // 格式不正确，作为单个IP处理
                        ipList.push(ipRange);
                    }
                } else {
                    // 单个IP地址
                    ipList.push(ipRange);
                }
                
                return ipList;
            }

            // 生成PING命令
            generateBtn.addEventListener('click', function() {
                const ipListText = ipListInput.value.trim();
                const pingCount = parseInt(pingCountInput.value) || 4;
                const timeout = parseInt(timeoutInput.value) || 1;
                const packetSize = parseInt(packetSizeInput.value) || 56;
                
                if (!ipListText) {
                    resultArea.textContent = '请输入IP地址列表！';
                    resultArea.className = 'bg-red-50 p-4 rounded-lg border border-red-200 overflow-x-auto max-h-[300px] text-sm font-mono text-red-800';
                    copyBtn.disabled = true;
                    downloadBtn.disabled = true;
                    return;
                }
                
                // 分割IP地址列表
                const lines = ipListText.split('\n').filter(line => line.trim() !== '');
                
                // 展开所有IP范围
                let allIps = [];
                for (const line of lines) {
                    const trimmedLine = line.trim();
                    allIps = allIps.concat(expandIpRange(trimmedLine));
                }
                
                // 生成Linux下的PING命令
                let linuxCommands = '#!/bin/bash\n';
                linuxCommands += 'clear\n';
                linuxCommands += 'echo "开始批量PING测试..."\n';
                linuxCommands += 'echo\n';
                
                for (const ip of allIps) {
                    linuxCommands += 'echo "正在PING ' + ip + '..."\n';
                    linuxCommands += 'ping -c ' + pingCount + ' -W ' + timeout + ' -s ' + packetSize + ' ' + ip + '\n';
                    linuxCommands += 'echo\n';
                }
                
                linuxCommands += 'echo "批量PING测试完成！"';
                
                // 显示结果
                resultArea.textContent = linuxCommands;
                resultArea.className = 'bg-gray-50 p-4 rounded-lg border border-gray-200 overflow-x-auto max-h-[300px] text-sm font-mono text-gray-800';
                
                // 启用复制和下载按钮
                copyBtn.disabled = false;
                downloadBtn.disabled = false;
            });

            // 清空输入
            clearBtn.addEventListener('click', function() {
                ipListInput.value = '';
                pingCountInput.value = 4;
                timeoutInput.value = 1;
                packetSizeInput.value = 56;
                resultArea.textContent = '请输入IP地址并点击"生成PING命令"按钮';
                resultArea.className = 'bg-gray-50 p-4 rounded-lg border border-gray-200 overflow-x-auto max-h-[300px] text-sm font-mono text-gray-800';
                copyBtn.disabled = true;
                downloadBtn.disabled = true;
            });

            // 加载示例
            sampleBtn.addEventListener('click', function() {
                ipListInput.value = `192.168.1.1
192.168.1.2
192.168.1.10-192.168.1.20
10.0.0.1-10.0.0.5`;
            });

            // 复制命令
            copyBtn.addEventListener('click', function() {
                resultArea.select();
                document.execCommand('copy');
                
                // 显示复制成功提示
                copySuccess.style.opacity = '1';
                setTimeout(() => {
                    copySuccess.style.opacity = '0';
                }, 2000);
            });

            // 下载脚本文件
            downloadBtn.addEventListener('click', function() {
                const blob = new Blob([resultArea.textContent], { type: 'text/plain' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = 'batch_ping.sh';
                a.click();
                URL.revokeObjectURL(url);
            });
        });
    </script>
</body>
</html>

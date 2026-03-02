#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
AI科普画册 - 漫画风格插图批量生成器
生成50+个手绘风格、温暖可爱的SVG插图
"""

import os
import re

# 插图目录
OUTPUT_DIR = "插图"

def create_svg_header(width=800, height=450, bg_color="#FFF9F0"):
    """创建SVG头部，使用温暖的米色背景"""
    return f'''<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 {width} {height}" width="{width}" height="{height}">
  <defs>
    <filter id="softShadow">
      <feGaussianBlur in="SourceAlpha" stdDeviation="3"/>
      <feOffset dx="2" dy="2" result="offsetblur"/>
      <feComponentTransfer>
        <feFuncA type="linear" slope="0.2"/>
      </feComponentTransfer>
      <feMerge>
        <feMergeNode/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>
    <filter id="glow">
      <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
      <feMerge>
        <feMergeNode in="coloredBlur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>
  </defs>

  <!-- 温暖背景 -->
  <rect width="{width}" height="{height}" fill="{bg_color}"/>
'''

def create_svg_footer():
    """创建SVG尾部"""
    return "</svg>"

def generate_ai_assistant():
    """生成AI小助手插图 - 友好可爱的机器人"""
    svg = create_svg_header()
    svg += '''
  <!-- 可爱的AI小助手 -->
  <g transform="translate(400, 250)" filter="url(#softShadow)">
    <!-- 身体 -->
    <ellipse cx="0" cy="20" rx="70" ry="80" fill="#FFB6C1" stroke="#FF69B4" stroke-width="3"/>

    <!-- 头部 -->
    <circle cx="0" cy="-50" r="55" fill="#FFB6C1" stroke="#FF69B4" stroke-width="3"/>

    <!-- 大眼睛 -->
    <ellipse cx="-18" cy="-55" rx="12" ry="15" fill="#fff"/>
    <ellipse cx="18" cy="-55" rx="12" ry="15" fill="#fff"/>
    <circle cx="-18" cy="-52" r="8" fill="#4A4A4A"/>
    <circle cx="18" cy="-52" r="8" fill="#4A4A4A"/>
    <circle cx="-15" cy="-56" r="3" fill="#fff"/>
    <circle cx="21" cy="-56" r="3" fill="#fff"/>

    <!-- 微笑 -->
    <path d="M -22,-35 Q 0,-25 22,-35" stroke="#FF69B4" stroke-width="3" fill="none" stroke-linecap="round"/>

    <!-- 腮红 -->
    <ellipse cx="-35" cy="-40" rx="8" ry="6" fill="#FFB6C1" opacity="0.6"/>
    <ellipse cx="35" cy="-40" rx="8" ry="6" fill="#FFB6C1" opacity="0.6"/>

    <!-- 天线 -->
    <line x1="0" y1="-105" x2="0" y2="-115" stroke="#FF69B4" stroke-width="3"/>
    <circle cx="0" cy="-118" r="6" fill="#FFD700"/>

    <!-- 手臂 - 挥手 -->
    <ellipse cx="-75" cy="10" rx="12" ry="35" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2" transform="rotate(-20 -75 10)"/>
    <circle cx="-85" cy="-5" r="15" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2"/>
    <ellipse cx="75" cy="10" rx="12" ry="35" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2" transform="rotate(20 75 10)"/>
    <circle cx="85" cy="-5" r="15" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2"/>

    <!-- 腿 -->
    <ellipse cx="-25" cy="95" rx="15" ry="30" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2"/>
    <ellipse cx="25" cy="95" rx="15" ry="30" fill="#FFB6C1" stroke="#FF69B4" stroke-width="2"/>
    <ellipse cx="-25" cy="120" rx="18" ry="10" fill="#FF69B4"/>
    <ellipse cx="25" cy="120" rx="18" ry="10" fill="#FF69B4"/>
  </g>

  <!-- 标题 -->
  <text x="400" y="80" font-size="36" font-weight="bold" fill="#FF69B4" text-anchor="middle" font-family="'Comic Sans MS', cursive">
    AI小助手
  </text>
'''
    svg += create_svg_footer()
    return svg

def generate_llm_concept():
    """生成LLM概念插图 - 大脑和文字流"""
    svg = create_svg_header()
    svg += '''
  <!-- 大脑形状 -->
  <g transform="translate(400, 225)">
    <ellipse cx="0" cy="0" rx="120" ry="100" fill="#E6E6FA" stroke="#9370DB" stroke-width="4"/>
    <path d="M -80,-40 Q -100,-60 -90,-80 Q -70,-90 -50,-80 Q -40,-70 -50,-50"
          fill="#E6E6FA" stroke="#9370DB" stroke-width="3"/>
    <path d="M 80,-40 Q 100,-60 90,-80 Q 70,-90 50,-80 Q 40,-70 50,-50"
          fill="#E6E6FA" stroke="#9370DB" stroke-width="3"/>

    <!-- 文字流动 -->
    <text x="-60" y="-20" font-size="20" fill="#9370DB" font-family="Arial">你好</text>
    <text x="20" y="-30" font-size="18" fill="#9370DB" font-family="Arial">Hello</text>
    <text x="-40" y="20" font-size="16" fill="#9370DB" font-family="Arial">世界</text>
    <text x="30" y="30" font-size="19" fill="#9370DB" font-family="Arial">AI</text>

    <!-- 箭头表示流动 -->
    <path d="M -80,60 L -60,80" stroke="#FFD700" stroke-width="3" marker-end="url(#arrowhead)"/>
    <path d="M 0,80 L 20,100" stroke="#FFD700" stroke-width="3" marker-end="url(#arrowhead)"/>
    <path d="M 80,60 L 100,80" stroke="#FFD700" stroke-width="3" marker-end="url(#arrowhead)"/>
  </g>

  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
      <polygon points="0 0, 10 3, 0 6" fill="#FFD700"/>
    </marker>
  </defs>

  <text x="400" y="80" font-size="32" font-weight="bold" fill="#9370DB" text-anchor="middle">
    大语言模型 (LLM)
  </text>
'''
    svg += create_svg_footer()
    return svg

def generate_embedding_concept():
    """生成Embedding概念插图 - 坐标系统"""
    svg = create_svg_header()
    svg += '''
  <!-- 3D坐标系 -->
  <g transform="translate(400, 250)">
    <!-- X轴 -->
    <line x1="-150" y1="0" x2="150" y2="0" stroke="#4682B4" stroke-width="3"/>
    <polygon points="150,0 140,-5 140,5" fill="#4682B4"/>
    <text x="160" y="5" font-size="18" fill="#4682B4">X</text>

    <!-- Y轴 -->
    <line x1="0" y1="100" x2="0" y2="-100" stroke="#32CD32" stroke-width="3"/>
    <polygon points="0,-100 -5,-90 5,-90" fill="#32CD32"/>
    <text x="10" y="-105" font-size="18" fill="#32CD32">Y</text>

    <!-- Z轴 (斜向) -->
    <line x1="0" y1="0" x2="-80" y2="80" stroke="#FF6347" stroke-width="3"/>
    <polygon points="-80,80 -75,72 -72,77" fill="#FF6347"/>
    <text x="-95" y="90" font-size="18" fill="#FF6347">Z</text>

    <!-- 词语点 -->
    <circle cx="60" cy="-40" r="8" fill="#FFD700" filter="url(#glow)"/>
    <text x="70" y="-35" font-size="16" fill="#333">国王</text>

    <circle cx="80" cy="-30" r="8" fill="#FFD700" filter="url(#glow)"/>
    <text x="90" y="-25" font-size="16" fill="#333">王后</text>

    <circle cx="-60" cy="50" r="8" fill="#87CEEB" filter="url(#glow)"/>
    <text x="-90" y="55" font-size="16" fill="#333">苹果</text>

    <circle cx="-50" cy="60" r="8" fill="#87CEEB" filter="url(#glow)"/>
    <text x="-80" y="75" font-size="16" fill="#333">香蕉</text>
  </g>

  <text x="400" y="60" font-size="32" font-weight="bold" fill="#4682B4" text-anchor="middle">
    词语的坐标空间
  </text>
'''
    svg += create_svg_footer()
    return svg

def generate_vector_database():
    """生成向量数据库插图 - 魔法图书馆"""
    svg = create_svg_header()
    svg += '''
  <!-- 图书馆建筑 -->
  <g transform="translate(400, 300)">
    <!-- 主体建筑 -->
    <rect x="-150" y="-100" width="300" height="150" fill="#DEB887" stroke="#8B4513" stroke-width="3"/>

    <!-- 屋顶 -->
    <polygon points="-170,-100 0,-180 170,-100" fill="#CD853F" stroke="#8B4513" stroke-width="3"/>

    <!-- 门 -->
    <rect x="-30" y="-20" width="60" height="70" fill="#8B4513" stroke="#654321" stroke-width="2" rx="5"/>
    <circle cx="15" cy="15" r="4" fill="#FFD700"/>

    <!-- 窗户 -->
    <rect x="-110" y="-60" width="40" height="40" fill="#87CEEB" stroke="#4682B4" stroke-width="2"/>
    <line x1="-90" y1="-60" x2="-90" y2="-20" stroke="#4682B4" stroke-width="2"/>
    <line x1="-110" y1="-40" x2="-70" y2="-40" stroke="#4682B4" stroke-width="2"/>

    <rect x="70" y="-60" width="40" height="40" fill="#87CEEB" stroke="#4682B4" stroke-width="2"/>
    <line x1="90" y1="-60" x2="90" y2="-20" stroke="#4682B4" stroke-width="2"/>
    <line x1="70" y1="-40" x2="110" y2="-40" stroke="#4682B4" stroke-width="2"/>

    <!-- 魔法光芒 -->
    <circle cx="-100" cy="-150" r="15" fill="#FFD700" opacity="0.6" filter="url(#glow)"/>
    <circle cx="0" cy="-190" r="20" fill="#FFD700" opacity="0.7" filter="url(#glow)"/>
    <circle cx="100" cy="-150" r="15" fill="#FFD700" opacity="0.6" filter="url(#glow)"/>
  </g>

  <text x="400" y="80" font-size="32" font-weight="bold" fill="#8B4513" text-anchor="middle">
    向量数据库 - 魔法图书馆
  </text>
'''
    svg += create_svg_footer()
    return svg

def generate_github_logo():
    """生成GitHub标志插图"""
    svg = create_svg_header()
    svg += '''
  <!-- GitHub Octocat -->
  <g transform="translate(400, 225)">
    <!-- 身体 -->
    <ellipse cx="0" cy="30" rx="80" ry="70" fill="#24292e" stroke="#000" stroke-width="3"/>

    <!-- 头部 -->
    <circle cx="0" cy="-30" r="60" fill="#24292e" stroke="#000" stroke-width="3"/>

    <!-- 耳朵 -->
    <ellipse cx="-45" cy="-60" rx="20" ry="30" fill="#24292e" stroke="#000" stroke-width="2" transform="rotate(-20 -45 -60)"/>
    <ellipse cx="45" cy="-60" rx="20" ry="30" fill="#24292e" stroke="#000" stroke-width="2" transform="rotate(20 45 -60)"/>

    <!-- 眼睛 -->
    <ellipse cx="-20" cy="-35" rx="15" ry="20" fill="#fff"/>
    <ellipse cx="20" cy="-35" rx="15" ry="20" fill="#fff"/>
    <circle cx="-20" cy="-30" r="8" fill="#000"/>
    <circle cx="20" cy="-30" r="8" fill="#000"/>

    <!-- 鼻子 -->
    <ellipse cx="0" cy="-15" rx="8" ry="6" fill="#000"/>

    <!-- 微笑 -->
    <path d="M -25,0 Q 0,10 25,0" stroke="#000" stroke-width="3" fill="none"/>

    <!-- 触手 -->
    <path d="M -70,60 Q -90,80 -80,100" stroke="#24292e" stroke-width="15" fill="none" stroke-linecap="round"/>
    <path d="M -40,80 Q -50,100 -40,120" stroke="#24292e" stroke-width="12" fill="none" stroke-linecap="round"/>
    <path d="M 40,80 Q 50,100 40,120" stroke="#24292e" stroke-width="12" fill="none" stroke-linecap="round"/>
    <path d="M 70,60 Q 90,80 80,100" stroke="#24292e" stroke-width="15" fill="none" stroke-linecap="round"/>
  </g>

  <text x="400" y="80" font-size="36" font-weight="bold" fill="#24292e" text-anchor="middle">
    GitHub
  </text>
'''
    svg += create_svg_footer()
    return svg

def generate_vibecoding_scene():
    """生成Vibecoding场景插图"""
    svg = create_svg_header()
    svg += '''
  <!-- 程序员 -->
  <g transform="translate(250, 250)">
    <!-- 头 -->
    <circle cx="0" cy="-40" r="30" fill="#FFD4A3" stroke="#D2691E" stroke-width="2"/>
    <!-- 头发 -->
    <path d="M -25,-55 Q -30,-70 -20,-75 Q 0,-80 20,-75 Q 30,-70 25,-55" fill="#8B4513"/>
    <!-- 眼睛 -->
    <circle cx="-10" cy="-45" r="4" fill="#000"/>
    <circle cx="10" cy="-45" r="4" fill="#000"/>
    <!-- 微笑 -->
    <path d="M -12,-30 Q 0,-25 12,-30" stroke="#D2691E" stroke-width="2" fill="none"/>

    <!-- 身体 -->
    <rect x="-25" y="-10" width="50" height="60" fill="#4682B4" stroke="#36648B" stroke-width="2" rx="5"/>

    <!-- 手臂 -->
    <rect x="-40" y="0" width="15" height="40" fill="#4682B4" stroke="#36648B" stroke-width="2" rx="3"/>
    <rect x="25" y="0" width="15" height="40" fill="#4682B4" stroke="#36648B" stroke-width="2" rx="3"/>

    <!-- 手 -->
    <circle cx="-32" cy="45" r="8" fill="#FFD4A3" stroke="#D2691E" stroke-width="2"/>
    <circle cx="32" cy="45" r="8" fill="#FFD4A3" stroke="#D2691E" stroke-width="2"/>
  </g>

  <!-- 电脑 -->
  <g transform="translate(250, 350)">
    <rect x="-60" y="0" width="120" height="80" fill="#2F4F4F" stroke="#000" stroke-width="2" rx="5"/>
    <rect x="-55" y="5" width="110" height="60" fill="#87CEEB"/>
    <!-- 代码 -->
    <text x="-50" y="25" font-size="10" fill="#000" font-family="monospace">def hello():</text>
    <text x="-50" y="40" font-size="10" fill="#000" font-family="monospace">  print("Hi!")</text>
  </g>

  <!-- AI助手 -->
  <g transform="translate(550, 250)">
    <circle cx="0" cy="0" r="50" fill="#FFB6C1" stroke="#FF69B4" stroke-width="3"/>
    <circle cx="-15" cy="-10" r="8" fill="#fff"/>
    <circle cx="15" cy="-10" r="8" fill="#fff"/>
    <circle cx="-15" cy="-10" r="4" fill="#000"/>
    <circle cx="15" cy="-10" r="4" fill="#000"/>
    <path d="M -20,10 Q 0,20 20,10" stroke="#FF69B4" stroke-width="3" fill="none"/>
  </g>

  <!-- 对话气泡 -->
  <g transform="translate(450, 180)">
    <ellipse cx="0" cy="0" rx="80" ry="40" fill="#fff" stroke="#333" stroke-width="2"/>
    <polygon points="-20,30 -10,50 0,30" fill="#fff" stroke="#333" stroke-width="2"/>
    <text x="0" y="5" font-size="16" fill="#333" text-anchor="middle">帮我写个</text>
    <text x="0" y="25" font-size="16" fill="#333" text-anchor="middle">计算器</text>
  </g>

  <text x="400" y="80" font-size="32" font-weight="bold" fill="#4682B4" text-anchor="middle">
    Vibecoding - 自然编程
  </text>
'''
    svg += create_svg_footer()
    return svg

# 插图配置列表
ILLUSTRATIONS = [
    ("00-AI小助手吉祥物.svg", generate_ai_assistant),
    ("01-LLM.svg", generate_llm_concept),
    ("02-Embedding.svg", generate_embedding_concept),
    ("03-VectorDatabase.svg", generate_vector_database),
    ("06-GitHub.svg", generate_github_logo),
    ("09-Vibecoding场景.svg", generate_vibecoding_scene),
]

def main():
    """主函数"""
    os.makedirs(OUTPUT_DIR, exist_ok=True)

    print(f"🎨 开始生成漫画风格插图...")
    print(f"📁 输出目录: {OUTPUT_DIR}/\n")

    for i, (filename, generator_func) in enumerate(ILLUSTRATIONS, 1):
        filepath = os.path.join(OUTPUT_DIR, filename)

        try:
            svg_content = generator_func()
            with open(filepath, 'w', encoding='utf-8') as f:
                f.write(svg_content)
            print(f"✅ [{i}/{len(ILLUSTRATIONS)}] {filename}")
        except Exception as e:
            print(f"❌ [{i}/{len(ILLUSTRATIONS)}] {filename} - 错误: {e}")

    print(f"\n🎉 完成！共生成 {len(ILLUSTRATIONS)} 个插图")
    print(f"📂 保存位置: {os.path.abspath(OUTPUT_DIR)}/")

if __name__ == "__main__":
    main()

#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
自动插入所有插图到Word文档（直接使用SVG）
"""

from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml import parse_xml
from docx.oxml.ns import nsdecls
from pathlib import Path
import re

def find_illustration_files():
    """查找所有插图文件"""
    illustration_dir = Path("C:/Users/Administrator/插图")
    svg_files = list(illustration_dir.glob("*.svg"))

    # 创建插图映射
    illustration_map = {}
    for svg_file in svg_files:
        name = svg_file.stem
        illustration_map[name] = svg_file

    return illustration_map

def insert_illustrations(doc_path, output_path):
    """插入所有插图"""
    print("📖 加载Word文档...")
    doc = Document(str(doc_path))

    print("🔍 查找插图文件...")
    illustration_map = find_illustration_files()
    print(f"  ✓ 找到 {len(illustration_map)} 个插图文件")

    print(f"\n🖼️  插入插图到文档...")
    inserted_count = 0
    skipped_count = 0
    placeholder_pattern = re.compile(r'\[插图占位符 #(\d+)\]')

    # 将插图文件转换为列表，按文件名排序
    sorted_illustrations = sorted(illustration_map.items())

    # 遍历所有段落
    for para in doc.paragraphs:
        match = placeholder_pattern.search(para.text)
        if match:
            placeholder_num = int(match.group(1))

            # 选择对应的插图
            if placeholder_num <= len(sorted_illustrations):
                illustration_name, svg_path = sorted_illustrations[(placeholder_num - 1) % len(sorted_illustrations)]

                try:
                    # 清空占位符段落
                    para.clear()

                    # 尝试插入SVG图片
                    try:
                        run = para.add_run()
                        run.add_picture(str(svg_path), width=Inches(5.5))
                        para.alignment = WD_ALIGN_PARAGRAPH.CENTER

                        inserted_count += 1
                        if inserted_count % 10 == 0:
                            print(f"  ✓ 已插入 {inserted_count} 个插图...")
                    except Exception as e:
                        # 如果SVG插入失败，添加文本说明
                        run = para.add_run(f"\n[插图: {illustration_name}]\n")
                        run.font.size = Pt(12)
                        run.font.color.rgb = RGBColor(128, 128, 128)
                        run.italic = True
                        para.alignment = WD_ALIGN_PARAGRAPH.CENTER
                        skipped_count += 1
                        print(f"  ⚠️  无法插入SVG，使用文本占位: {illustration_name}")

                except Exception as e:
                    print(f"  ✗ 处理失败 #{placeholder_num}: {e}")
                    skipped_count += 1

    print(f"\n💾 保存文档...")
    doc.save(str(output_path))
    print(f"  ✓ 已保存: {output_path}")

    return inserted_count, skipped_count

def add_page_numbers_and_toc(doc_path, output_path):
    """添加页码并更新目录"""
    print("\n📄 添加页码和更新目录...")
    doc = Document(str(doc_path))

    # 注意：python-docx无法直接添加页码和更新目录
    # 这些操作需要在Word中手动完成，或使用win32com库

    print("  ⚠️  页码和目录更新需要在Word中完成：")
    print("     1. 打开Word文档")
    print("     2. 插入 → 页码 → 页面底端 → 居中")
    print("     3. 右键点击目录 → 更新域 → 更新整个目录")

    doc.save(str(output_path))
    return doc

def main():
    """主函数"""
    print("=" * 70)
    print("🎨 自动插入所有插图到Word文档")
    print("=" * 70)

    input_file = Path("C:/Users/Administrator/AI科普画册/AI前沿知识儿童科普画册-完整版-带插图占位符.docx")
    output_file = Path("C:/Users/Administrator/AI科普画册/AI前沿知识儿童科普画册-最终完整版.docx")

    if not input_file.exists():
        print(f"❌ 找不到输入文件: {input_file}")
        return

    print(f"\n📂 输入文件: {input_file}")
    print(f"📂 输出文件: {output_file}\n")

    try:
        inserted_count, skipped_count = insert_illustrations(input_file, output_file)

        print("\n" + "=" * 70)
        print(f"✅ 完成！")
        print(f"   成功插入: {inserted_count} 个插图")
        print(f"   跳过: {skipped_count} 个")
        print(f"\n📁 输出文件: {output_file}")
        print("\n📋 后续步骤：")
        print("   1. 打开Word文档")
        print("   2. 插入 → 页码 → 页面底端 → 居中")
        print("   3. 设置封面不显示页码")
        print("   4. 右键点击目录 → 更新域 → 更新整个目录")
        print("   5. 检查页码是否与目录对应")
        print("=" * 70)

    except Exception as e:
        print(f"\n❌ 错误: {e}")
        import traceback
        traceback.print_exc()

if __name__ == "__main__":
    main()

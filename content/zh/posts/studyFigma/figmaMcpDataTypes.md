---
title: "Figma MCP 工具数据类型详解"
date: 2024-12-21T10:00:00+08:00
draft: ture
tags: ["figma", "mcp", "design-to-code"]
categories: ["other"]
---
（Created by AI）
# Figma MCP 工具数据类型详解

本文将详细介绍 Figma MCP (Model Context Protocol) 工具可以获取的各种数据类型，以及如何建立连接和获取设计稿信息。

## 工具概览

Figma MCP 提供以下核心工具：

1. **get_metadata** - 获取节点元数据信息
2. **get_code** - 生成 UI 代码实现
3. **get_variable_defs** - 获取设计变量定义
4. **get_code_connect_map** - 获取代码连接映射（企业版功能）
5. **get_image** - 生成节点图像
6. **create_design_system_rules** - 创建设计系统规则

## 示例分析

使用 Figma 设计稿：`https://www.figma.com/design/k0mSHNH3VmapS9ZamxFkVl/PLAUD-AI-SDK?node-id=12849-12248&m=dev`

从 URL 中提取的 node-id: `12849-12248`

## 1. 元数据信息 (get_metadata)

元数据提供了节点的基本结构信息，包括：

- 节点类型（frame、instance、symbol 等）
- 节点名称
- 位置坐标 (x, y)
- 尺寸信息 (width, height)
- 子节点层级结构

### 示例输出：
```xml
<frame id="12849:12248" name="Frame 1739333131" x="0" y="60" width="952" height="676">
  <instance id="12849:12249" name="Filter Bar" x="0" y="0" width="952" height="32" />
  <symbol id="13088:67609" name="Frame 1739333044" x="0" y="52" width="952" height="572" />
  <frame id="12849:12627" name="per page" x="0" y="644" width="244" height="32">
    <instance id="12849:12628" name="Pagination" x="0" y="1" width="30" height="30" />
    <instance id="12849:12629" name="Pagination" x="34" y="0" width="32" height="32" />
    <!-- 更多子节点... -->
  </frame>
</frame>
```

**用途：** 快速了解设计稿的整体结构，适合在实现前进行架构规划。

## 2. 代码实现 (get_code)

这是最核心的功能，将 Figma 设计直接转换为可运行的代码：

### 主要特性：
- **React + TypeScript** 组件代码
- **Tailwind CSS** 样式系统
- **完整的 Props 接口定义**
- **图片资源处理**
- **响应式设计支持**

### 代码结构示例：
```typescript
interface FilterOptionProps {
  state?: "Default" | "Selected" | "Active";
}

function FilterOption({ state = "Default" }: FilterOptionProps) {
  return (
    <div className="bg-white relative rounded-[5px] size-full" 
         data-name="State=Default" 
         data-node-id="12555:10420">
      <div className="box-border content-stretch flex items-center justify-between min-w-inherit overflow-clip px-2 py-1 relative size-full">
        <div className="font-['Inter:Regular',_sans-serif] font-normal leading-[0] not-italic relative shrink-0 text-[#3d3d3d] text-[12px] text-nowrap" 
             data-node-id="12555:10397">
          <p className="leading-[18px] whitespace-pre">Last 24 hours</p>
        </div>
        {/* 更多组件内容... */}
      </div>
    </div>
  );
}
```

### 重要特性：
- **data-node-id 属性**：每个元素都包含对应的 Figma 节点 ID
- **精确的样式还原**：包括字体、颜色、间距、圆角等
- **组件化设计**：自动识别重复元素并生成可复用组件
- **图片资源管理**：自动处理 SVG 和位图资源

## 3. 设计变量定义 (get_variable_defs)

获取设计系统中定义的所有变量，包括：

### 变量类型：
- **颜色变量**：主色、辅助色、状态色等
- **字体变量**：字体族、字重、字号、行高
- **间距变量**：padding、margin、gap 等
- **圆角变量**：border-radius 值
- **其他样式变量**

### 示例输出：
```json
{
  "Labels/Primary": "#000000",
  "Labels/Secondary": "#3d3d3d",
  "Labels/Tertiary": "#858c9b",
  "Labels/Quaternary": "#a3a3a3",
  "Labels/White": "#ffffff",
  "Font Family/Font": "Inter",
  "Font Size/Body": "14",
  "Font Size/Footnote": "14",
  "Line Height/Body": "22",
  "Line Height/Footnote": "20",
  "Spacing_4": "4",
  "Spacing_8": "8",
  "Spacing_20": "20",
  "Spacing_24": "24",
  "Radius_5": "5",
  "Status/Destructive": "#ff503f",
  "Template Community/Icon/Moss": "#4cbf69",
  "Template Community/Icon/Ember": "#cc7447"
}
```

**用途：** 建立统一的设计系统，确保代码实现与设计保持一致。

## 4. 代码连接映射 (get_code_connect_map)

> **注意：** 此功能仅在 Figma Organization 和 Enterprise 计划中可用

用于建立 Figma 组件与代码库中实际组件的映射关系：

### 功能：
- 将 Figma 组件链接到代码库中的具体文件
- 提供组件名称映射
- 支持版本管理和同步

### 预期输出格式：
```json
{
  "1:2": {
    "codeConnectSrc": "https://github.com/foo/components/Button.tsx",
    "codeConnectName": "Button"
  }
}
```

## 5. 图像生成 (get_image)

为指定节点生成图像文件，用于：
- 设计稿预览
- 文档展示
- 设计评审
- 开发参考

## 6. 设计系统规则 (create_design_system_rules)

生成适用于当前项目的设计系统规则和约束。

## 使用流程

### 1. 建立连接
- 确保 Figma 桌面应用已打开
- 选择要处理的节点或提供 node-id

### 2. 数据获取顺序建议
1. **get_metadata** - 了解整体结构
2. **get_variable_defs** - 获取设计变量
3. **get_code** - 生成具体实现
4. **get_image** - 获取视觉参考（可选）

### 3. 从 URL 提取 node-id
URL 格式：`https://figma.com/design/:fileKey/:fileName?node-id=1-2`
提取的 node-id：`1:2`（将 `-` 替换为 `:`）

## 实际应用场景

### 设计到代码的完整流程：
1. **设计师** 在 Figma 中完成设计
2. **开发者** 使用 MCP 工具获取元数据了解结构
3. **自动生成** 基础组件代码
4. **开发者** 基于生成的代码进行业务逻辑开发
5. **设计变量** 确保样式一致性

### 优势：
- **高效率**：大幅减少手动编写 UI 代码的时间
- **高精度**：像素级别的设计还原
- **可维护**：生成的代码结构清晰，易于维护
- **一致性**：通过设计变量确保整个项目的视觉一致性

## 注意事项

1. **图片资源**：生成的代码中图片资源指向 localhost 服务器，实际使用时需要替换为项目中的资源路径
2. **样式系统**：工具会自动检测项目的样式方案（Tailwind CSS、CSS-in-JS 等）
3. **组件复用**：工具会自动识别重复设计模式并生成可复用组件
4. **TypeScript 支持**：生成的代码包含完整的类型定义

## 总结

Figma MCP 工具提供了从设计到代码的完整解决方案，通过多种数据类型的获取，可以实现高效、精确的设计实现流程。合理使用这些工具，可以显著提升前端开发效率，并确保设计与实现的一致性。

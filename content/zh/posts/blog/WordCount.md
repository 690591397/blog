---
title: 五分钟搭建个人博客：字数统计，可配置的front-matter
date: 2024-04-29T06:47:40.000Z
draft: false
description: ' 在探索Hugo博客系统的优化过程中，作者通过node脚本实现了front-matter配置的自动化，解决了中文环境下字数统计不准确的问题。文章详细记录了从发现问题到解决问题的全过程，包括对CJK语言特性的理解、Hugo配置的调整、以及最终通过node脚本批量修改文章配置的实现。作者分享了这一过程中遇到的技术挑战和解决方案，旨在帮助其他从其他博客工具迁移到Hugo的用户，实现更平滑的过渡。文章最后，作者提出了对现有解决方案的反思，以及对未来可能改进方向的思考。'
---

这次是使用node脚本实现的front-matter可配置。

上次在[系列文章](/series/五分钟搭建个人博客/)“更多设置”中简单介绍过字数统计功能。在英文环境下稍微验证了下效果后就没过多留意了，但是，在今天切到中文语言下，发现只有短短的100来个字，啊？！

![ 2024-04-29 22.45.58.png](https://s2.loli.net/2024/04/29/nR1P2IgWKtZQHFs.png)

### hasCJKLanguage
简单研究了下，大概是在文本渲染、分词、搜索或者字体处理等方面，CJK语言因为其独特的字符集和排版规则，通常需要特别对待。CJK是中文（Chinese）、日文（Japanese）、韩文（Korean）的缩写。

所以Hugo提供了[hasCJKLanguage](https://gohugo.io/getting-started/configuration/#hascjklanguage)配置。同样的，也需要再front-matter上设置[isCJKLanguage](https://gohugo.io/content-management/front-matter/#iscjklanguage)。
```yaml
# config yaml file
params:
  ShowWordCount: true
  ShowReadingTime: true
  hasCJKLanguage: true
```
```md
<!-- article front-matter  -->
+++
ShowReadingTime = true
ShowWordCount = true
isCJKLanguage = true
+++
```

在将参数复制了几个文件之后... 我发现还有好多草稿文件🤦🏻‍♀️。
就又简单研究了下。

在全局配置`isCJKLanguage: true`并不可以，后续有机会再研究源码看看这些参数都是什么个实现方式。

### archetypes
发现有推荐这个东西的，看了这个不是在创建时候的模板么。在使用指令创建文件的时候，会根据该目录路径对应起来，使用模板。比如在`content/zh/`下创建新文件，会使用`archetypes/zh/`下的模板。

对于现有存量文章来说，并不适用。

况且模板文件存在最大一个问题就是，模板修改，通过模板创建的文件并不会修改。

### data
接着又找到了data目录下的数据使用，但是只能在模板文件中使用，模板参数中还不知道有什么参数可以获取，怎么设置...

### node代码修改文章isCJKLanguage配置
半天也没有找到有暴露构建前的生命周期钩子，或者在某个步骤能够修改，构建完成之后倒是有个[PostProcess](https://gohugo.io/hugo-pipes/postprocess/)。

一气之下... 掏出老工具node（主要是Vercel设置了node，可以直接使用）。直接修改文章内容，给头部添加上isCJKLanguage属性/doge。

这一下午迭代的心路历程就不说了，简述下代码做了什么事情吧。

1. 开始遍历 front matter 配置目录。
1. 获取所有文件(yaml|toml)和目录，优先处理所有配置文件，同级同名的文件和目录，文件优先，比如en.yaml和/en/。
1. 获取配置信息。
1. 获取与配置文件对应的content路径，遍历路径所有文件。
1. 合并文件原配置和另外设置的配置。同key后覆盖前。
1. 回写文件。

源码如下：
```javascript
const fs = require('fs');
const path = require('path');
const matter = require('gray-matter');
const toml = require('@iarna/toml');
const yaml = require('js-yaml');

// 定义内容和 front matter 配置的目录
const contentDir = path.join(__dirname, './content');
const frontMatterDir = path.join(__dirname, './front-matter');
console.info('contentDir, frontMatterDir', contentDir, frontMatterDir);

// 读取 front matter 配置文件
function readFrontMatterConfig(filePath) {
  const ext = path.extname(filePath);
  const content = fs.readFileSync(filePath, 'utf8');
  if (ext === '.toml') {
    return toml.parse(content);
  } else if (ext === '.yaml' || ext === '.yml') {
    return yaml.load(content);
  } else {
    throw new Error(`Unsupported front matter config format: ${ext}`);
  }
}

// 更新指定目录下的 Markdown 文件的 front matter
function updateDirectoryFrontMatter(dirPath, frontMatterConfig) {
  const updateFile = () => {
    if (fs.existsSync(`${dirPath}.md`)) {
      updateFrontMatter(`${dirPath}.md`, frontMatterConfig); // 更新文件
    }
  };
  try {
    if (fs.lstatSync(dirPath).isDirectory()) {
      fs.readdirSync(dirPath).forEach(file => {
        const fullPath = path.join(dirPath, file);
        if (fs.lstatSync(fullPath).isDirectory()) {
          updateDirectoryFrontMatter(fullPath, frontMatterConfig); // 递归子目录
        } else if (path.extname(fullPath) === '.md') {
          updateFrontMatter(fullPath, frontMatterConfig); // 更新文件
        }
      });
    } else {
      console.info('这里到得了？？？');
      updateFile();
    }
  } catch (error) {
    updateFile();
  }
}

function replacePlusPlusPlus(str) {
  let count = 0;
  return str.replace(/^\+\+\+\n/gm, function(match) {
      count++;
      if (count === 1) {
          return '---toml\n';
      } else if (count === 2) {
          return '---\n';
      } else {
          return match;
      }
  });
}

const parseFrontMatter = (content) => {
  if (content.startsWith('+++')) {
    return matter(
      replacePlusPlusPlus(content),
      {
        engines: {
          toml: toml.parse.bind(toml)
        }
      }
    );
  }
  return matter(content);
};

// 读取并更新 Markdown 文件的 front matter
function updateFrontMatter(filePath, frontMatterConfig) {
  const fileContent = fs.readFileSync(filePath, 'utf8');
  const parsedContent = parseFrontMatter(fileContent);
  console.info('parsedContent.data', parsedContent.data);

  // 使用配置文件更新 front matter 数据
  Object.assign(parsedContent.data, frontMatterConfig);

  const newContent = matter.stringify(parsedContent.content, parsedContent.data);
  fs.writeFileSync(filePath, newContent, 'utf8');
  
  console.log(`Updated: ${filePath}`, newContent);
}

// 遍历 front matter 配置目录
function traverseFrontMatterConfigDir(dir) {
  // 获取所有文件和目录
  const items = fs.readdirSync(dir);
  // 先处理所有文件
  console.info(`处理 ${dir} 下的所有文件:`, items.filter(item => !fs.lstatSync(path.join(dir, item)).isDirectory()));
  items.filter(item => !fs.lstatSync(path.join(dir, item)).isDirectory()).forEach(file => {
    const fullPath = path.join(dir, file);
    const relativePath = fullPath.substring(frontMatterDir.length + 1);
    const contentPath = path.join(contentDir, relativePath);

    // 获取配置文件的内容
    const frontMatterConfig = readFrontMatterConfig(fullPath); // 读取失败
    // 构建对应的内容目录路径
    const dirToUpdate = contentPath.replace(path.extname(contentPath), '');

    console.table({
      fullPath, relativePath, contentPath,
      frontMatterConfig: JSON.stringify(frontMatterConfig),
      dirToUpdate,
    })

    if (dirToUpdate) {
      updateDirectoryFrontMatter(dirToUpdate, frontMatterConfig); // 更新对应目录
    }
  });

  // 然后递归处理所有目录
  items.filter(item => fs.lstatSync(path.join(dir, item)).isDirectory()).forEach(subDir => {
    traverseFrontMatterConfigDir(path.join(dir, subDir)); // 递归子目录
  });
}

// 开始遍历 front matter 配置目录
traverseFrontMatterConfigDir(frontMatterDir);

```


可以在Vercel 添加该文件的执行 `node update-front-matter.js && git submodule update --init --recursive && hugo --gc`，倒是也有npm相关依赖，不用install。优势可以作为忘记添加一些通用配置的保底行为，避免配置缺失。不足之处也很明显，开发环境如果不执行脚本，则与最终部署环境内容不一致，处理方式略显暴力。

后续接触得更多，对于这个场景会找到更合适的解法。这个方式分享出来，也是希望从别的博客工具迁移到Hugo的，能够更平滑、更简单。

也让我不禁思考，现有能简单达到效果的方案，有必要花更多更长时间，去研究合理一些的办法吗？不过，我想我应该会去。

今天这五分钟，希望对您有帮助/doge。



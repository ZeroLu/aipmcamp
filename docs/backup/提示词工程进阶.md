<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>AI PM Camp 文档</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="AI PM Camp 文档网站">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <!-- Docsify CSS -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <!-- Docsify 内容 -->
  <div id="app"></div>
  
  <!-- Docsify 配置 -->
  <script>
    // Docsify 配置
    window.$docsify = {
      name: 'AI PM Camp',
      repo: '',
      loadSidebar: true,
      subMaxLevel: 3,
      auto2top: true,
      search: {
        maxAge: 86400000,
        paths: 'auto',
        placeholder: '搜索',
        noData: '没有找到结果',
        depth: 6
      },
      pagination: {
        previousText: '上一章',
        nextText: '下一章',
      },
      copyCode: {
        buttonText: '复制',
        errorText: '错误',
        successText: '已复制'
      }
    };
  </script>
  
  <!-- Docsify 脚本 -->
  <script src="https://cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="https://cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/docsify/lib/plugins/zoom-image.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/docsify-copy-code"></script>
  <script src="https://cdn.jsdelivr.net/npm/docsify-pagination/dist/docsify-pagination.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/prismjs/components/prism-bash.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/prismjs/components/prism-javascript.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/prismjs/components/prism-python.min.js"></script>
</body>
</html>

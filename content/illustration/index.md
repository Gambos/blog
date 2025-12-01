---
showTableOfContents: false
showDate: false
---

<style>
  /* === 核心瀑布流容器 === 使用 CSS Multi-column 布局实现*/
  .santjer-gallery {
      /* 关键：设置列数 */
      column-count: 2;
      /* 关键：设置列间距 (Gutter) */
      column-gap: 20px;
      width: 100%;
      max-width: 1600px; /* 限制最大宽度，防止大屏太散 */
      margin: 0 auto;
  }

  /* 平板端：2列 */
  @media (min-width: 768px) {
      .santjer-gallery {
          column-count: 3;
      }
  }

  /* 桌面端：3列 (复刻原站的默认视图) */
  @media (min-width: 1200px) {
      .santjer-gallery {
          column-count: 3;
      }
  }

  /* 超大屏：4列 (可选) */
  @media (min-width: 1800px) {
      .santjer-gallery {
          column-count: 4;
      }
  }

  /* === 单个图片容器 === */
  .gallery-item {
      /* 关键：防止元素被分割到两列之间 */
      break-inside: avoid;
      /* 这里的 margin-bottom 必须和上面的 column-gap 一致，保证间距均匀 */
      margin-bottom: 20px;
      position: relative;
      cursor: zoom-in;
      line-height: 0;
      opacity: 0;
      transform: translateY(20px);
      transition: opacity 0.8s ease-out, transform 0.8s ease-out;
      cursor: pointer; /* 提示可点击 */
  }

  .gallery-item.in-view {
      opacity: 1;             /* 变回不透明 */
      transform: translateY(0); /* 回到原位 */
  }

  /* 悬停效果：模仿原站的轻微透明度变化 */
  .gallery-item:hover {
      opacity: 0.85;
  }

  /* === 图片样式 === */
  .gallery-item img {
      width: 100%;
      height: auto; /* 高度自适应，保持原比例 */
      display: block; /* 消除底部空隙 */
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  }

</style>

<script>
document.addEventListener("DOMContentLoaded", function () {
  const imgs = document.querySelectorAll(".gallery-item img");
  let loaded = 0;

  imgs.forEach(img => {
    if (img.complete) {
      loaded++;
      if (loaded === imgs.length) initObserver();
    } else {
      img.addEventListener("load", () => {
        loaded++;
        if (loaded === imgs.length) initObserver();
      });
    }
  });

  function initObserver() {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add('in-view');
          observer.unobserve(entry.target);
        }
      });
    });

    document.querySelectorAll('.gallery-item').forEach(el => observer.observe(el));
  }
});
</script>

<div class="santjer-gallery">
  <div class="gallery-item" style="animation-delay: 0.1s;">
    <img src="img001.jpeg" alt="Art 001" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.2s;">
    <img src="img034.jpeg" alt="Art 002" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.3s;">
    <img src="img004.jpeg" alt="Art 003" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.4s;">
    <img src="img003.jpeg" alt="Art 004" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.5s;">
    <img src="img005.jpeg" alt="Art 005" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.6s;">
    <img src="img006.jpeg" alt="Art 006" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.7s;">
    <img src="img007.jpeg" alt="Art 007" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.8s;">
    <img src="img008.jpeg" alt="Art 008" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 0.9s;">
    <img src="img009.jpeg" alt="Art 009" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.0s;">
    <img src="img010.jpeg" alt="Art 010" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.1s;">
    <img src="img033.jpeg" alt="Art 011" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.2s;">
    <img src="img013.jpeg" alt="Art 012" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.3s;">
    <img src="img012.jpeg" alt="Art 013" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.4s;">
    <img src="img014.jpeg" alt="Art 014" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.5s;">
    <img src="img015.jpeg" alt="Art 015" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.6s;">
    <img src="img016.jpeg" alt="Art 016" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.7s;">
    <img src="img018.jpeg" alt="Art 017" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.8s;">
    <img src="img019.jpeg" alt="Art 018" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 1.9s;">
    <img src="img022.jpeg" alt="Art 019" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.0s;">
    <img src="img020.jpeg" alt="Art 020" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.1s;">
    <img src="img017.jpeg" alt="Art 021" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.2s;">
    <img src="img021.jpeg" alt="Art 022" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.3s;">
    <img src="img023.jpeg" alt="Art 023" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.4s;">
    <img src="img024.jpeg" alt="Art 024" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.5s;">
    <img src="img025.jpeg" alt="Art 025" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.6s;">
    <img src="img026.jpg" alt="Art 026" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 2.9s;">
    <img src="img029.jpeg" alt="Art 029" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 3.0s;">
    <img src="img030.jpeg" alt="Art 030" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 3.1s;">
    <img src="img031.jpeg" alt="Art 031" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 3.2s;">
    <img src="img032.jpeg" alt="Art 032" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 3.3s;">
    <img src="img011.jpeg" alt="Art 033" loading="lazy">
  </div>
  <div class="gallery-item" style="animation-delay: 3.4s;">
    <img src="img002.jpeg" alt="Art 034" loading="lazy">
  </div>
</div>
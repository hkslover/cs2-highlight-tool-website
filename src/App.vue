<template>
  <div class="landing-page">
    <HeroSection />

    <div id="features">
      <FeatureSection
        id="feature-import"
        badge="Step 1"
        title="多平台 Demo 导入"
        description="一键获取你的比赛 Demo。支持本地文件、完美对战平台战绩同步、5E 平台批量导入。"
      >
        <ImportVisual />
      </FeatureSection>

      <FeatureSection
        id="feature-clips"
        badge="Step 2"
        title="可视化片段选择"
        description="解析 Demo 后自动提取击杀数据，以直观的击杀通知条展示每一次击杀。按玩家、回合筛选，支持被害者视角录制。"
        :reverse="true"
      >
        <ClipsVisual />
      </FeatureSection>

      <FeatureSection
        id="feature-produce"
        badge="Step 3"
        title="一键自动录制"
        description="点击开始，软件自动启动游戏并逐个录制你选好的高光片段。全程无需手动操作。"
      >
        <ProduceVisual />
      </FeatureSection>

      <FeatureSection
        id="feature-edit"
        badge="Step 4"
        title="快速剪辑"
        description="从录制历史直接选取片段加入剪辑序列，设置转场效果，一键导出成片。"
        :reverse="true"
      >
        <EditVisual />
      </FeatureSection>
    </div>

    <FooterSection />
  </div>
</template>

<script setup lang="ts">
import HeroSection from './components/HeroSection.vue'
import FeatureSection from './components/FeatureSection.vue'
import ImportVisual from './components/visuals/ImportVisual.vue'
import ClipsVisual from './components/visuals/ClipsVisual.vue'
import ProduceVisual from './components/visuals/ProduceVisual.vue'
import EditVisual from './components/visuals/EditVisual.vue'
import FooterSection from './components/FooterSection.vue'
</script>

<style>
.landing-page {
  min-height: 100vh;
}
</style>

